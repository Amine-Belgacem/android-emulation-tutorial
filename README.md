# Android emulation without Android Studio

Set up the Android SDK, create virtual devices, and run the emulator from PowerShell without installing Android Studio.

This guide retains the original **Windows 7 x64 Ultimate and PowerShell** environment, Java 8, and Android API 23/26 examples. Dependencies are downloaded individually.

## Contents

- [Download the prerequisites](#download-the-prerequisites)
- [Set up Java and the SDK](#set-up-java-and-the-sdk)
- [Install SDK packages](#install-sdk-packages)
- [Create and run virtual devices](#create-and-run-virtual-devices)
- [Google Play Store](#google-play-store)
- [Command reference](#command-reference)

## Download the prerequisites

| Component | Original version or selection | Download source |
|---|---|---|
| Java JDK | Windows x64 JDK 8u45: `jdk-8u45-windows-x64.exe` | [Oracle Java 8 archive](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html) |
| Android SDK Tools | `sdk-tools-windows-3859397.zip` | [Google-hosted SDK archive](https://dl.google.com/android/repository/sdk-tools-windows-3859397.zip) |
| 7-Zip | An archive manager for ZIP and TAR files | [7-Zip downloads](https://www.7-zip.org/download.html) |
| Lzip, for manual Play Store installation | Original Windows package: `lzip-1.11-w32.zip` | [GNU-hosted Windows archive](https://download.savannah.gnu.org/releases/lzip/lzip-1.11-w32.zip) |
| Open GApps, for manual Play Store installation | **x86 · Android 6.0 · pico** | [Open GApps](https://opengapps.org/) |

Oracle's archive requires an Oracle account. The SDK download above uses the historical `tools/bin` layout described here. Google's current command-line tools use a different layout; see the [legacy SDK Tools release notes](https://developer.android.com/tools/releases/sdk-tools).

The emulator, platform tools, build tools, and Android images are installed separately through the SDK manager. The package identifiers below retain the original examples; availability depends on Google's legacy package repository.

## Set up Java and the SDK

1. Install the downloaded JDK.
2. Create `%LOCALAPPDATA%\Android\Sdk`.
3. Extract the SDK Tools ZIP into that directory. Its `tools` folder should be directly inside `Sdk`.

The resulting layout is:

```text
%LOCALAPPDATA%\Android\Sdk\
    tools\
        bin\
            sdkmanager.bat
            avdmanager.bat
```

In PowerShell, set the paths for the current terminal. Adjust the Java path if the installer used a different directory:

```powershell
$SdkRoot = Join-Path $env:LOCALAPPDATA 'Android\Sdk'
$env:JAVA_HOME = 'C:\Program Files\Java\jdk1.8.0_45'
$env:ANDROID_HOME = $SdkRoot
$env:ANDROID_SDK_ROOT = $SdkRoot
$env:Path = "$env:JAVA_HOME\bin;$SdkRoot\tools\bin;$SdkRoot\platform-tools;$SdkRoot\emulator;$env:Path"

java -version
sdkmanager --help
```

To reuse these settings in new terminals, open **System Properties → Advanced → Environment Variables**. Add `JAVA_HOME`, `ANDROID_HOME`, and `ANDROID_SDK_ROOT` as user variables with the same values. Add these entries to the user `Path`:

```text
%JAVA_HOME%\bin
%ANDROID_HOME%\tools\bin
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\emulator
```

Open a new PowerShell window after saving the variables. No environment-variable editor is required.

## Install SDK packages

Inspect the available packages and accept the required licenses interactively:

```powershell
sdkmanager --list --verbose
sdkmanager --licenses
```

Install the packages used by the original setup:

```powershell
sdkmanager 'build-tools;26.0.1' 'platform-tools' 'platforms;android-26'
sdkmanager 'emulator' 'system-images;android-23;default;x86' 'system-images;android-26;google_apis;x86'
```

### Hardware acceleration in the original environment

The original Intel-based Windows setup used HAXM. It requires an Intel processor with the required virtualization features enabled.

```powershell
sdkmanager 'extras;intel;Hardware_Accelerated_Execution_Manager'
& "$env:ANDROID_HOME\extras\intel\Hardware_Accelerated_Execution_Manager\intelhaxm-android.exe"
```

If the legacy SDK repository no longer offers the HAXM package, its historical Windows installers are preserved in [Intel's HAXM releases](https://github.com/intel/haxm/releases). Select a release compatible with the original operating system and emulator. This section documents the original acceleration method.

## Create and run virtual devices

List device definitions:

```powershell
avdmanager list device
```

Choose an identifier from that output and assign it below. Device identifiers depend on the installed definitions, so the guide does not assume a fixed numeric ID:

```powershell
$DeviceId = 'replace-with-an-identifier-from-the-device-list'
avdmanager create avd -n android23 -k 'system-images;android-23;default;x86' -b x86 -c 500M -d $DeviceId -g default
avdmanager create avd -n android26 -k 'system-images;android-26;google_apis;x86' -b x86 -c 500M -d $DeviceId -g google_apis
```

To enable the host keyboard, close the emulator and set `hw.keyboard=yes` in each AVD's `config.ini`, normally under:

```text
%USERPROFILE%\.android\avd\android23.avd\config.ini
%USERPROFILE%\.android\avd\android26.avd\config.ini
```

Run either virtual device:

```powershell
emulator -avd android23 -skin 320x480
emulator -avd android26 -skin 480x800
```

## Google Play Store

### Images with Play Store included

Inspect the package list for an image containing the `google_apis_playstore` tag. The original tutorial used this distinction:

| Package example | Play Store |
|---|---|
| `system-images;android-25;google_apis_playstore;x86` | Included |
| `system-images;android-25;google_apis;x86_64` | Not included |

An available Play Store image can be installed through `sdkmanager` and selected when creating an AVD.

### Manual installation on the original Android 6.0 image

This is the original Android 6.0 x86 workflow. It uses the `android23` default image created above and requires a writable system image with `adb root` support.

1. Download an **x86 / Android 6.0 / pico** package from [Open GApps](https://opengapps.org/). The original example was named `open_gapps-x86-6.0-pico-20170829.zip`.
2. Extract it with 7-Zip into a working directory of your choice.
3. Extract the downloaded Windows Lzip archive and add the directory containing `lzip.exe` to the user `Path`.
4. Open PowerShell in the extracted Open GApps `Core` directory.

Decompress these component archives when present in that package:

```powershell
lzip -d .\gmscore-x86.tar.lz
lzip -d .\gsfcore-all.tar.lz
lzip -d .\gsflogin-all.tar.lz
lzip -d .\vending-all.tar.lz
```

Extract the resulting `.tar` files with 7-Zip. Locate the following APKs in the extracted component directories and copy them into one working directory:

```text
PrebuiltGmsCore.apk
GoogleServicesFramework.apk
GoogleLoginService.apk
Phonesky.apk
```

Component and APK names refer to the original package; check the extracted contents when using a different build date.

Start the Android 6.0 emulator with a writable system:

```powershell
emulator -writable-system -avd android23 -skin 320x480
```

Wait for Android to boot. In another PowerShell window, change into the directory containing the four APKs. With only that emulator connected, run:

```powershell
adb root
adb remount
adb push .\PrebuiltGmsCore.apk /system/priv-app/
adb push .\GoogleServicesFramework.apk /system/priv-app/
adb push .\GoogleLoginService.apk /system/priv-app/
adb push .\Phonesky.apk /system/priv-app/
adb shell stop
adb shell start
```

The final two commands restart the Android framework. The original tutorial documents Play Store installation on this AVD.

## Command reference

### SDK manager

```powershell
sdkmanager --help
sdkmanager --list --verbose
sdkmanager --list --verbose > packages.txt
sdkmanager 'package-identifier-from-the-list'
sdkmanager --licenses
```

### AVD manager

```powershell
avdmanager --help
avdmanager list avd
avdmanager list target
avdmanager list device
avdmanager delete avd -n 'AVDName'
avdmanager move avd -n 'AVDName' -r 'NewAVDName'
avdmanager move avd -n 'AVDName' -p 'C:\path\to\new-location'
```

In a system-image identifier such as `system-images;android-23;default;x86`, `default` is the tag and `x86` is the ABI. A device identifier can be selected from `avdmanager list device`.

### Emulator

```powershell
emulator -help
emulator -list-avds
emulator -avd 'AVDName' -skin '320x480'
```
