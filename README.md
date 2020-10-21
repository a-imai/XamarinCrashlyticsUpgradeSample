# Overview
Firebase sent the following message. This forced developers to upgrade from the Fabric SDK to the Fireabse SDK by November 15, 2020.  

"Note: The Fabric SDK is now deprecated and will continue reporting your app's crashes until November 15, 2020. On this date, the Fabric SDK and old versions of the Firebase Crashlytics SDK will stop sending crashes for your app. To continue getting crash reports in the Firebase console, make sure you upgrade to the Firebase Crashlytics SDK versions 17.0.0+ for Android,4.0.0+ for iOS, and 6.15.0+ for Unity."  

Perhaps most Xamarin users used the `Xamarin.Firebase.Crash` package on Android.  
However, this package is not compatible with SDK 17.  
Instead, it seems that the `Xamarin.Firebase.Crashlytics` package was created.  
There is no documentation on how to use this package, and developers have to rely on the official Firebase documentation to implement it.  

This document records how I upgrade to the Firebase SDK using `Xamarin.Firebase.Crashlytics` 117.0.0-preview02.  
I hope it helps someone.  

!! This is not an official procedure. This's just a record I've tried, not a PERFECT procedure. Please note.  

[Xamarin.Firebase.Crashlytics](https://www.nuget.org/packages/Xamarin.Firebase.Crashlytics/117.0.0-preview02)  


# My environment
Operating System & Version : macOS 10.15.4 (19E287)  
Xamarin.Android Version : 11.0.2.0  

Xamarin.Forms : 4.8.0.1451  

My app was previously implemented using `Xamarin.Android.Crashlytics` 2.9.4.4, and is already running on the Crashlytics console.  


# Procedure
## 1. Uninstall existing crashlytics packages
MyApp.Android contained the following Crashlytics related packages.  
I uninstalled all of these.  
|package|version|
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


## 2. Install new packages
Install the following packages on MyApp.Android.  
|package|version|
----|----
|Xamarin.Firebase.Crashlytics|117.0.0-preview02|
|Xamarin.Google.Dagger|2.25.2.1|
|Xamarin.Firebase.Messaging|120.1.7-preview02|
|Xamarin.Google.Android.DataTransport.TransportBackendCct|2.2.2|
|Xamarin.Google.Android.DataTransport.TransportRuntime|2.2.2|
|Xamarin.AndroidX.AppCompat.AppCompatResources|1.1.0.1|

`Xamarin.AndroidX.AppCompat.AppCompatResources` was pasted in .csproj due to an error that could not find the package at build time.  

`Xamarin.Google.Dagger`,`Xamarin.Firebase.Messaging`,`Xamarin.Google.Android.DataTransport.TransportBackendCct`,`Xamarin.Google.Android.DataTransport.TransportRuntime` is needed to clear the error when launching the app.  
https://github.com/xamarin/GooglePlayServicesComponents/issues/385  
I really appreciate [@sasa-bobic](https://github.com/sasa-bobic)'s advice!


## 3. Reinstall `google-service.json`
MyApp.Android already has `google-service.json`, but I re-downloaded it from the Firebase site and replaced it just in case.  

Make sure the build action is `Google Services Json`.  


## 4. com.crashlytics.android.build_id
When I was using the previous `Xamarin.Firebase.Crash`, I had the following files in MyApp.Android/Resources/values.  

strings.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="com.crashlytics.android.build_id">1.0</string>
</resources>
```

If you remove this, an error will occur, so leave it as it is.  
If not, add it.  
https://github.com/xamarin/XamarinComponents/issues/956#issuecomment-702037279  

(Honestly, I don't know what to set for build_id.  
 1.0 is a sloppy value...)  


## 5. Removed `firebase_crashlytics_collection_enabled` from `AndroidManifest.xml`
When I was using `Xamarin.Firebase.Crash`, I put the following in `AndroidManifest.xml`.  
However, this does not enable crash logging, so delete it.  

AndroidManifest.xml
```xml
<meta-data android:name="firebase_crashlytics_collection_enabled" android:value="false" />
```


## 6. Implementation fix
Refer to the Firebase upgrade procedure and modify the implementation on MyApp.Android.  
[Upgrade to the Firebase Crashlytics SDK](https://firebase.google.com/docs/crashlytics/upgrade-sdk?hl=en&platform=android)

### Remove Fabric reference
MainActivity.cs
```C#
// Deleted below
Fabric.Fabric.With(this, new Crashlytics.Crashlytics());
```

### Log method changes
The static `Crashlytics.log` method has been removed and now uses the instance method instead.  
```C#
// Old
Crashlytics.Crashlytics.Log(XXX);

// New
FirebaseCrashlytics.Instance.Log(XXX);
```

### Unification to setCustomKey
