# Handover of TVAppInstaller to Cline + Ollama

This file summarizes the project state and contains a prompt to continue working with Cline and the local model. Use it as initial context; the current source files in the workspace are the definitive reference.

## How to use this handover
1. Open the folder `F:\Users\Abdal\Desktop\TVAppInstaller` in VS Code.
2. Copy the **Prompt for Cline** block below into Cline.
3. Make sure Cline has file-reading permissions and explicitly ask it to read the paths before proposing or making changes.
4. Do not paste or send the `adb` executable from the root directory to external services: it is not part of the current Android backend.

## Prompt for Cline

```text
Continue the existing Android project TVAppInstaller at:
F:\Users\Abdal\Desktop\TVAppInstaller

Before answering or modifying anything, use the read-file tools in this workspace and read these complete files:

- app/src/main/java/com/example/tvappinstaller/MainActivity.java
- app/src/main/java/com/example/tvappinstaller/AdbRunner.java
- app/src/main/res/layout/activity_main.xml
- app/src/main/AndroidManifest.xml
- app/build.gradle
- build.gradle
- settings.gradle
- gradle/wrapper/gradle-wrapper.properties
- gradle.properties

Also check the files present under app/src/main (especially assets), as the state may have changed since this handover. Do not assume the text of this document replaces reading the code.

PRODUCT OBJECTIVE

This is an Android app named TVAppInstaller, package `com.example.tvappinstaller`, running on an Android phone/emulator that manages an Android TV controlled and authorized by the user via classic ADB TCP over the local network.
The interface must allow:

1. Entering IPv4 and optional port (5555 by default), connecting, and disconnecting.
2. Upon connecting, toggling the connection button to “Desconectar” and opening a menu.
3. Having separate screens to install, list/uninstall apps, and manage launchers; each feature screen has an “Atrás” button at the top left.
4. Displaying results and errors in a persistent console inside the app.
5. Listing third-party packages on the TV and displaying them as checkboxes.
6. Selecting one or more apps, confirming, and uninstalling them.
7. Tapping “Instalar APK”, picking an APK via the Android file picker, sending it to the TV, and installing it.
8. Querying HOME/launcher activities, selecting one, and requesting the TV to set it as default, with confirmation.

The app must only use TVs authorized by the user. Do not add network scanning, automatic target discovery, authentication bypass, or hidden features. The RSA authorization requested by the TV must remain active and visible.

CURRENT ARCHITECTURE

- `MainActivity.java` manages the screens, validates IPv4/port, and handles buttons, the APK picker, the package list, and launcher selection.
- `activity_main.xml` contains the screen host and persistent console. Individual screens are `screen_connection.xml`, `screen_menu.xml`, `screen_install.xml`, `screen_apps.xml`, and `screen_launchers.xml`.
- The connection screen only contains IP, port, and the toggle button; if the port is empty, 5555 is used. Connecting opens the menu; going back from the menu returns to connection and the button shows “Desconectar”.
- `AdbRunner.java` is the asynchronous backend. It uses the Java library AdbLib (`com.tananaev:adblib:1.3`), `Socket`, and the classic ADB protocol; it does not spawn an `adb.exe` process or a native binary.
- The RSA key is generated/read inside the app's private storage (`filesDir/adbkey` and `adbkey.pub`). The first connection may display the authorization prompt for this key on the TV.
- To list apps, it uses the ADB shell service with `pm list packages -3`.
- To uninstall, it uses the ADB shell service with `pm uninstall <package>`.
- To query launchers, `AdbRunner` attempts `cmd package query-activities` with the MAIN/HOME intent and uses `pm query-activities` as a fallback when the subcommand is unavailable. To change the launcher, it tries `cmd package set-home-activity` with a fallback to `pm set-home-activity`.
- To install, the current code selects a `Uri` via `ACTION_OPEN_DOCUMENT`, temporarily copies the content to cache, validates that the ZIP contains `AndroidManifest.xml`, transfers the file via the `sync:` service to `/data/local/tmp/`, executes `pm install -r`, and then attempts to delete the remote temporary file.
- The `adb ...` lines in the console are human-readable traces; they are not commands executed in CMD/PowerShell. Data is sent via the ADB protocol.
- The main layout is in `activity_main.xml`. Keep the IDs used by `MainActivity`; if you change the UI, update both sides consistently.

IMPORTANT FILES

- `app/src/main/java/com/example/tvappinstaller/MainActivity.java`
- `app/src/main/java/com/example/tvappinstaller/AdbRunner.java`
- `app/src/main/res/layout/activity_main.xml`
- `app/src/main/res/layout/screen_connection.xml`
- `app/src/main/res/layout/screen_menu.xml`
- `app/src/main/res/layout/screen_install.xml`
- `app/src/main/res/layout/screen_apps.xml`
- `app/src/main/res/layout/screen_launchers.xml`
- `app/src/main/AndroidManifest.xml`
- `app/src/main/res/values/strings.xml`
- `app/build.gradle`
- `build.gradle`
- `settings.gradle`
- `gradle/wrapper/gradle-wrapper.properties`

CURRENT CONFIGURATION

- applicationId and namespace: `com.example.tvappinstaller`
- minSdk 21, targetSdk 33, compileSdk 33
- Android Gradle Plugin 7.2.1 and Gradle wrapper 7.4
- ADB dependency: `com.tananaev:adblib:1.3`
- The manifest declares `INTERNET`. Document selection uses Storage Access Framework; it should not require `READ_EXTERNAL_STORAGE`.
- `local.properties` contains a local SDK path specific to this machine. Do not replace it with a made-up path or add it to version control.

KNOWN LIMITATIONS AND ITEMS TO VALIDATE

- The chosen library implements classic ADB TCP with RSA authentication. It does not implement modern “Wireless debugging” pairing via code/TLS. Do not advertise compatibility with that mode without replacing/verifying the backend.
- The TV must allow ADB TCP connections, normally on port 5555, be reachable on the local network, and accept this app's key.
- Commands to enumerate and set HOME activities may vary across Android TV manufacturers/versions. Compilation does not validate these commands; test the launcher screen on a real TV and report the full output before claiming compatibility.
- The `adb` file that was in the workspace root is an x86_64 Linux ELF requiring `/lib64/ld-linux-x86-64.so.2`; it is not an Android executable. The copy in `app/src/main/assets/adb` was removed. Do not copy that ELF back into assets or execute downloaded binaries from `filesDir`.
- Java code compiled cleanly with `.\gradlew.bat :app:compileDebugJavaWithJavac --console=plain`. No APK was generated during that check, and APK installation was not tested against a real TV. The user builds/installs the APK on their own.
- ADB `sync:` transfer and the `pm install -r` flow must be tested on a real TV. If they fail, inspect the app log, verify the SEND/DATA/DONE/OKAY protocol, the remote file mode/path, and the actual response from `pm install`; do not report success unless the TV confirms it.
- `MainActivity` uses `startActivityForResult`, which is deprecated but functional. Migrating to the Activity Result API is an optional improvement, not a requirement to change unnecessarily.
- The `release` build is configured with debug signing: suitable for development, not for distributing a final release.

WORKING MODE

- Inspect first; then briefly explain the issue found.
- Make small, coherent changes following the existing Java/XML/Gradle style.
- Keep all network operations off the main thread.
- Do not concatenate user input into a generic shell command. Keep operations scoped (connect, list, install, uninstall) and validate package names/paths.
- Do not delete user files or change packages/dependencies unless necessary and explained.
- Do not claim the app works on a TV just because it compiles.
- After making changes, run the narrowest check that validates them. For Java compilation use:
  `.\gradlew.bat :app:compileDebugJavaWithJavac --console=plain`
- Do not run `assembleDebug` if the user only asked for source changes or stated they will generate the APK themselves.
- Respond in clear Spanish, as the user is not a programmer.