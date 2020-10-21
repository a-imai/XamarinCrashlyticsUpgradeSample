# 概要
Firebaseから下記のような注意書きが発せられ、デベロッパーは、2020年11月15日までにFabric SDKからFireabse SDKにアップデートしないといけなくなりました。

"Note: The Fabric SDK is now deprecated and will continue reporting your app's crashes until November 15, 2020. On this date, the Fabric SDK and old versions of the Firebase Crashlytics SDK will stop sending crashes for your app. To continue getting crash reports in the Firebase console, make sure you upgrade to the Firebase Crashlytics SDK versions 17.0.0+ for Android,4.0.0+ for iOS, and 6.15.0+ for Unity."

Xamarinユーザーの場合、AndroidではXamarin.Firebase.Crashのパッケージを使って実装している人が多いと思いますが、  
このパッケージはSDK 16系までしか対応していません。  
代わりに、Xamarin.Firebase.Crashlyticsというパッケージが作られたようなのですが、  
このパッケージの使用方法に関するドキュメントは存在せず、Firebase公式ドキュメントなどを頼りに、手探りで実装する必要があります。

このドキュメントは、私がXamarin.Firebase.Crashlyticsの117.0.0-preview02を用いてFirebase SDKに対応した方法を記載したものです。  
誰かの助けになれば幸いです。


# 実装環境
Operating System & Version : macOS 10.15.4 (19E287)  
Xamarin.Android Version : 11.0.2.0

Xamarin.Forms : 4.8.0.1451

Xamarin.Android.Crashlytics 2.9.4.4を用いて実装済みで、すでに運用しているプロジェクトにて実装。


# 手順
## 1. 既存のパッケージのアンインストール
MyApp.Androidには、下記のCrashlyticsに関連するパッケージが入っていた。  
これらを全てアンインストールした。
|パッケージ|バージョン|
----|----
|Xamarin.Android.Crashlytics|2.9.4.4|
|Xamarin.Android.Crashlytics.Answers|1.4.2.4|
|Xamarin.Android.Crashlytics.Beta|1.2.9.4|
|Xamarin.Android.Crashlytics.Core|2.6.3.4|
|Xamarin.Android.Fabric|1.4.3.4|
|Xamarin.Build.Download|0.9.0|
|Xamarin.Firebase.Common|71.1610.0|
|Xamarin.Firebase.Core|71.1601.0|
|Xamarin.Firebase.Iid|71.1710.0|
|Xamarin.GooglePlayServices.Basement|71.1620.0|
|Xamarin.GooglePlayServices.Tasks|71.1601.0|


## 2. パッケージのインストール
MyApp.Androidに下記のパッケージを追加。
|パッケージ|バージョン|
----|----
|Xamarin.Firebase.Crashlytics|117.0.0-preview02|
|Xamarin.Google.Dagger|2.25.2.1|
|Xamarin.Firebase.Messaging|120.1.7-preview02|
|Xamarin.Google.Android.DataTransport.TransportBackendCct|2.2.2|
|Xamarin.Google.Android.DataTransport.TransportRuntime|2.2.2|
|Xamarin.AndroidX.AppCompat.AppCompatResources|1.1.0.1|

Xamarin.AndroidX.AppCompat.AppCompatResourcesは、ビルド時にパッケージが見つからないというエラーが発生したため、.csprojに記載した。

Xamarin.Google.Dagger、Xamarin.Firebase.Messaging、Xamarin.Google.Android.DataTransport.TransportBackendCct、Xamarin.Google.Android.DataTransport.TransportRuntimeは、起動時のエラーを解消するために必要だった。  
https://github.com/xamarin/GooglePlayServicesComponents/issues/385  
[@sasa-bobic](https://github.com/sasa-bobic)のアドバイスに本当に感謝しています！


## 3. google-service.jsonの再配置
MyApp.Androidには、すでにgoogle-service.jsonが配置されているが、  
念のため、Firebaseのサイトから再ダウンロードし、置き換えた。

ビルドアクションがGoogle Services Jsonになっていることを確認。


## 4. com.crashlytics.android.build_id
以前のXamarin.Firebase.Crashを使用していた時に、下記のファイルをMyApp.Android/Resources/valuesに配置していた。

strings.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="com.crashlytics.android.build_id">1.0</string>
</resources>
```

これを抜くとエラーになってしまうため、そのままにしておく。  
ない場合は追加する。  
https://github.com/xamarin/XamarinComponents/issues/956#issuecomment-702037279

※正直、build_idに何を設定するのが適切なのかわかっていません。1.0は適当な値です……。


## 5. AndroidManifest.xmlからfirebase_crashlytics_collection_enabledを削除
Xamarin.Firebase.Crashを使用していた際に、AndroidManifest.xmlに下記を入れていた。  
しかし、これがあるとクラッシュログが有効にならないため、削除する。

AndroidManifest.xml
```xml
<meta-data android:name="firebase_crashlytics_collection_enabled" android:value="false" />
```


## 6. 実装の修正
Firebaseの移行手順を参考に、MyApp.Androidにて実装の修正を行う。  
[Upgrade to the Firebase Crashlytics SDK](https://firebase.google.com/docs/crashlytics/upgrade-sdk?hl=en&platform=android)

### Fabricの呼び出しを削除
MainActivity.cs
```C#
// 下記を削除
Fabric.Fabric.With(this, new Crashlytics.Crashlytics());
```

### ログメソッドの変更
静的な`Crashlytics.log`メソッドが削除され、代わりにインスタンスメソッドを用いるようになった。  
```C#
// 旧
Crashlytics.Crashlytics.Log(XXX);

// 新
private static FirebaseCrashlytics crashlytics = FirebaseCrashlytics.Instance;
crashlytics.Log(XXX);
```

### setCustomKeyへの統一
`setBool`、`setString`などが全て`setCustomKey`に変更となった。  
```C#
// 旧
Crashlytics.Crashlytics.SetString(XXX);

// 新
private static FirebaseCrashlytics crashlytics = FirebaseCrashlytics.Instance;
crashlytics.SetCustomKey(XXX);
```

### setUserIdentifierをsetUserIdに変更
メソッドが置き換わった。
```C#
// 旧
Crashlytics.Crashlytics.SetUserIdentifier(userId);

// 新
private static FirebaseCrashlytics crashlytics = FirebaseCrashlytics.Instance;
crashlytics.SetUserId(userId);
```

### setUserNameとsetUserEmailの削除
メソッドが削除された。
```C#
// 以下を削除
Crashlytics.Crashlytics.SetUserName(userName);
```

### logExceptionをrecordExceptionに変更
メソッドが置き換わった。
```C#
// 旧
Crashlytics.Crashlytics.LogException(exception);

// 新
private static FirebaseCrashlytics crashlytics = FirebaseCrashlytics.Instance;
crashlytics.RecordException(exception);
```


## 7. リリースビルドで実行する
私の場合、以上の手順で、**以前からCrashlyticsを有効にしていたアプリ**ではクラッシュログを取得することができました。  
しかし、Firebase上で新規で作成したアプリは、コンソールが初期化されず、クラッシュログが取れないままとなっていました。  
下記に続きます。


# Failed to retrieve settings from ...
