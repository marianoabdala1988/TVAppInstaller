# TVAppInstaller

TVAppInstaller connects an Android phone to an Android TV or TV box over ADB on the local network. Use it to install APKs, manage apps, and choose the TV's home launcher. It speaks ADB directly in Java, so it does not run `adb.exe` or require Platform-Tools on the phone.

[English](README.md) | [Español](README.es.md)

## Current version

| | |
|---|---|
| Version | `1.0` |
| Version code | `1` |
| APK | [TVAppInstaller_v1.0-vc1_20261120.010954.apk](app/build/outputs/apk/debug/TVAppInstaller_v1.0-vc1_20261120.010954.apk) |

This is a **debug** build signed with the project's debug key. It is suitable for testing and manual installation, but is not a production release.

## Features

- Connect to and disconnect from a TV by IP address and ADB port (`5555` by default).
- Install an APK selected with Android's file picker.
- List third-party apps and uninstall selected apps after confirmation.
- Find available home launchers and ask the TV to make one the default.
- View operation status and output in the app's persistent log.
- Use separate screens for connecting, installing, managing apps, and launchers.

## Requirements and compatibility

- The phone running the app must use Android **5.0 (API 21) or later** (`minSdk 21`).
- The TV must expose **classic ADB over TCP**, be reachable on the network, and accept the app's RSA key.
- The phone and TV must be able to communicate on the selected port.
- TV commands vary by Android version and manufacturer, so launcher management and APK installation may behave differently across models.

**Wireless debugging pairing is not supported.** This app uses classic ADB TCP, usually on port `5555`; it cannot pair using the modern Wireless debugging code or QR flow.

## Enable Developer Options and ADB on your TV

Settings labels vary between Android TV versions and manufacturers. These are the usual steps:

1. Open **Settings** and go to **Device Preferences** (or **System**) > **About**.
2. Find **Build** or **Build number**, then press **Select/OK** seven times until the TV says that Developer Options are enabled.
3. Go back to Settings and open **Developer options**.
4. Enable the option that exposes **ADB over the network**. It may be called **ADB debugging**, **Network debugging**, or **Debugging over network**. The exact option differs by TV.
5. If the TV only offers **Wireless debugging** with a pairing code or QR code, this app cannot connect using that mode. Enabling Developer Options or USB debugging alone does not necessarily make classic ADB TCP available.
6. Find the TV's IP address in its network settings. Keep the port at `5555` unless the TV specifies a different one.
7. Connect the phone and TV to a network that allows them to communicate.

If you cannot find a setting for classic ADB over TCP, check the instructions for your TV model. The app cannot enable ADB on the TV remotely.

## First connection

1. Open TVAppInstaller and enter the TV's IP address and ADB port.
2. Tap **Connect**.
3. Accept the RSA authorization prompt on the TV. The app cannot connect until you approve it.
4. Choose **Install APK**, **Apps**, or **Launchers** from the menu. The app asks for confirmation before uninstalling an app or changing the launcher.

If it does not connect, check the IP address, port, network, and ADB-over-TCP setting, then look for the authorization prompt on the TV.

## How ADB works in the app

`AdbRunner` uses **AdbLib** to connect directly to the TV. The app stores its RSA keys in its private storage and uses the authorized connection to list or uninstall third-party apps, install a selected APK, and query or change the home launcher.

For installation, the APK is copied temporarily to the TV and the app attempts to remove the temporary file afterward. The `adb ...` lines in the log are readable operation traces; they are not commands run through an external ADB program.

## Privacy and safe use

- The app connects only to the address you enter; it does not scan the network for devices.
- The TV must approve the RSA key. The app does not bypass this authorization.
- The selected APK is stored temporarily during transfer, and the app attempts to delete temporary copies afterward.
- Use the app only with TVs you are authorized to manage. Review and confirm each uninstall or launcher change.

## Build from source

On Windows, install the Android SDK and run this from the project root:

```powershell
.\gradlew.bat :app:assembleDebug --console=plain
```

The debug APK is written to `app/build/outputs/apk/debug/`. The project uses Android Gradle Plugin `7.2.1`, Gradle `7.4`, `compileSdk 33`, and `targetSdk 33`.

> The current release build configuration also uses the debug signing key. Configure and protect your own signing key before distributing a release.

## Main dependencies

- [AdbLib](https://github.com/tananaev/adblib) — `com.tananaev:adblib:1.3`, a Java implementation of the ADB protocol.
- AndroidX Core KTX `1.9.0`.
- AndroidX AppCompat `1.6.1`.
- Material Components `1.9.0`.
- ConstraintLayout `2.1.4`.

## Project structure

- `app/src/main/java/com/example/tvappinstaller/MainActivity.java`: screens, navigation, input validation, and UI actions.
- `app/src/main/java/com/example/tvappinstaller/AdbRunner.java`: authenticated ADB connection and device operations.
- `app/src/main/res/layout/`: activity and feature screen layouts.
- `app/src/main/AndroidManifest.xml`: app manifest and network permission.
- `app/build.gradle`: Android module configuration and dependencies.

## Version 1.0 highlights

- Added IPv4 connection, default port `5555`, and RSA authorization.
- Added third-party app listing and confirmed uninstall.
- Added APK selection, transfer, and installation.
- Added separate screens, a persistent log, and home launcher selection.

## Ideas for the future

The long-term dream is to let TVAppInstaller connect to a TV remotely and feel a bit like TeamViewer: see the TV screen and control it from a phone. VNC could be one option to explore, but nothing is decided or implemented yet. It will depend on TV and Android support, user approval, and a secure connection.

## Testing and known limitations

The Java code compiled successfully during development, but ADB commands can vary by TV manufacturer. Test APK installation and launcher changes on the target TV model.

## Thanks and licenses

Thanks to [Anton Tananaev (@tananaev)](https://github.com/tananaev) for [AdbLib](https://github.com/tananaev/adblib), and to [Cameron Gutman (@cgutman)](https://github.com/cgutman), creator of the original project. Thanks as well to the [Android Open Source Project](https://android.googlesource.com/platform/system/core/+/master/adb/protocol.txt) for the ADB protocol documentation, and to everyone who tested the app and shared feedback.

AdbLib is licensed under **BSD 3-Clause**; see its [license](https://github.com/tananaev/adblib/blob/master/LICENSE). TVAppInstaller's own source code does not have a license yet.

For details, see the official Android guides to [Developer Options](https://developer.android.com/studio/debug/dev-options) and [ADB over Wi-Fi](https://developer.android.com/tools/adb#connect-to-a-device-over-wi-fi).
