# DMS Installation Guide

This guide is for installing DMS on a Windows PC from the public release repository.

## What You Need

- Windows 10 or Windows 11.
- The latest DMS release files:
  - `DMS-Setup-vX.Y.Z.exe`
  - `DMS_Publisher.cer`
  - `Install-DMS.ps1`
- An internet connection for GitHub download and in-app updates.
- Administrator approval may be required when the installer writes to `Program Files`.

## First-Time Install

Download these three files from the latest public GitHub release and keep them in the same folder:

```text
DMS-Setup-vX.Y.Z.exe
DMS_Publisher.cer
Install-DMS.ps1
```

Open PowerShell in that folder.

For the internal self-signed DMS certificate, run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\Install-DMS.ps1 -CertificatePath ".\DMS_Publisher.cer" -InstallerPath ".\DMS-Setup-vX.Y.Z.exe" -TrustAsRoot
```

For a public CA-issued certificate, run without `-TrustAsRoot`:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\Install-DMS.ps1 -CertificatePath ".\DMS_Publisher.cer" -InstallerPath ".\DMS-Setup-vX.Y.Z.exe"
```

The script installs publisher trust, verifies that the installer is signed by the trusted DMS publisher certificate, then starts the installer.

## Offline or USB Install

Copy the three release files to the target PC, then run the same first-time install command from that folder:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\Install-DMS.ps1 -CertificatePath ".\DMS_Publisher.cer" -InstallerPath ".\DMS-Setup-vX.Y.Z.exe" -TrustAsRoot
```

## App Activation

After installation, open DMS.

If the device is not activated yet:

1. Go to `Settings`.
2. Open the license/status area.
3. Select `Export Request`.
4. Send the generated request file to the developer.
5. Import the approved activation file when you receive it.

New unactivated installs have a 14-day grace period. After grace expires, DMS becomes read-only until activation is imported.

## Updating DMS

After the first install, updates are handled inside DMS:

1. Open DMS.
2. Sign in as an `Admin`.
3. Go to `Settings > Overview > App Updates`.
4. Select `Check for Updates`.
5. Select `Update Now`.

DMS downloads the new installer and `DMS_Publisher.cer`, checks that the installer signer matches the publisher certificate, installs publisher trust for the current Windows user if needed, starts the installer, and closes DMS.

The app does not run `Install-DMS.ps1` during normal updates.

## Troubleshooting

If PowerShell blocks the first-install script, use:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

This only affects the current PowerShell window.

If Windows still warns during first install:

- Make sure `DMS_Publisher.cer` came from the same release as the installer.
- Use `-TrustAsRoot` only for the internal self-signed DMS certificate.
- Do not continue if the script reports a signer mismatch.

If in-app update says `DMS_Publisher.cer` is missing:

- Re-upload `DMS_Publisher.cer` to the same GitHub release as the installer.
- Re-run the release workflow or rebuild with:

```powershell
.\build_windows.ps1 -AppVersion X.Y.Z -Signed
```

If the installer asks for administrator permission, approve it or ask IT/admin staff to install it.

## Developer Release Checklist

Build a signed release:

```powershell
.\build_windows.ps1 -AppVersion X.Y.Z -Signed
```

Upload these files from `Output\vX.Y.Z` to the public release:

```text
DMS-Setup-vX.Y.Z.exe
DMS_Publisher.cer
Install-DMS.ps1
```

Never upload source code, private keys, `.pfx` files, activation private keys, databases, backup files, or the `secrets` folder.
