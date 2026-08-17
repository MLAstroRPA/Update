# MLAstroRPA — Firmware & Web UI Update

Official release repository for the **MLAstroRPA** Robotic Polar Alignment mount. This repository hosts the firmware and Web UI (SPIFFS) packages that the device uses when you run an update.

---

## What's in this repository

| Path | Description |
|---|---|
| `firmware X.Y.Z.bin` | ESP32 application firmware (versioned). |
| `spiffs X.Y.Z.bin` | Web UI assets (`index.html`, `script.js`, `style.css`). |
| `bootloader.bin` | ESP32 bootloader (rarely changes). |
| `partitions.bin` | Partition table with OTA support. |
| `meta.json` | Release metadata consumed by the update tool. |
| `CHANGELOG.md` | Version history and release notes. |
| `docs/` | The update web tool (served via GitHub Pages). |

---

## Update the device (recommended — no tools required)

1. Open the update page in a Chromium browser (Chrome / Edge):

   **https://mlastrorpa.github.io/Update/**

2. Click **🔍 CHECK FOR UPDATES**.
3. Select the version you want to install (Firmware and/or Web UI).
4. Follow the on-screen instructions to install using one of the two paths:
   - **OTA (over Wi-Fi)** — when the device is already connected to your network.
   - **USB Serial (Web Serial)** — connect the device to the PC with a USB cable.

> ⚠️ Do not power off or disconnect the device during the update.

---

## Using the MLAstroRPA Webserver

### 1. Power & connect

- Power on the mount. It either creates an Access Point or joins the saved Wi-Fi network.
- Default Access Point settings:
  - SSID: `MLAstro RPA`
  - Password: `MLAstro RPA`
  - IP: `192.168.4.1`

### 2. Open the Web UI

- Browse to the device IP (e.g. `http://192.168.4.1`).
- The page automatically connects to the WebSocket endpoint `/ws`.

### 3. Basic workflow

1. Set limits and motor parameters.
2. Run calibration / polar-alignment routines.
3. Save settings (`SAVE ALL & REBOOT`).
4. Integrate with a PC app (N.I.N.A / custom) via WebSocket or USB Serial.

### 4. Serial control (PC / N.I.N.A)

- Baud: `115200`, 8 data bits, no parity (`8N1`).
- Start a session with the handshake command:

```text
[MLAstroRPA-TC]
```

- Once Serial control is active, the Web UI is locked to avoid command collision.

---

## Other update methods (advanced)

### USB flash scripts (Windows developers)

Use the `.bat` scripts from the firmware repository:

- `Flash Firmware (BAT)`
- `Flash SPIFFS (BAT)`
- `Flash All (Firmware + SPIFFS) (BAT)`

Flash address mapping:

- Firmware: `0x10000`
- SPIFFS: `0x290000`
- OTA boot data (erase before firmware flash): `0xE000`, size `0x2000`

### PlatformIO CLI (developers)

```bash
# Firmware
platformio run -e upesy_wrover -t upload

# SPIFFS (Web UI)
platformio run -e upesy_wrover -t uploadfs
```

---

## Versioning

- The latest release is **`1.2.58`** (see `CHANGELOG.md` for the full history).
- Firmware and Web UI are released together; install both from the same version.

---

## Support

- Report issues: https://github.com/MLAstroRPA/Update/issues
- Contact: `trong.minh@mlastro.com`
