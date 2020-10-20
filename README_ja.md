## 概要
Firebaseから下記のような注意書きが発せられ、デベロッパーは2020年11月15日までにFabric SDKからFireabse SDKにアップデートする必要ができました。

"Note: The Fabric SDK is now deprecated and will continue reporting your app's crashes until November 15, 2020. On this date, the Fabric SDK and old versions of the Firebase Crashlytics SDK will stop sending crashes for your app. To continue getting crash reports in the Firebase console, make sure you upgrade to the Firebase Crashlytics SDK versions 17.0.0+ for Android,4.0.0+ for iOS, and 6.15.0+ for Unity."

Xamarinユーザーの場合、AndroidではXamarin.Firebase.Crashのパッケージを使っている人が主だと思いますが、
このパッケージはSDK 16系までしか対応していません。
代わりにXamarin.Firebase.Crashlyticsというパッケージが作られました。
このパッケージの使用方法に関するドキュメントはほぼありません。

このドキュメントは、私がXamarin.Firebase.Crashlyticsの117.0.0-preview02を用いてFirebase SDKに対応した方法を記載したものです。
