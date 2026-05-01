# DMS Installation Guide

This guide is for installing DMS on a Windows PC from the public release repository.

## What You Need

- Windows 10 or Windows 11.
- The latest `DMS-vX.Y.Z-release.zip`.
- An internet connection for GitHub download and in-app updates.
- Administrator approval may be required when the installer writes to `Program Files`.

## First-Time Install

Download `DMS-vX.Y.Z-release.zip` from the latest public GitHub release and extract it.

For the easiest install, double-click:

```text
Install-DMS.cmd
```

If Windows shows an `Open File - Security Warning` for `Install-DMS.cmd`, select `Run`. The launcher removes the downloaded-from-internet warning from the extracted DMS files, asks for administrator approval, installs publisher trust, verifies that the installer is signed by the trusted DMS publisher certificate, then starts the installer. If Windows blocks machine-wide certificate trust, the launcher retries with current-user trust automatically.

The extracted folder should contain:

```text
DMS-Setup-vX.Y.Z.exe
Install-DMS.cmd
_DMS_Setup_Files\
```

The `_DMS_Setup_Files` folder contains the certificate and PowerShell helper used by the launcher. Users should leave that folder in place.

## PowerShell Install

If the one-click launcher cannot be used, open PowerShell in the extracted folder.

For the internal self-signed DMS certificate, run PowerShell as Administrator:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\_DMS_Setup_Files\Install-DMS.ps1 -CertificatePath ".\_DMS_Setup_Files\DMS_Publisher.cer" -InstallerPath ".\DMS-Setup-vX.Y.Z.exe" -StoreScope LocalMachine -TrustAsRoot
```

For a public CA-issued certificate, run without `-TrustAsRoot`:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\_DMS_Setup_Files\Install-DMS.ps1 -CertificatePath ".\_DMS_Setup_Files\DMS_Publisher.cer" -InstallerPath ".\DMS-Setup-vX.Y.Z.exe"
```

The script performs the same checks as `Install-DMS.cmd`.

## Offline or USB Install

Copy the extracted release folder to the target PC, then double-click:

```text
Install-DMS.cmd
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

If the one-click launcher opens and closes quickly, right-click `Install-DMS.cmd` and choose `Run as administrator`, or open it from an Administrator Command Prompt to see the message.

If Windows still warns during first install:

- Make sure `DMS_Publisher.cer` came from the same release as the installer.
- Use `-TrustAsRoot` only for the internal self-signed DMS certificate.
- Do not continue if the script reports a signer mismatch.
- If SmartScreen still appears after using `Install-DMS.cmd`, the installer may still be signed by an old/untrusted certificate. Rebuild with a PFX whose certificate is `CN=LCS Software Systems`.
- If the script reports `Access is denied` while installing to `Cert:\LocalMachine`, use the latest `Install-DMS.cmd`; it falls back to `Cert:\CurrentUser`.

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
Install-DMS.cmd
Install-DMS.ps1
DMS-vX.Y.Z-release.zip
```

The manual ZIP should show only the launcher and installer at the top level:

```text
DMS-Setup-vX.Y.Z.exe
Install-DMS.cmd
_DMS_Setup_Files\
```

Never upload source code, private keys, `.pfx` files, activation private keys, databases, backup files, or the `secrets` folder.
