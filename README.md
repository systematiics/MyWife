# Clean TV for Hisense SmartTV 4K FFM

Native Java Android TV launcher, Android 12/API 31 minimum, target/compile SDK 35. AndroidX RecyclerView and spring animations. No Internet permission, network polling, videos, or blur. The only repeating task updates the clock once per minute while visible. Typeface: Outfit, SIL Open Font License (assets/OFL.txt).

Dock: YouTube -> Netflix -> Disney+ -> Hulu. Other visible TV apps are sorted alphabetically in four columns. Up/Down moves one row. Up from the dock reaches the top bar; top bar ends stop navigation. The last selected app is remembered. One shared translucent highlight follows focus. Inputs shows actual visible TV input framework entries; the TV determines their labels and availability. Wi-Fi and Settings open system settings. System firmware controls whether the network name is visible; otherwise the pill says Wi-Fi. The input popup uses the Android system popup style, not a blur effect.

## Installation: preserve stock Home

Extract this bundle on Windows. Keep the TV and PC on the same trusted network. USB debugging may show an authorization dialog; accept only for your own computer. Do not expose ADB to the public Internet.

Run PowerShell from the platform-tools folder, replacing the APK path with its real location:

```powershell
.\adb.exe -s 192.168.1.67:5555 push "C:\path\Clean-TV-debug.apk" /data/local/tmp/clean-tv.apk
.\adb.exe -s 192.168.1.67:5555 shell pm install -r /data/local/tmp/clean-tv.apk
.\adb.exe -s 192.168.1.67:5555 shell am start -n tv.cleanhome/.MainActivity
```

Stop on any install error. A verifier failure or manufacturer HOME restriction must be investigated without disabling verification or trying repeated equivalent installers. Successful reinstallation of Tubi did not prove a new HOME app was allowed; the subsequent Clean TV 0.1 installation and direct launch succeeded on this Hisense. Default Home selection is still unverified.

Alternatively, from the extracted bundle invoke build-and-install.ps1 with an explicit -Adb path. It installs and opens the app but does not set default Home or disable anything. The PowerShell helper is provided for Windows; it was not executed on the Linux build machine. Do not relax machine-wide PowerShell execution policy.

Before changing Home, capture the real screen, test D-pad navigation and each app, inputs, Wi-Fi and Settings. Check logs on crashes. No emulator screenshot is proof of real remote functionality.

```powershell
.\adb.exe -s 192.168.1.67:5555 shell screencap -p /sdcard/clean-tv-first.png
.\adb.exe -s 192.168.1.67:5555 pull /sdcard/clean-tv-first.png .
```

After validation only, the default Home can be changed reversibly:

```powershell
.\adb.exe -s 192.168.1.67:5555 shell cmd package set-home-activity --user 0 tv.cleanhome/.MainActivity
```

Do not disable stock Google TV yet. It may handle physical app, settings and input buttons; this version has no accessibility helper to replace those handlers. Test the actual remote buttons before any later decision. Do not disable setupwraith: its recovery activity appeared disabled in the original HOME report, and Wi-Fi ownership has not been checked.

## Existing debloat state

Backups are on the user's Windows PC in tv-backup-20261020-202311; preserve them privately. No full settings or account data is bundled here.

- ACR com.vt.tv.evservice disabled; home screenshot and YouTube playback verified by user.
- Retail demo com.vt.storemode disabled; Quick Settings screenshot and YouTube playback verified by user.
- com.vidaa.oemtabcontentprovider was disabled, broke the Hisense tab/Quick Settings workflow, and was re-enabled. Keep it enabled.
- Original disabled packages com.mediatek.tis, com.mediatek.dtv.tvinput.isdbtuner and com.google.android.youtube.tvmusic are left unchanged.
- Keep voice assistant, Chromecast, Miracast, firmware/remote updates, Bluetooth, input/tuner services, TV app, keyboard, Play Store and Play Services.
- Keep YouTube, Netflix, Disney+, Hulu, Prime Video, Paramount+, Peacock, Max and Tubi. Other apps remain enabled without an explicit removal preference.

Never uninstall system packages. Keep one pm operation per ADB shell call. To re-enable a package:

```powershell
.\adb.exe -s 192.168.1.67:5555 shell pm enable --user 0 PACKAGE.NAME
```

Place restore-tv.cmd beside adb.exe and run it while authorized. It re-enables our disabled packages and restores Google TV as Home. It does not enable the three originally disabled packages or restore arbitrary settings. It will not undo unrelated future changes.

## Build and customize

Install a full JDK 17 or newer (tested build environment described in VALIDATION.md), Android SDK platform 35 and build-tools 35.0.0. Configure Android SDK with a local.properties sdk.dir or ANDROID_HOME. Gradle wrapper pins 8.13; Android Gradle Plugin pins 8.9.2. From this source directory:

```powershell
.\gradlew.bat :app:assembleDebug :app:lintDebug :app:testDebugUnitTest
```

Then use build-and-install.ps1 -Build -Adb C:\path\platform-tools\adb.exe. It requires an already authorized TV. A debug build made on another computer uses a different signing key; Android rejects replacing the delivered APK with a different signer. Keep the stock launcher available and return to it before uninstalling only your custom tv.cleanhome app if a signing-key change is needed. Never uninstall the stock launcher. Debug signing keys are not included in this bundle.

Edit app/src/main/assets/config.json to change dock package order, greeting, background hex color, or background_style (black, midnight, lavender or violet), then rebuild. The gradient styles are drawn locally without bitmap downloads, blur, or animation. For a static image wallpaper, place wallpaper.jpg in assets and rebuild. It is decoded at reduced size to limit RAM. Default in version 0.3 is a static lavender gradient, selected by the user. Use background_style black for pitch black. Dock apps must have a LEANBACK_LAUNCHER activity and be enabled. The app does not edit system wallpaper.

## Optional DNS and final cleanup

Ad-blocking Private DNS was not enabled. If desired, discuss it before changing settings. It does not block YouTube ads and can affect playback. Back up the prior DNS configuration; undo to the exact prior settings, or private_dns_mode opportunistic if that was the baseline. Test streaming after any DNS change.

When device work is complete, use TV settings to switch USB/network debugging off, revoke debugging authorizations, and reboot. From Windows check:

```powershell
Test-NetConnection 192.168.1.67 -Port 5555
```

TcpTestSucceeded must be False. Disconnecting the PC's ADB client alone does not close the TV's port. No cleanup or port-closure verification has happened yet.
