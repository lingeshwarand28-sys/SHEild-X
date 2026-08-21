# Night Safety - Flutter App

Real mobile app version of the silent distress detection prototype. Same
detection logic as the web demo, but now using real phone accelerometer
data and installable on an actual Android/iOS phone.

## 1. Install Flutter SDK (one-time)

1. Download from https://docs.flutter.dev/get-started/install (pick your OS)
2. Extract it, e.g. to `C:\flutter` (Windows) or `~/flutter` (Mac)
3. Add the `flutter/bin` folder to your system PATH
4. Open a terminal and run:
   ```bash
   flutter doctor
   ```
   Follow whatever it tells you is missing (Android Studio, Android SDK, an emulator or a USB-connected phone, etc.)
5. In VS Code, install the **Flutter** extension (Extensions panel → search "Flutter")

## 2. Set up this project

1. Open this folder in VS Code: `File > Open Folder` → select `night_safety_flutter`
2. Open a terminal in VS Code (`` Ctrl+` ``) and run:
   ```bash
   flutter pub get
   ```
   This downloads the `sensors_plus` and `geolocator` packages listed in `pubspec.yaml`.

## 3. Run it

**Option A — Android phone via USB (recommended, real sensors)**
1. On your phone: enable Developer Options → USB Debugging (search "enable developer options [your phone model]" if unsure)
2. Connect phone to laptop via USB
3. In terminal:
   ```bash
   flutter devices
   ```
   Your phone should appear in the list.
4. Run:
   ```bash
   flutter run
   ```
   This builds the app and installs it directly on your phone.

**Option B — Android emulator (no real phone needed, but no real sensor data)**
1. Open Android Studio → Device Manager → create a virtual device → start it
2. In VS Code terminal:
   ```bash
   flutter run
   ```

## 4. Build an installable APK (to share/install without USB debugging)

```bash
flutter build apk --release
```
The APK will be at `build/app/outputs/flutter-apk/app-release.apk` — copy this to
your phone and install it directly (you may need to allow "install from unknown sources").

## Permissions

This app needs location permission (for the alert's GPS coordinates) and motion
sensor access. Android usually grants these automatically for `geolocator` and
`sensors_plus`, but if location doesn't work, add this to
`android/app/src/main/AndroidManifest.xml` inside the `<manifest>` tag:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

## Trusted contacts + SMS alert (added)

- Tap the contacts icon (top right) to add trusted contacts (name + phone number with country code, e.g. `+91XXXXXXXXXX`).
- Contacts are saved on-device using `shared_preferences` — they persist between app restarts.
- When an alert triggers (countdown reaches 0), the app:
  1. Captures current GPS location
  2. Builds a message with a Google Maps link to that location
  3. Opens the phone's SMS app pre-filled with the message and all trusted contacts as recipients
- **Important**: the user still has to tap "Send" in the SMS app. Android does not allow apps to
  silently send SMS without additional permissions (`SEND_SMS`) and a native platform channel —
  this pre-fill approach works with zero extra native code and is the safest starting point.
- To make sending fully automatic (no tap needed), you'd need to request the `SEND_SMS` runtime
  permission and use a plugin like `flutter_sms` or a platform channel calling Android's
  `SmsManager` directly — this requires more setup and Play Store scrutiny since auto-sending SMS
  is a sensitive permission.

## What's next (making this production-ready)

| Area | What to do |
|---|---|
| Fully silent SMS | Request `SEND_SMS` permission + native `SmsManager` call to send without the user tapping Send |
| Better detection | Swap the threshold rules for a trained ML model (`tflite_flutter` package + a model trained on HAR datasets) |
| Background running | Right now detection only runs while the app is open. For real use, you'd need a background service (`flutter_background_service` package) so it works even when the phone is locked |
