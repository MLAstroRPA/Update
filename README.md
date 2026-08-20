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

The Web UI is split into two tabs:

- **🎮 CONTROL** — everything you use during a session: moving the mount, monitoring, polar alignment, and logs.
- **🛠️ CONFIG** — all limits, motor, network, calibration and save actions.

A normal session follows the **MAN → AUTO → SAFETY → CONFIG & SAVE** workflow below.

#### 3.1 MAN — Manual movement (🎮 CONTROL → 🕹️ Manual Movement)

1. Pick a **Speed Level (1–5)** — higher levels move faster.
2. Choose a move mode:
   - **Jog (Hold)** — press and hold an arrow to move continuously; release to stop.
   - **Relative (Step)** — set a step size in degrees / arcminutes / arcseconds, then press an arrow once to move exactly that amount.
3. Use the directional pad: **▲/▼ Altitude**, **◄/► Azimuth**, **⏹** to stop.
4. In an emergency, hit the red **FORCE STOP** button in the header — it halts both axes immediately.

The **📐 Position** panel shows current Azimuth/Altitude (from home), step counters, output speed and the **Homed** status. From there you can:

- **🏠 SET HOME HERE** — mark the current position as home.
- **↻ RETURN TO HOME** — move both axes back to home.
- **⚠️ RESET HOME** — clear the home reference.

#### 3.2 AUTO — Polar alignment (🎮 CONTROL → 🎯 Polar Alignment)

1. In **Alt Error** / **Az Error**, enter the measured error in Deg / Min / Sec.
2. Choose the correction direction: **Up/Down** for Alt, **Left/Right** for Az.
3. Click **Align Alt** or **Align Az** to correct a single axis, or **✓ ALIGN ALL** to correct both at once.
4. Watch **Azimuth Moved** / **Altitude Moved** to see the applied correction.

**Automatic calibration** (used to compute exact `steps/degree` after mechanics or microstep changes):

- Go to **🛠️ CONFIG → 🔑 Admin Config → Travel Calibration**.
- Set the **Travel Angle** for Azimuth (default 20°) and Altitude (default 30°).
- Click **AZ Calib**, **ALT Calib** or **Calib All**. The axis travels to both hard limits — **make sure the path is clear**.
- When the result appears, either **Apply Steps/Deg Only** or **Apply result & Auto center** (moves back to center and sets home).
- **Sensorless Auto Tuning** (same area) auto-configures the StallGuard thresholds per speed level (`AZ/ALT SGTHRS`) and per microstep (`AZ/ALT TCOOL`) instead of entering values by hand.

> Calibration requires **Hard Limits to be enabled first** — the Web UI warns you if they are not.

#### 3.3 SAFETY — Limits & monitoring

- **📏 Soft Limits** (🛠️ CONFIG): enable and set the AZ (±9°) and ALT (±14°) travel range in degrees. The mount refuses moves outside this range.
- **🛡️ Hard Limits (Sensorless / StallGuard)** (🛠️ CONFIG): enter the **SGTHRS** value for each speed level (1–5) and **TCOOLTHRS** for each microstep on both axes, set **Debounce Time** and **Escape Rotations**, then tick **Enable Hard Limit**. When a stall is detected the axis stops and backs off.
- **📈 Hardlimit Monitor** (🎮 CONTROL): live chart of AZ/ALT StallGuard load and motor current while moving — great for verifying threshold values (enable **Show hardlimit monitor** in Admin Config).
- Header **FORCE STOP** and **⚠️ RESET ERROR** (in **📋 System Log**) let you stop and clear error states after a limit trip.
- In **🔑 Admin Config**, keep **Enable Communication Watchdog** on so the mount performs an E-STOP if the control app loses the heartbeat.

#### 3.4 CONFIG & SAVE (🛠️ CONFIG tab)

- **⚙️ Motor Driver (TMC2209)** — per-axis **Run/Hold current**, **Start-up Booster**, **Soft CoolStep**, **Microsteps**, **Accel/Decel**, **Steps/Degree**, **StealthChop / SpreadCycle** mode and **Reverse Direction**.
- **↔️ Backlash & P.A Overshoot** — anti-backlash compensation in steps, and the Alt P.A overshoot (direction + amount) used by the two-leg alignment move.
- **📶 WiFi Configuration** — Access Point (SSID/password/IP) and Station mode (connect to your router, scan for networks, see connected clients).
- **🔑 Admin Config** — serial port settings, swap Az-Alt motor ports (reboot required), show hardlimit monitor / steps, factory zero, max motor RPM, sensorless auto tuning, travel calibration, and password change.

Finally, use **💾 Configuration Management** to commit your changes:

| Button | What it does |
|---|---|
| **⚡ APPLY SETTINGS** | Apply to RAM only — lost on reboot. Good for quick tests. |
| **✓ SAVE ALL & REBOOT** | Persist everything to FRAM and reboot. Use this to keep changes. |
| **⏻ REBOOT** | Reboot without saving (discards unapplied changes). |
| **⚠️ FACTORY RESET** | Restore factory defaults. |

#### 3.5 First-time setup — quick sequence

1. Open the Web UI and go to **🛠️ CONFIG**.
2. Set **Soft Limits** (and **Hard Limits** if using sensorless StallGuard).
3. Verify **Motor Driver** parameters; if `Steps/Degree` is unknown, run **Travel Calibration** and **Apply result & Auto center**.
4. Click **✓ SAVE ALL & REBOOT** to persist.
5. Use **🎮 CONTROL → 🕹️ Manual Movement** (or **Return to Home**) to position the mount, then apply corrections with **🎯 Polar Alignment**.
6. When aligned, click **🏠 SET HOME HERE**, then **✓ SAVE ALL & REBOOT**.
7. Optionally integrate with **N.I.N.A / custom software** via USB Serial (see section 4) or the WebSocket API.

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

- The latest release is **`1.2.61`** (see `CHANGELOG.md` for the full history).
- Firmware and Web UI are released together; install both from the same version.

---

## Support

- Report issues: https://github.com/MLAstroRPA/Update/issues
- Contact: `trong.minh@mlastro.com`
