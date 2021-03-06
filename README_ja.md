# 概要
Firebaseから下記のような注意書きが発せられ、デベロッパーは、2020年11月15日までにFabric SDKからFireabse SDKにアップデートしないといけなくなりました。

"Note: The Fabric SDK is now deprecated and will continue reporting your app's crashes until November 15, 2020. On this date, the Fabric SDK and old versions of the Firebase Crashlytics SDK will stop sending crashes for your app. To continue getting crash reports in the Firebase console, make sure you upgrade to the Firebase Crashlytics SDK versions 17.0.0+ for Android,4.0.0+ for iOS, and 6.15.0+ for Unity."

Xamarinユーザーの場合、Androidでは`Xamarin.Firebase.Crash`のパッケージを使って実装している人が多いと思いますが、  
このパッケージはSDK 16系までしか対応していません。  
代わりに、`Xamarin.Firebase.Crashlytics`というパッケージが作られたようなのですが、  
このパッケージの使用方法に関するドキュメントは存在せず、Firebase公式ドキュメントなどを頼りに、手探りで実装する必要があります。

このドキュメントは、私が`Xamarin.Firebase.Crashlytics`の117.0.0-preview02を用いてFirebase SDKに対応した方法を記載したものです。  
誰かの助けになれば幸いです。  
※公式な手順ではありません。あくまで私が試した記録であり、完璧な手順でもありません。ご了承ください。

[Xamarin.Firebase.Crashlytics](https://www.nuget.org/packages/Xamarin.Firebase.Crashlytics/117.0.0)  
ちなみに、このパッケージにより参照されるCrashlyticsのSDKは17.0.0。

# (2020.10.26)Xamarin.Firebase.Crashlytics 117.0.0 release
2020/10/25に、Xamarin.Firebase.Crashlyticsは正式版の117.0.0をリリースしました。  
さっそく試して、このドキュメントの手順で問題なく動くことを確認済みです。  
よって、概要以外の欄では117.0.0-preview02から117.0.0に書き換えました。  

ま、ほんとは依存関係のバグなんかも一緒に解決されていることを期待したんですがね……。ダメでしたね。

# (2020.12.09)Xamarin.Firebase.Crashlytics 117.1.1 release
別の案件でv117.1.1を使用したところ、その他のパッケージをインストールすることなく、正常に動きました！  
2. パッケージのインストール欄に記載しています。  

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
+++++**(2020.12.09)**  
v117.1.1ですと、Xamarin.Firebase.Crashlyticsのみで正常に動きました。  
|package|version|
----|----
|Xamarin.Firebase.Crashlytics|117.1.1|

+++++  

MyApp.Androidに下記のパッケージを追加。
|パッケージ|バージョン|
----|----
|Xamarin.Firebase.Crashlytics|117.0.0|
|Xamarin.Google.Dagger|2.25.2.1|
|Xamarin.Firebase.Messaging|120.1.7|
|Xamarin.Google.Android.DataTransport.TransportBackendCct|2.2.2|
|Xamarin.Google.Android.DataTransport.TransportRuntime|2.2.2|
|Xamarin.AndroidX.AppCompat.AppCompatResources|1.1.0.1|

`Xamarin.AndroidX.AppCompat.AppCompatResources`は、ビルド時にパッケージが見つからないというエラーが発生したため、.csprojに記載した。

`Xamarin.Google.Dagger`、`Xamarin.Firebase.Messaging`、`Xamarin.Google.Android.DataTransport.TransportBackendCct`、`Xamarin.Google.Android.DataTransport.TransportRuntime`は、起動時のエラーを解消するために必要だった。  
https://github.com/xamarin/GooglePlayServicesComponents/issues/385  
[@sasa-bobic](https://github.com/sasa-bobic)のアドバイスに本当に感謝しています！


## 3. google-service.jsonの再配置
MyApp.Androidには、すでに`google-service.json`が配置されているが、  
念のため、Firebaseのサイトから再ダウンロードし、置き換えた。

ビルドアクションが`Google Services Json`になっていることを確認。


## 4. com.crashlytics.android.mapping_file_id
+++++**(2020.11.05)**  
[@jonathanpeppers](https://github.com/jonathanpeppers)のコメントから、`com.crashlytics.android.build_id`が非推奨になっていることが分かりました。  
https://github.com/xamarin/GooglePlayServicesComponents/issues/393#issuecomment-721753382  
そこで、本ドキュメントでもbuild_idの代わりに`com.google.firebase.crashlytics.mapping_file_id`を使うよう変更しています。  
+++++

以前の`Xamarin.Firebase.Crash`を使用していた時に、下記のファイルをMyApp.Android/Resources/valuesに配置していた。

strings.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="com.crashlytics.android.mapping_file_id">1.0</string>
</resources>
```

これを抜くとエラーになってしまうため、そのままにしておく。  
ない場合は追加する。  
https://github.com/xamarin/XamarinComponents/issues/956#issuecomment-702037279

※正直、mapping_file_idに何を設定するのが適切なのかわかっていません。1.0は適当な値です……。


## 5. AndroidManifest.xmlからfirebase_crashlytics_collection_enabledを削除
`Xamarin.Firebase.Crash`を使用していた際に、`AndroidManifest.xml`に下記を入れていた。  
しかし、これがあるとクラッシュログが有効にならないため、削除する。

AndroidManifest.xml
```xml
// 下記を削除
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
FirebaseCrashlytics.Instance.Log(XXX);
```

### `setCustomKey`への統一
`setBool`、`setString`などが全て`setCustomKey`に変更となった。  
```C#
// 旧
Crashlytics.Crashlytics.SetString(XXX);

// 新
FirebaseCrashlytics.Instance.SetCustomKey(XXX);
```

### `setUserIdentifier`を`setUserId`に変更
メソッドが置き換わった。
```C#
// 旧
Crashlytics.Crashlytics.SetUserIdentifier(userId);

// 新
FirebaseCrashlytics.Instance.SetUserId(userId);
```

### `setUserName`と`setUserEmail`の削除
メソッドが削除された。
```C#
// 下記を削除
Crashlytics.Crashlytics.SetUserName(userName);
```

### `logException`を`recordException`に変更
メソッドが置き換わった。
```C#
// 旧
Crashlytics.Crashlytics.LogException(exception);

// 新
FirebaseCrashlytics.Instance.RecordException(exception);
```


## 7. リリースビルドで実行する
私の場合、以上の手順で、**以前からCrashlyticsを有効にしていたアプリ**ではクラッシュログを取得することができました。  
しかし、Firebase上で新規で作成したアプリは、コンソールが初期化されず、クラッシュログが取れないままとなっていました。  
下記に続きます。


# Failed to retrieve settings from ...
## 状況
Firebaseで新規でアプリを作成し、Crashlyticsを有効にした。  
このアプリに対しクラッシュを発生させたが、コンソールが初期化されず、クラッシュログが取れない。  
<img src="https://github.com/a-imai/XamarinCrashlyticsUpdateSample/blob/image/crashlyticsConsole.png" width="320">

下記手順に従い、コンソールでログを取得した。  
[Enable Crashlytics debug logging](https://firebase.google.com/docs/crashlytics/test-implementation?authuser=0&platform=android#enable_debug_logging)  
すると、下記のようなエラーが出ている。
```
E FirebaseCrashlytics: Failed to retrieve settings from https://firebase-settings.crashlytics.com/spi/v2/platforms/android/gmp/XXXX/settings
```

このエラーについて、Googleに確認を取りながら対処した。  


## 対処方法
### re-onboard
Googleに問い合わせたところ、アプリのre-onboardを試すよう言われた。  
下記手順で実施。  
1. Firebaseのプロジェクトから、エラーが発生しているアプリを削除
2. 削除したアプリと同じパッケージ名で、再度アプリを作成
3. `google-service.json`をダウンロードしてMyApp.Androidに配置
4. Crashlyticsのコンソールで、Crashlyticsを有効にするを押下
5. MyAppをリリースビルドで実行し、クラッシュさせ、再起動する

5の時点で、`Failed to retrieve settings from ...`のエラーは発生せず、設定の読み込みに成功したとログに出力された。  
しかし、実際にCrashlyticsのコンソールでクラッシュログが確認できたのは、下記手順を行った時だった。  

6. 再度起動したアプリを、**もう一度クラッシュさせる**  


Googleいわく、こうしたエラー（`Failed to retrieve settings from ...`）に陥った際に、re-onboardを試みるのは一般的な解決策であるという。  
まずはre-onboardを試し、それでも解決しなければGoogleに問い合わせを行う方がよさそう。  

二度目のクラッシュでのみクラッシュログが確認できた事に関しては、  
Crashlyticsは、クラッシュ後にバックグラウンドのネットワーク接続を介してレポートを送信しようとするのだが、  
これが保証されるのは、アプリが再起動してからであるため、とのこと。  
……つまり、最初のクラッシュでレポート送信が保留され、再起動後の別のクラッシュ時に、保留になっていたレポートが送信された？？  
これが良い仕様だとは思えないのだが、Googleは意図された動作だと言っている。うまく伝わっていない可能性もあるけど……。

また、Googleによると、CrashlyticsのSDKバージョンが17.0.0であることが、現象の原因の一つの可能性があるとのこと。  
（そりゃあ、Google側はどんな状況であっても最新SDKを使えってまず言うよね……）  
これに関してはXamarin.Firebase.Crashlyticsのバージョンアップが行われることを期待するしかない。  
