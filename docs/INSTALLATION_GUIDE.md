# Installation Guide
## AeroCogito H7 Digital Flight Controller - Signed Firmware

**Last Updated:** October 28, 2025

---

## Overview

This guide covers all firmware installation methods for the AeroCogito H7 Digital Flight Controller with signed firmware and secure bootloader.

**Choose your installation method:**
- **[Method A: APJ Update](#method-a-apj-firmware-update)** - Standard firmware update (already have secure bootloader)
- **[Method B: SD Card Update](#method-b-sd-card-firmware-update)** - Update via SD card
- **[Method C: DFU Flash](#method-c-dfu-complete-flash)** - Fresh install or recovery with bootloader

---

## Prerequisites

### Hardware Requirements

- **AeroCogito H7 Digital Flight Controller**
- **USB Cable** (USB-C to USB-A or USB-C)
- **SD Card** (optional, for Method B only)
- **Computer** running Windows, macOS, or Linux

### Software Requirements

Choose based on your installation method:

| Method | Required Software | Download Link |
|--------|------------------|---------------|
| **Method A: APJ** | Mission Planner or QGroundControl | [Mission Planner](https://ardupilot.org/planner/docs/mission-planner-installation.html) / [QGC](https://docs.qgroundcontrol.com/Stable_V5.0/en/qgc-user-guide/getting_started/download_and_install.html) |
| **Method B: SD Card** | File manager only | Built-in to OS |
| **Method C: DFU** | STM32CubeProgrammer or BetaFlight Configurator | [STM32Cube](https://www.st.com/en/development-tools/stm32cubeprog.html) / [BetaFlight](https://app.betaflight.com) |

### Windows-Specific Requirements

**For Method C (DFU mode):**

1. **Install DFU Drivers** - Install STM32 DFU drivers:
   - Download & install: [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) (includes DFU drivers)
   - Or use [ImpulseRC Driver Fixer](https://impulserc.com/pages/downloads)
   - Or use Zadig: [Zadig](https://zadig.akeo.ie/)

### Downloads

Before starting, download firmware from [Releases](https://github.com/AeroCogito/h7-digital-firmware/releases/latest):

- For **Method A**: `arducopter-Copter-X.Y.Z-signed.apj`
- For **Method B**: `arducopter-Copter-X.Y.Z-signed.abin`
- For **Method C**: `arducopter-Copter-X.Y.Z-with-bootloader-signed.hex`
- **Checksums**: `arducopter-Copter-X.Y.Z-SHA256SUMS.txt`

**⚠️ IMPORTANT:** Always [verify checksums](VERIFICATION_GUIDE.md) before installation!

---

## Which Method Should I Use?

### Decision Tree

```
Do you already have AeroCogito secure bootloader installed?
│
├─ YES → Is the board currently running SIGNED ArduPilot?
│   │
│   ├─ YES → Use Method A (APJ Update) ✅ Easiest
│   │
│   └─ NO/UNSURE → Can you access SD card slot?
│       │
│       ├─ YES → Try Method B (SD Card) first
│       │
│       └─ NO → Use Method C (DFU Flash)
│
└─ NO (or coming from BetaFlight/other firmware)
    │
    └─ Use Method C (DFU Flash) ✅ Required for first-time install
```

### Quick Reference

| Situation | Recommended Method |
|-----------|-------------------|
| Regular firmware update | Method A (APJ) |
| No USB connection available | Method B (SD Card) |
| First-time AeroCogito installation | Method C (DFU) |
| Recovering from failed update | Method C (DFU) |
| Board won't boot / bricked | Method C (DFU) |
| Switching from BetaFlight or official ArduPilot | Method C (DFU) |

---

## Method A: APJ Firmware Update

**Use when:** You already have the AeroCogito secure bootloader installed

**Requirements:**
- Secure bootloader already installed
- USB connection
- Mission Planner or QGroundControl

**Time:** 5-10 minutes

### Step 1: Verify Download

```bash
# Linux/macOS
sha256sum -c arducopter-Copter-X.Y.Z-SHA256SUMS.txt

# Windows PowerShell
Get-FileHash arducopter-Copter-X.Y.Z-signed.apj -Algorithm SHA256
```

✅ **Must show "OK" or matching hash before proceeding**

See [Verification Guide](VERIFICATION_GUIDE.md) for details.

### Step 2: Connect Flight Controller

1. Connect flight controller to computer via USB
2. Wait for driver installation (first time only)
3. **Do NOT enter bootloader mode** - connect normally

**LED Behavior:** Solid or slow blinking (normal operation or waiting for firmware)

### Step 3: Install with Mission Planner

#### Mission Planner (Windows)

1. **Open Mission Planner**

2. **Connect to board:**
   - Select COM port (e.g., COM3)
   - Baud rate: 115200
   - Click "Connect"
   - If connection fails, board may be in bootloader - that's OK, continue

3. **Load firmware:**
   - Go to: `Setup → Install Firmware`
   - Click: `Load custom firmware` (bottom right)
   - Browse to downloaded `.apj` file
   - Click "Open"

4. **Upload:**
   - Mission Planner will upload firmware
   - **Progress bar:** Shows upload status
   - **Wait for completion** (1-2 minutes)

5. **Verify:**
   - Look for "Upload Done" or "Done" message
   - Board will automatically reboot
   - LED should start blinking in normal ArduPilot pattern

6. **Connect and verify version:**
   - Reconnect if needed
   - Check firmware version in Mission Planner status bar
   - Should match downloaded version (e.g., Copter-4.5.7)

#### QGroundControl (Cross-Platform)

1. **Open QGroundControl**

2. **Connect board** (should auto-detect)

3. **Flash firmware:**
   - Click vehicle icon (top toolbar)
   - Select: `Vehicle Setup → Firmware`
   - Click: `Advanced settings` (bottom)
   - Check: `Custom firmware file`
   - Click: `Select firmware file`
   - Choose downloaded `.apj` file
   - Click "OK" to start

4. **Upload:**
   - QGC will upload automatically
   - **Do not disconnect** during upload
   - Wait for completion (1-2 minutes)

5. **Verify:**
   - Board reboots automatically
   - QGC should reconnect
   - Check firmware version in parameters (ArduCopter V4.X.Y)

### Step 4: Post-Installation

1. **Verify bootloader accepted firmware:**
   - If board boots normally → ✅ Signature valid
   - If board stays in bootloader (only solid amber on) → ❌ Signature rejected (see [Troubleshooting](TROUBLESHOOTING.md))

2. **Reconnect and configure:**
   - Connect via Mission Planner/QGC
   - Run initial setup wizard (if first ArduPilot install)
   - Verify all parameters loaded correctly

3. **Test flight controller:**
   - Check sensors (accelerometer, gyro, compass)
   - Verify motor outputs (without props!)
   - Confirm radio control

---

## Method B: SD Card Firmware Update

**Use when:** Remote update needed, or USB not available

**Requirements:**
- Secure bootloader already installed
- SD card (FAT32 formatted)
- SD card reader

**Time:** 3-5 minutes + reboot time

### Step 1: Verify Download

```bash
sha256sum -c arducopter-Copter-X.Y.Z-SHA256SUMS.txt
```

✅ **Must verify before proceeding**

### Step 2: Prepare Firmware File

**CRITICAL:** The filename MUST be exactly `ardupilot.abin`

```bash
# Rename downloaded file
mv arducopter-Copter-X.Y.Z-signed.abin ardupilot.abin

# Windows: Right-click → Rename → ardupilot.abin
```

⚠️ **Common mistakes:**
- ❌ `arducopter.abin` (wrong name)
- ❌ `ardupilot.abin.abin` (double extension)
- ❌ Leaving original filename (won't work)

### Step 3: Copy to SD Card

1. **Insert SD card** into computer
2. **Copy file to SD card root** (not in any folder)
3. **Verify:**
   ```
   SD Card Root:
   └── ardupilot.abin  ✅ Correct location

   NOT:
   └── firmware/
       └── ardupilot.abin  ❌ Wrong (in folder)
   ```
4. **Safely eject SD card**

### Step 4: Install on Flight Controller

#### Local Installation

1. **Remove power** from flight controller
2. **Insert SD card** into flight controller
3. **Power on** (connect USB or battery)
4. **Wait ~1 minute:**
   - LED may blink in special pattern
   - Board will reboot automatically when done
5. **Success indicator:**
   - File renamed to `ardupilot-flashed.abin` on SD card
   - Board boots normally

#### Remote Installation (via MAVFTP)

For vehicles already in the field:

1. **Connect via Mission Planner/QGC**
2. **Use MAVFTP** to transfer file:
   ```
   # Mission Planner → Data → Mavlink File Transfer (MAVFTP)
   # Transfer ardupilot.abin to SD card root
   ```
3. **Send reboot command** to trigger update
4. **Wait for reboot and reconnect**

### Step 5: Verification

1. **Check SD card:**
   - File should be renamed to `ardupilot-flashed.abin`
   - If still `ardupilot.abin` → Update failed (see troubleshooting)

2. **Verify firmware version:**
   - Connect via ground station
   - Check version matches downloaded firmware

---

## Method C: DFU Complete Flash

**Use when:** First-time install, recovery, or coming from other firmware (Betaflight or official ArduPilot)

**Requirements:**
- STM32CubeProgrammer or BetaFlight Configurator
- DFU drivers installed (Windows)

**Time:** 10-15 minutes

### Step 1: Verify Download

```bash
sha256sum -c arducopter-Copter-X.Y.Z-SHA256SUMS.txt
```

✅ **Must verify .hex file before proceeding**

### Step 2: Enter DFU Mode

1. **Disconnect USB** from flight controller
2. **Locate BOOT button** on board (front, left edge)
3. **Hold BOOT button**
4. **Connect USB** while holding BOOT
5. **Release BOOT button** after 2 seconds

### Step 3: Verify DFU Mode

If there is only a solid amber LED on, then the board is in DFU mode.

**Windows:**
- Open Device Manager
- Look for: "STM32 BOOTLOADER" under "Universal Serial Bus devices"
- Or: "DFU in FS Mode" under "Universal Serial Bus devices"

**Linux:**
```bash
lsusb | grep STM
# Should show: STM32 BOOTLOADER or DFU Mode
```

**macOS:**
```bash
system_profiler SPUSBDataType | grep STM
```

✅ **If you see DFU device → Proceed to flash**
❌ **If no DFU device → Check drivers (Windows) or try different USB port/cable**

### Step 4A: Flash with STM32CubeProgrammer (Recommended)

1. **Launch STM32CubeProgrammer**

2. **Connect to board:**
   - Connection: `USB` (dropdown)
   - Click `Refresh` button
   - Select USB port showing DFU mode
   - Click `Connect` (button turns green if successful)

3. **Erase flash:**
   - Go to: `Erasing & Programming` tab (left sidebar)
   - **Important:** Check ✅ `Full chip erase`
   - Click `Start erasing`
   - Wait for completion (~10 seconds)

4. **Program firmware:**
   - File path: Click `Browse` → Select `.hex` file
   - Start address: `0x08000000` (should be automatic)
   - ✅ Check `Verify programming`
   - ✅ Check `Skip flash erase` (already erased)
   - ❌ Uncheck `Run after programming` (board will auto-run)
   - Click `Start Programming`

5. **Wait for completion:**
   - Progress bar shows status
   - "File download complete" → Success
   - **Do not disconnect** until complete

6. **Disconnect:**
   - Click `Disconnect` in STM32CubeProgrammer
   - **Disconnect USB** from board
   - **Reconnect USB** (board should boot to ArduPilot)

### Step 4B: Flash with BetaFlight Configurator (Alternative)

1. **Launch BetaFlight Configurator**

2. **Firmware Flasher tab:**
   - Click `Firmware Flasher` (left sidebar)

3. **Configure:**
   - **Uncheck** "Auto-connect"
   - **Check** "Full chip erase"
   - Click `Load Firmware [Local]` (bottom right)

4. **Select firmware:**
   - Browse to downloaded `.hex` file
   - Click "Open"

5. **Flash:**
   - Click `Flash Firmware`
   - **Do not disconnect** during flash
   - Wait for "Programming: SUCCESSFUL" (2-3 minutes)

6. **Disconnect and reboot:**
   - Unplug USB
   - Reconnect → Board should boot to ArduPilot

### Step 5: Verify Secure Bootloader Installed

After flashing, verify the secure bootloader is active:

#### Windows

1. **Disconnect and reconnect USB**

2. **Device Manager:**
   - Look for device name
   - Should contain: `AeroCogito-H7Digital-Secure-BL-v10`
   - "**Secure**" in name = bootloader is active ✅

#### Linux

```bash
# Reconnect board in bootloader mode
dmesg | grep "Product:"

# Expected output:
# Product: AeroCogito-H7Digital-Secure-BL-v10
# Look for "Secure-BL" ✅
```

**NOTE**: this might be momentary, so it is difficult to catch before the board enters the main firmware, **BUT** if ArduPilot loads after flashing the signed firmware, then the secure bootloader has successfully validated the signed firmware and is operating normally.

### Step 6: Post-Installation Setup

1. **Connect via Mission Planner or QGC**

2. **Run Initial Setup Wizard:**
   - Frame type selection
   - Accelerometer calibration
   - Compass calibration
   - Radio calibration
   - ESC calibration

3. **Configure parameters:**
   - Set up failsafes
   - Configure flight modes
   - Tune PID values (if needed)

4. **Test all systems:**
   - Sensors (accel, gyro, baro, compass)
   - Motor outputs (**no props!**)
   - Radio control
   - GPS (if installed)

---

## Post-Installation Checklist

After any installation method:

- [ ] Firmware version matches downloaded version
- [ ] All sensors detected and calibrated
- [ ] Radio control responds correctly
- [ ] Motor outputs work (tested without props)
- [ ] Flight modes switch correctly
- [ ] Failsafe configured and tested
- [ ] Compass calibration completed (if using GPS)
- [ ] Pre-arm checks pass

---

## Installation Troubleshooting

For installation issues, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md):

- DFU mode not detected
- Signature verification failed
- Upload errors
- Bootloader recovery
- Bricked board recovery

---

## Additional Resources

- **[Verification Guide](VERIFICATION_GUIDE.md)** - Verify firmware before installation
- **[Troubleshooting Guide](TROUBLESHOOTING.md)** - Common installation issues
- **[Security Policy](../SECURITY.md)** - Compliance and security details
- **[ArduPilot First-Time Setup](https://ardupilot.org/copter/docs/initial-setup.html)** - Official ArduPilot setup guide
- **[Mission Planner Docs](https://ardupilot.org/planner/)** - Mission Planner documentation
- **[QGroundControl User Guide](https://docs.qgroundcontrol.com/)** - QGC documentation

---

## Support

**Installation help:**
- Email: support@aerocogito.com

**Security concerns:**
- Email: security@aerocogito.com

---

**Document Version:** 1.0
**Last Updated:** October 28, 2025
