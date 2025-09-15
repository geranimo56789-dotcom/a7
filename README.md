# Simple Flutter App + iOS CI (GitHub Actions)

This repo contains a tiny Flutter app and a GitHub Actions workflow that builds an iOS app on macOS. It can:
- Always: build an unsigned .app artifact (no Apple signing required)
- If you add signing secrets: produce a signed .ipa suitable for App Store Connect (export method configurable)

## What you get
- `lib/main.dart`: counter app
- `pubspec.yaml`
- `.github/workflows/ios-build.yml`: build on demand via Workflow Dispatch
- `.gitignore`: ignores generated platform folders; CI generates them

## Prerequisites
- A GitHub repo with this code pushed
- For signed IPA: Apple Developer Program account, distribution certificate (.p12) and a matching provisioning profile for your bundle id, plus Team ID

## Trigger a build
Use the Actions tab in GitHub and run the “iOS Build (Flutter)” workflow. Choose:
- export_method: app-store (default), ad-hoc, development, enterprise
- build_mode: release (default) or debug

Artifacts are uploaded as `ios-build-<mode>`.

## Configure signing (to get a signed .ipa)
Add the following GitHub Secrets (Repo Settings > Secrets and variables > Actions):
- APPLE_CERT_P12: Base64 of your .p12 distribution certificate
- APPLE_CERT_PASSWORD: Password for the .p12
- PROVISIONING_PROFILE_BASE64: Base64 of your .mobileprovision
- APPLE_TEAM_ID: Your Team ID (e.g., 1A2B3C4D5E) [Secret]

Add the following GitHub Variable (Repo Settings > Variables > Actions):
- IOS_BUNDLE_ID: Your iOS bundle id (e.g., com.example.simple)

### How to prepare base64 values (Windows PowerShell)
Convert your certificate and provisioning profile to Base64 strings:

```powershell
# Convert .p12 to base64
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\path\to\dist.p12")) | Set-Clipboard

# Convert .mobileprovision to base64
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\path\to\profile.mobileprovision")) | Set-Clipboard
```
Paste the clipboard contents into the corresponding GitHub Secret.

## Notes
- Without signing secrets, the workflow still builds `build/ios/iphoneos/*.app` for device installs via Xcode.
- For TestFlight/App Store upload, you can upload the produced IPA with Transporter/Xcode or add a fastlane step. This template keeps it simple and does not auto-upload.
- Platform folders (ios/android/etc.) are not committed; CI generates them. If you prefer, you can commit them and remove the create step.

## Local run (optional)
If you have Flutter installed locally and Xcode on macOS, you can run:

```powershell
flutter pub get
flutter create .  # if ios/ folder is missing
flutter run       # choose a device/simulator
```

## Troubleshooting
- Codesign errors: Ensure the provisioning profile matches the bundle id and the certificate belongs to the same team.
- No IPA generated: Check that all required secrets are present and that your profile is a distribution profile for the chosen export method.
