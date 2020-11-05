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

[Xamarin.Firebase.Crashlytics](https://www.nuget.org/packages/Xamarin.Firebase.Crashlytics/117.0.0)  
the Crashlytics SDK referenced by this package is 17.0.0.  

# (2020.10.26)Xamarin.Firebase.Crashlytics 117.0.0 release
On October 25, 2020, Xamarin.Firebase.Crashlytics released the official version 117.0.0.  
I've tried it out and found that the steps in this document work fine.  
Therefore, I rewrote 117.0.0-preview02 to 117.0.0 in this document other than the Overview.  

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
|Xamarin.Firebase.Crashlytics|117.0.0|
|Xamarin.Google.Dagger|2.25.2.1|
|Xamarin.Firebase.Messaging|120.1.7|
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


## 4. com.google.firebase.crashlytics.mapping_file_id
+++++**(2020.11.05)**  
From the comments of [@jonathanpeppers](https://github.com/jonathanpeppers), I learned that `com.crashlytics.android.build_id` is deprecated.  
https://github.com/xamarin/GooglePlayServicesComponents/issues/393#issuecomment-721753382  
So I updated the documentation. Use `com.google.firebase.crashlytics.mapping_file_id` instead.  
+++++

When I was using the previous `Xamarin.Firebase.Crash`, I had the following files in MyApp.Android/Resources/values.  

strings.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="com.google.firebase.crashlytics.mapping_file_id">1.0</string>
</resources>
```

If you remove this, an error will occur, so leave it as it is.  
If not, add it.  
https://github.com/xamarin/XamarinComponents/issues/956#issuecomment-702037279  

(Honestly, I don't know what to set for mapping_file_id.  
 1.0 is a sloppy value...)  


## 5. Removed `firebase_crashlytics_collection_enabled` from `AndroidManifest.xml`
When I was using `Xamarin.Firebase.Crash`, I put the following in `AndroidManifest.xml`.  
However, this does not enable crash logging, so delete it.  

AndroidManifest.xml
```xml
// Deleted below
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

### Unification to `setCustomKey`
`setBool`, ` setString`, etc. have all been changed to `setCustomKey`.  
```C#
// Old
Crashlytics.Crashlytics.SetString(XXX);

// New
FirebaseCrashlytics.Instance.SetCustomKey(XXX);
```

### Change `setUserIdentifier` to `setUserId`
The method has been replaced.  
```C#
// Old
Crashlytics.Crashlytics.SetUserIdentifier(userId);

// New
FirebaseCrashlytics.Instance.SetUserId(userId);
```

### Delete `setUserName` and` setUserEmail`
The method has been deleted.  
```C#
// Deleted below
Crashlytics.Crashlytics.SetUserName(userName);
```

### Change `logException` to` recordException`
The method has been replaced.  
```C#
// Old
Crashlytics.Crashlytics.LogException(exception);

// New
FirebaseCrashlytics.Instance.RecordException(exception);
```


## 7. Run in RELEASE build
In my case, with the above procedure, I was able to get the crash log for **the app that had Crashlytics enabled before**.  
However, for newly created apps on Firebase, the console was not initialized and crash logs could not be taken.  
Continued below.  


# Failed to retrieve settings from ...
## Detail
I created a new app in Firebase and enabled Crashlytics.  
I caused a crash for this app, but the Crashlytics console is not initialized and I cannot get the crash log.  
<img src="https://github.com/a-imai/XamarinCrashlyticsUpdateSample/blob/image/crashlyticsConsole.png" width="320">

I got the log on the console according to the following procedure.  
[Enable Crashlytics debug logging](https://firebase.google.com/docs/crashlytics/test-implementation?authuser=0&platform=android#enable_debug_logging)  
Then, the following error is displayed.  
```
E FirebaseCrashlytics: Failed to retrieve settings from https://firebase-settings.crashlytics.com/spi/v2/platforms/android/gmp/XXXX/settings
```

I dealt with this error while checking with Google.  


## Workaround
### re-onboard
When I contacted Google, they told me to try the app's re-onboard.  
I did this with the following steps.  
1. Remove the failing app from my Firebase project.  
2. Create the app again with the SAME package name as the deleted app.
3. Download `google-service.json` and put it on MyApp.Android.  
4. In the Crashlytics console, clicke Enable Crashlytics.  
5. Run MyApp in release build and crash, relaunch.  

At the time of 5, the error of `Failed to retrieve settings from ...` did not occur, and it was output to the log that the settings were successfully read.  
However, the crash log was actually confirmed on the Crashlytics console when the following procedure was performed.  

6. **Crash the relaunched app again**.  

Google says it's a common solution to try re-onboard when you get into these errors (`Failed to retrieve settings from ...`).  
It seems better to try re-onboard first, and if that doesn't work, contact Google.  

About the fact that the crash log could be confirmed only in the second crash,  
Google says Crashlytics tries to submit the reports via background network connection post-crash,  
but it's not guaranteed until the app is relaunched.  
...That means, on the first crash the report was pending,  and on another crash after relaunch the pending report was submitted?  
I don't think this is good, but Google says it's the intended behavior. Or, it's possible that my explanation didn't get through well...  

Also, according to Google, the SDK version of Crashlytics is 17.0.0, which may be one of the causes of the error.  
(Well, we know, of course Google says that we must use the latest SDK in any situation...)  
Regarding this, we can only expect that the version of Xamarin.Firebase.Crashlytics will be upgraded.  
