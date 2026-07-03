# ESP32 Marauder CYD

ESP32 Marauder build customized for the Cheap Yellow Display ESP32-2432S028R / CYDUSB2 with ILI9341 display, XPT2046 resistive touch, GPS, SD logging, dark UI, and expanded passive Flock detection/survey tools.

This repo is a local CYD-focused PlatformIO workspace. It intentionally keeps the CYD-specific fork separate from the generic JustCallMeKoko ESP32 Marauder source.

## Credits

This project is a fork/custom build based on:

- [Fr4nkFletcher/ESP32-Marauder-Cheap-Yellow-Display](https://github.com/Fr4nkFletcher/ESP32-Marauder-Cheap-Yellow-Display), credited for the Cheap Yellow Display Marauder port and CYD hardware support.
- [justcallmekoko/ESP32Marauder](https://github.com/justcallmekoko/ESP32Marauder), credited for the original ESP32 Marauder project.
- [0xXyc/flock-you-wifi-recon](https://github.com/0xXyc/flock-you-wifi-recon), credited as research inspiration for the additional Flock-oriented WiFi reconnaissance ideas and signatures.

The custom changes in this repo are focused on making the CYDUSB2 build reproducible in VS Code + PlatformIO and improving the small-screen field workflow.

## Target Hardware

Primary target:

- Cheap Yellow Display ESP32-2432S028R / CYDUSB2
- ILI9341 240x320 TFT
- XPT2046 resistive touch
- MicroSD card slot
- Optional GY-NEO6MV2 / NEO-6M GPS module

Tested local setup:

- Board environment: `cydusb2`
- USB upload port: `COM6`
- Serial monitor speed: `115200`
- Upload speed: `460800`
- PlatformIO platform: `espressif32@6.5.0`
- Arduino ESP32 core: `2.0.14`

## Important Hardware Notes

### GPS

This build was tested with a GY-NEO6MV2 / NEO-6M GPS module.

On this CYDUSB2 unit, the working GPS receive path was detected on UART0 RX / GPIO3. The firmware includes probing logic that can detect the attached GPS and report it in serial output.

Practical note: the GPS shares the serial path in a way that can interfere with flashing. For reliable uploads, unplug the GPS before uploading firmware, then reconnect it after the upload completes.

### SD Card

The SD card is used for capture/log output. The on-screen SD status icon should show:

- Red when no SD card is installed
- Green when the SD card is detected

For large files such as PCAP, KML, GPX, or CSV logs, pulling the SD card and reading it on a PC is currently the easiest export path.

## PlatformIO Build

This repo is set up to use PlatformIO-managed/local project dependencies only.


Build:

```powershell
& "$env:USERPROFILE\.platformio\penv\Scripts\platformio.exe" run -e cydusb2
```

Upload:

```powershell
& "$env:USERPROFILE\.platformio\penv\Scripts\platformio.exe" run -e cydusb2 -t upload --upload-port COM6
```

Serial monitor:

```powershell
& "$env:USERPROFILE\.platformio\penv\Scripts\platformio.exe" device monitor -p COM6 -b 115200
```

If upload fails with a timeout while the GPS is connected, unplug the GPS, reset/boot the device if needed, and upload again.

## Current `platformio.ini`

The active environment is `cydusb2`.

Key settings:

```ini
[platformio]
default_envs = cydusb2
src_dir = esp32_marauder
lib_dir = pio-libs

[env:cydusb2]
platform = espressif32@6.5.0
board = esp32dev
framework = arduino
board_build.partitions = min_spiffs.csv
board_build.flash_mode = dio
board_build.f_flash = 80000000L
monitor_speed = 115200
upload_speed = 460800
lib_ldf_mode = deep+
lib_compat_mode = off
```

The display/touch build flags are configured for the CYDUSB2 ILI9341 + XPT2046 pinout.

## Custom Features In This Build

### CYD UI Improvements

- Dark/black menu background
- Higher-contrast text
- Reduced white flash during menu transitions
- Avoids unnecessary TFT reinitialization on CYD menu returns
- Flock summary stays on screen until touch instead of disappearing after a short delay

### Touch And Device Diagnostics

Added small-device friendly helper screens:

- Diagnostics screen
- Touch test screen

These are meant to make field troubleshooting easier on the small CYD display.

### GPS Support

GPS support has been tested with a NEO-6M module. The build can detect GPS data and supports GPS-aware Flock/Wardrive workflows.

Observed working behavior:

- GPS detected at boot
- Live NMEA data visible
- GPS menu reports satellites and fix status
- Flock Survey and Flock WiFi Wardrive appear when GPS lock is available

### Flock-Oriented Passive Recon

This build adds/expands passive Flock-oriented discovery features intended for authorized research and mapping.

Included items:

- Flock Sniff menu item
- Flock Survey menu item for combined BLE + WiFi field checks
- Flock WiFi Wardrive menu item for WiFi-only review/export runs
- WiFi PCAP capture support for Flock sniff sessions
- CSV/Wigle-style logging through the existing Marauder buffer path
- CSV review logs for matched and rejected WiFi observations
- KML hit export
- GPX track export
- BLE hit export to CSV/KML/GPX when GPS/SD are available
- Raw WiFi Flock hit export from Flock Sniff sessions
- Session summary screen
- Unique device counters
- Separate BLE and WiFi hit counters
- GPS-aware dashboard values
- Robust tagged SSID parser
- Probe request support
- Probe response support
- Beacon support
- Hidden SSID review flag
- Confidence labels for hits
- XUNTONG manufacturer-data BLE matching
- Numeric Penguin battery-name matching
- Known Flock WiFi OUI and SSID pattern matching

Flock-related modes:

- `Flock Survey`: recommended driving mode. Runs WiFi AP checks and BLE Flock matching in one GPS-aware session, then exports hits and review data.
- `Flock Sniff`: passive BLE plus raw WiFi management-frame sniffing. Useful when parked near a known camera for a control test.
- `Flock WiFi Wardrive`: WiFi-only wardrive/review mode. Useful for checking visible APs, but not sufficient by itself when cameras do not advertise WiFi.

The Flock dashboard counters count actual matches only. Non-matching WiFi observations are still written to the review CSV, but they do not increment Flock hit totals.

Flock export files use mode-specific names:

- `flock_survey_*.kml`
- `flock_survey_track_*.gpx`
- `flock_survey_review_*.csv`
- `flock_wifi_wardrive_*.kml`
- `flock_wifi_wardrive_track_*.gpx`
- `flock_wifi_wardrive_review_*.csv`
- `flock_sniff_*.kml`
- `flock_sniff_track_*.gpx`
- `flock_sniff_review_*.csv`

Flock CSV rows and session filenames use US Eastern local time derived from GPS UTC. GPX timestamps remain UTC/Zulu so mapping tools interpret tracks correctly.

The Flock additions are passive detection/logging features. This repo does not add exploit/RCE behavior.

## Flock Workflow

Recommended field test flow:

1. Insert a working microSD card.
2. Power on the CYD.
3. Wait for GPS lock.
4. Start `Flock Survey` for normal driving.
5. Tap the screen to stop.
6. Review the summary screen.
7. Tap again to return to the menu.
8. Pull the SD card to inspect CSV/KML/GPX/PCAP output.

Recommended driving notes:

- Use `Flock Survey` first. It has the widest passive coverage.
- Keep the device near a window with a clear GPS/radio view.
- Drive slowly and, when safe, make more than one pass.
- Let the scan run before and after passing the camera because BLE advertisements can be bursty.
- Use `Flock Sniff` while parked near a known camera for a 2-5 minute control test.
- Use `Flock WiFi Wardrive` only when you specifically want WiFi AP review data.

For best validation, compare:

- Whether GPS coordinates appear in exported files
- Whether unique device counts are reasonable
- Whether timestamps/session names are consistent
- Whether KML/GPX output imports cleanly into mapping tools
- Whether review CSV rows are `MATCH` or `REVIEW`
- Whether KML contains hit placemarks in addition to track placemarks


## Safety And Legal Notice

This project is for educational, defensive, and authorized research use only.

Only test on devices, networks, and locations where you have permission. Local laws and policies may apply to wireless scanning, packet capture, and signal collection.

## Status

Known working on the local CYDUSB2 setup:

- Display
- Resistive touch
- Black UI theme
- SD card detection
- GPS detection and lock
- Flock Sniff menu
- Flock Survey menu
- Flock WiFi Wardrive menu
- BLE Flock hit export to CSV/KML/GPX
- WiFi review CSV with non-match rows preserved
- Faster PlatformIO upload at `460800`

Known inconvenience:

- GPS should be unplugged before uploading firmware over USB.
