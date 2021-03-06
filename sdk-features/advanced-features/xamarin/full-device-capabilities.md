---
description: >-
  Xamarin specific docs regarding full device screenshare for iOS and full
  device remote control for Android.
---

# Full device capabilities

### Overview

By default, the Cobrowse.io SDKs for iOS and Android will capture the user activity inside your app only. You can enable the capture of the full device, i.e. screens outside your app, including home screen, device settings, and everything else. 

Follow this guide to add the required App Extension for iOS and enable an accessibility service on Android required for capturing full device frames.

{% tabs %}
{% tab title="Xamarin.iOS" %}
**Add a Broadcast Extension project**

In Visual Studio for Mac:

1. Open your Xamarin solution
2. Right click on the solution, select Add &gt; New Project...
3. Navigate iOS &gt; Extension
4. Pick "Broadcast Upload Extension"
5. Enter a name for the target, e.g. `YourApp.iOS.BroadcastUploadExtension`
6. Select your iOS app to add the extension to
7. Create the location for the extension and press "Create"
8. Visual Studio for Mac will create two extension projects for you: `YourApp.iOS.BroadcastUploadExtension` and `YourApp.iOS.BroadcastUploadExtensionUI`. The second project is not required and you can safely delete it.
9. Change the target SDK of your Broadcast Extension target to at least iOS 10.0

**Set up Keychain Sharing**

Your app and the app extension you created above need to share some secrets via the iOS Keychain. They do this using their own Keychain group so they are isolated from the rest of your apps Keychain.

In **both** your **iOS app** and your **extension project** add a Keychain Sharing entitlement for the "io.cobrowse" keychain group.

**Add the bundle ID to your plist**

Take the bundle ID of the **extension** you created above, and add the following entry in your apps `Info.plist` \(_Note:_ **not** in the extensions `Info.plist`\), replacing the bundle ID below with your own:

```markup
<key>CBIOBroadcastExtension</key>
<string>your.app.extension.bundle.ID.here</string>
```

**Add the Cobrowse.io AppExtension NuGet**

The app extension needs a dependency on the CobrowseIO app extension framework. Add the following NuGet to the **extension project** \(not to the iOS app project\):

* [![CobrowseIO.AppExtension.iOS NuGet](https://img.shields.io/nuget/v/CobrowseIO.AppExtension.iOS.svg?label=CobrowseIO.AppExtension.iOS)](https://www.nuget.org/packages/CobrowseIO.AppExtension.iOS/)

**Implement the extension**

Visual Studio will have added `SampleHandler.cs` file as part of the extension project you created earlier. Replace the content of the file with the following:

```csharp
using Xamarin.CobrowseIO.AppExtension;

[Register("SampleHandler")]
public class SampleHandler : CobrowseIOReplayKitExtension
{
    protected SampleHandler(IntPtr handle) : base(handle)
    {
    }
}
```

**Make sure Info.plist points to the correct class**

Open Info.plist of the extension project and make sure that `NSExtension` section looks like this:

```markup
<plist version="1.0">
<dict>
    ...
    <key>NSExtension</key>
    <dict>
        <key>NSExtensionPointIdentifier</key>
        <string>com.apple.broadcast-services-upload</string>
        <key>NSExtensionPrincipalClass</key>
        <string>SampleHandler</string>
        <key>RPBroadcastProcessMode</key>
        <string>RPBroadcastProcessModeSampleBuffer</string>
    </dict>
</dict>
</plist>
```

**Build and run your app**

You're now ready to build and run your app. The full device capability is only available on phsyical devices, it will not work in the iOS simulator.
{% endtab %}

{% tab title="Xamarin.Android" %}
Full device remote control for Android uses an accessibility service that must be enabled on the device to grant access. This feature is supported in API 21 \(5.0 Lollipop\) and above.

**Configure Full Device Control flag**

Add the following line to one of your resources xml files, eg. in `res/values/bools.xml`:

```markup
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <bool name="cobrowse_enable_full_device_control">true</bool>
</resources>
```

_Note: Please add this value to a file with Build Action set to `AndroidResource`, and not to an XML resource file._

Enable the accessibility service the Cobrowse SDK will have added in the main device settings, eg. Settings -&gt; Accessibility -&gt; Your App Name. Note: this only has to be done the very first time.

We also have built some logic to detect if accessibility service is running, and if not, to deep link the user to the settings to enable it.

Show the sample UI with:

```csharp
CobrowseAccessibilityService.ShowSetup(...)
```

Check if accessibility service is already running with:

```csharp
CobrowseAccessibilityService.IsRunning(...)
```

Deep link user to accessibility settings with:

```csharp
Intent intent = new Intent(global::Android.Provider.Settings.ActionAccessibilitySettings);
intent.AddFlags(ActivityFlags.NewTask);
StartActivity(intent);
```

**Notes for unattended access**

For unattended full device access, we strongly recommend:

* Please initiate sessions with push notifications, rather than our default sockets. This will enable unattended access, even when your app has been backgrounded a long time, or force closed. More info at [Initiate sessions with push](../../initiate-sessions-with-push.md).
* Please turn off "Require User Consent" prompts at [https://cobrowse.io/dashboard/settings](https://cobrowse.io/dashboard/settings). Otherwise, a user must always accept the consent prompt on the device before a session can start.
{% endtab %}
{% endtabs %}

### 

### 

