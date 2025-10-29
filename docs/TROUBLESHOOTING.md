# Troubleshooting Guide
## AeroCogito H7 Digital Flight Controller

**Last Updated:** October 28, 2025

---

## Quick Navigation

- [DFU Mode Issues](#dfu-mode-issues)
- [Firmware Upload Errors](#firmware-upload-errors)
- [Signature Verification Failures](#signature-verification-failures)
- [Bootloader Issues](#bootloader-issues)
- [Connection Problems](#connection-problems)
- [SD Card Update Issues](#sd-card-update-issues)
- [Recovery Procedures](#recovery-procedures)

---

## DFU Mode Issues

### Problem: Board Not Entering DFU Mode

**Symptoms:**
- Holding BOOT button doesn't enter DFU
- No DFU device shown in Device Manager/lsusb
- Board boots normally instead of entering bootloader

**Solutions:**

#### Solution 1: Check BOOT Button Timing

```
Correct sequence:
1. Disconnect USB completely
2. Press and HOLD BOOT button
3. Connect USB while holding
4. Wait 2-3 seconds
5. Release BOOT button
```

**Common mistakes:**
- ❌ Pressing BOOT after connecting USB (too late)
- ❌ Releasing BOOT too quickly (before connection established)
- ❌ Not disconnecting USB first (board already running)

#### Solution 2: Try Different USB Port/Cable

- **Use USB 2.0 port** (not USB 3.0) if available
- **Try different cable** - some cables are **charge-only**
- **Connect directly to computer** (not through USB hub)

#### Solution 3: Verify Board Power

```bash
# Check if board is receiving power
# LED should light up when USB connected

No LED at all:
- Try different USB port
- Check USB cable
- Board may be damaged (contact support)
```

---

### Problem: DFU Device Detected But Can't Connect

**Windows:**

#### Solution 1: Install/Update DFU Drivers

1. **Download Zadig:** https://zadig.akeo.ie/
2. **Run Zadig as Administrator**
3. **Options → List All Devices**
4. **Select:** "STM32 BOOTLOADER" or "DFU in FS Mode"
5. **Driver:** Select "WinUSB" or "libusb-win32"
6. **Click:** "Replace Driver" or "Install Driver"
7. **Wait for completion**
8. **Reconnect board**

#### Solution 2: Install ImpulseRC Driver Fixer

1. **Download and run**: [ImpulseRC Driver Fixer](https://impulserc.com/pages/downloads)

#### Solution 3: Install STM32 DFU Drivers as part of STM32CubeProgrammer

1. **Download:** [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)
2. **Extract and run installer**
3. **Reboot computer**
4. **Reconnect board in DFU mode**

**Linux:**

```bash
# Add udev rules for STM32 DFU access
sudo nano /etc/udev/rules.d/99-stm32.rules

# Add this line:
SUBSYSTEM=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="df11", MODE="0666"

# Save and reload rules
sudo udevadm control --reload-rules
sudo udevadm trigger

# Reconnect board
```

**macOS:**

```bash
# Usually works without additional drivers
# If issues, try:
brew install libusb

# Reconnect board
```

---

## Firmware Upload Errors

### Problem: "Upload Failed" in Mission Planner

**Error message:** "Upload failed" or "Communication error"

**Solutions:**

#### Solution 1: Close Other Programs

```
Close these programs before upload:
- QGroundControl
- Other instances of Mission Planner
- Serial monitor programs
- Arduino IDE
- PuTTY or other terminal programs
```

#### Solution 2: Reboot Upload Process

```
1. Disconnect USB from board
2. Close Mission Planner completely
3. Wait 10 seconds
4. Reconnect USB
5. Reopen Mission Planner
6. Try upload again
```

#### Solution 3: Update Mission Planner

```
Mission Planner → Help → Check for Updates

Minimum required version: 1.3.70+
```

#### Solution 4: Try Different Upload Method

- If Mission Planner fails → Try QGroundControl
- If QGC fails → Try Mission Planner
- If both fail → Use Method C (DFU flash)

---

### Problem: STM32CubeProgrammer Connection Failed

**Error:** "Cannot connect to device" or "No device found"

**Solutions:**

#### Solution 1: Verify DFU Mode

```
STM32CubeProgrammer → Refresh button

Should show:
Port: USB1 or similar
Serial number: (hexadecimal)

If blank → Board not in DFU mode
```

#### Solution 2: Update STM32CubeProgrammer

- Current version should be 2.14.0 or newer
- Download from: https://www.st.com/en/development-tools/stm32cubeprog.html
- Uninstall old version first

#### Solution 3: Run as Administrator (Windows)

```
Right-click STM32CubeProgrammer
→ Run as Administrator
```

#### Solution 4: Check USB Settings

```
In STM32CubeProgrammer:
1. Connection dropdown → USB
2. Click "Refresh"
3. Port dropdown → Select available port
4. Click "Connect"

If fails:
- Try different USB port
- Reconnect board in DFU mode
```

---

## Signature Verification Failures

### Problem: Bootloader Stays in Bootloader Mode (Won't Boot Firmware)

**Symptoms:**
- Uploaded firmware successfully
- Board doesn't boot into main firmware (only amber LED is on)

**Cause:** Firmware signature doesn't match bootloader's public key

**Solutions:**

#### Solution 1: Verify You Downloaded Correct Firmware

```bash
# Check filename contains "-signed"
ls -l arducopter-*.apj

Correct:
✅ arducopter-Copter-4.5.7-signed.apj

Wrong:
❌ arducopter-Copter-4.5.7.apj (not signed)
❌ arducopter.apj (wrong source)
```

#### Solution 2: Verify Checksum

```bash
sha256sum -c arducopter-Copter-X.Y.Z-SHA256SUMS.txt

If FAILED:
- Re-download firmware
- Download may have been corrupted
```

#### Solution 3: Check Public Key Match

**The firmware must be signed with the same key embedded in bootloader**

```bash
# Verify public key fingerprint
sha256sum keys/AeroCogito_public_key.dat | awk '{print $1}' | fold -w2 | paste -sd':' -

# Should match:
e3:37:0c:eb:9f:e4:78:de:fd:82:ba:a7:79:8c:4e:dc:95:be:3e:3b:95:fd:ca:ac:07:06:22:f2:f6:1b:16:91

If different → You have wrong public key or wrong firmware
```

#### Solution 4: Reflash Complete System (Bootloader + Firmware)

```
If signature keeps failing:
1. Download: arducopter-Copter-X.Y.Z-with-bootloader-signed.hex
2. Use Method C (DFU flash)
3. This includes matching bootloader and firmware
```

---

### Problem: "Checksum Failed" When Verifying Download

**Error:** `sha256sum: WARNING: 1 computed checksum did NOT match`

**Solutions:**

#### Solution 1: Re-Download Firmware

```
Cause: Download was corrupted

Fix:
1. Delete downloaded file
2. Clear browser cache
3. Re-download from GitHub Releases
4. Verify checksum again
```

#### Solution 2: Verify Downloaded Correct Files

```bash
# Make sure firmware and checksum file are from SAME release

Check tags match:
✅ arducopter-Copter-4.5.7-signed.apj
✅ arducopter-Copter-4.5.7-SHA256SUMS.txt
   (both say 4.5.7)

❌ arducopter-Copter-4.5.7-signed.apj
❌ arducopter-Copter-4.5.6-SHA256SUMS.txt
   (versions don't match)
```

#### Solution 3: Check File Not Modified

```
After download, don't:
❌ Edit/rename the file
❌ Open in text editor
❌ Compress/decompress
❌ Upload to cloud storage (may modify)

Download directly → Verify → Install
```

---

## Bootloader Issues

### Problem: Can't Tell If Secure Bootloader Is Installed

**How to check:**

#### Windows Method

```
1. Connect board via USB (in bootloader mode)
   - To force bootloader: Upload unsigned firmware (Install Method A or B with official ArduPilot builds)

2. Open Device Manager
   - Universal Serial Bus devices
   - Look for device name

Should show:
✅ "AeroCogito-H7Digital-Secure-BL-v10"
   (contains "Secure-BL")

Without secure bootloader:
❌ "AeroCogito-H7Digital-BL-v10"
   (no "Secure")
```

#### Linux Method

```bash
# With board in bootloader mode
dmesg | tail -20 | grep "Product:"

Expected:
✅ Product: AeroCogito-H7Digital-Secure-BL-v10

Without secure bootloader:
❌ Product: AeroCogito-H7Digital-BL-v10
```

#### Behavior Method

```
Upload unsigned firmware (.apj or .abin from standard ArduPilot):

With secure bootloader:
✅ Board stays in bootloader (only amber LED on)
✅ Won't boot unsigned firmware

Without secure bootloader:
❌ Board boots unsigned firmware normally
❌ No signature checking
```

---

### Problem: Need to Remove Secure Bootloader

**Use case:** Want to go back to unsigned firmware (official ArduPilot or Betaflight)

**Warning:** ⚠️ This removes signature verification security

**Procedure:** See [Installation Guide](INSTALLATION_GUIDE.md#method-c-dfu-complete-flash) - use Method C (DFU) with an official ArduPilot or BetaFlight `*.hex` release (without secure bootloader)

Other methods (see [Reverting to normal boot](https://github.com/ArduPilot/ardupilot/tree/master/Tools/scripts/signing#reverting-to-normal-boot)):
- Private key (for secure command)
- MAVProxy with SecureCommand module

[Maintainer Guide - Reverting to Normal Boot](MAINTAINER_GUIDE.md#disaster-recovery)

---

## Connection Problems

### Problem: Mission Planner Won't Connect

**Error:** "No Heartbeat" or "Connection failed"

**Solutions:**

#### Solution 1: Check COM Port

```
Windows:
- Device Manager → Ports (COM & LPT)
- Look for: "USB Serial Device (COMX)"
- Use that COM port in Mission Planner

Linux:
ls /dev/ttyACM* /dev/ttyUSB*
# Usually /dev/ttyACM0

macOS:
ls /dev/cu.*
# Usually /dev/cu.usbmodem*
```

#### Solution 2: Set Correct Baud Rate

```
Mission Planner:
- Baud rate: 115200 (default)
- Or try: 57600

QGroundControl:
- Usually auto-detects
```

#### Solution 3: Reboot Board

```
1. Disconnect USB
2. Wait 10 seconds
3. Reconnect USB
4. Try connecting again
```

#### Solution 4: Check USB Cable

```
Try different cable:
- Some cables are **charge-only** (no data)
- Cable may be damaged
```

---

### Problem: Board Connects Then Immediately Disconnects

**Symptoms:**
- Brief connection, then drops
- "Heartbeat Lost" message
- Reconnects in loop

**Solutions:**

#### Solution 1: USB Power Issues

```
Symptom: Board resets when connecting

Fix:
- Use powered USB hub
- Connect to different USB port (better power)
- Use external power source (battery) + USB data only
```

#### Solution 2: Parameter Corruption

```
1. Connect via USB
2. Mission Planner → Config → Full Parameter List
3. Click "Load from File"
4. Select default parameters for ArduCopter
5. Write parameters
6. Reboot
```

#### Solution 3: Firmware Mismatch

```
If recently updated firmware:

1. Do full parameter reset:
   - Mission Planner → Config → Full Parameter Reset
2. Reboot board
3. Reconnect and reconfigure from scratch
```

---

## SD Card Update Issues

### Problem: SD Card Update Not Working

**Symptoms:**
- Copied ardupilot.abin to SD card
- Power cycled board
- File still named ardupilot.abin (not renamed to ardupilot-flashed.abin)
- Firmware not updated

**Solutions:**

#### Solution 1: Verify Filename Exactly

```
MUST be exactly:
✅ ardupilot.abin

NOT:
❌ arducopter.abin
❌ ardupilot.abin.abin (double extension)
❌ ARDUPILOT.ABIN (wrong case on Linux)
❌ ardupilot .abin (space in name)

Check:
ls -la /path/to/sd/card
```

#### Solution 2: Verify File Location

```
File must be in ROOT of SD card:

Correct:
✅ /ardupilot.abin

Wrong:
❌ /firmware/ardupilot.abin
❌ /DCIM/ardupilot.abin
❌ /ArduPilot/ardupilot.abin
```

#### Solution 3: Check SD Card Format

```
SD card must be FAT32:

Check format:
- Windows: Right-click SD drive → Properties
- Linux: sudo fdisk -l /dev/sdX
- macOS: diskutil info /dev/diskX

If not FAT32:
1. Backup data
2. Format as FAT32
3. Copy ardupilot.abin again
```

#### Solution 4: Verify Secure Bootloader Installed

```
SD card update requires secure bootloader

Check:
- See "Bootloader Issues" section above
- If no secure bootloader → Use Method C (DFU) first
```

#### Solution 5: Check File Size

```bash
ls -lh ardupilot.abin

Should be ~2-5 MB:
✅ 3.6M ardupilot.abin

Too small/large indicates problem:
❌ 100K ardupilot.abin (incomplete download)
❌ 10M ardupilot.abin (wrong file)
```

---

## Recovery Procedures

### Recovery 1: Bricked Board (Won't Boot, No USB)

**Symptoms:**
- No LED when powered
- No USB device detected
- Completely unresponsive

**Solutions:**

#### Step 1: Check Power

```
1. Try different USB cable
2. Try different USB port
3. Measure voltage on 4.5V pad (should be ~5V)
4. Try external power source (battery connector)
```

#### Step 2: If Still No Response

```
Hardware failure possible

Contact support:
- Email: support@aerocogito.com
- Include: Purchase date, what happened before brick
```

---

### Recovery 2: Bootloader Loop (Board Won't Boot Firmware)

**Symptoms:**
- Only solid amber LED is on
- Stays in bootloader forever
- USB shows as bootloader device

**Solutions:**

#### Quick Fix: Reflash Complete System

```
1. Download: arducopter-Copter-X.Y.Z-with-bootloader-signed.hex
2. Enter DFU mode (hold BOOT button)
3. Use STM32CubeProgrammer:
   - Full chip erase
   - Program .hex file to 0x08000000
4. Reboot
```

Should fix:
- Signature mismatches
- Corrupted bootloader
- Firmware/bootloader incompatibility

---

### Recovery 3: Wrong Firmware Installed

**Symptoms:**
- Installed firmware for wrong board
- Board behaves incorrectly
- Sensors not working

**Solutions:**

```
1. Download CORRECT firmware:
   - Must be for AeroCogito-H7Digital
   - Correct vehicle type (Copter, not Plane/Rover)

2. Reflash:
   - Use Method A (APJ) if board still boots
   - Use Method C (DFU) if board won't boot

3. Reset parameters:
   - Mission Planner → Full Parameter Reset
   - Reconfigure from scratch
```

---

### Recovery 4: Accidentally Installed Unsigned Firmware

**Symptoms:**
- Used standard ArduPilot firmware (not AeroCogito signed)
- Board stuck in bootloader
- Won't boot

**Solution:**

```
Secure bootloader is working correctly - rejecting unsigned firmware

Fix:
1. Download signed firmware:
   arducopter-Copter-X.Y.Z-signed.apj

2. Board is already in bootloader mode (good)

3. Upload via Mission Planner:
   - Connect to bootloader COM port
   - Load custom firmware
   - Select signed .apj file

Board will boot once signed firmware uploaded
```

---

## Common Error Messages

### "Failed to verify signature"

**Cause:** Firmware signature doesn't match bootloader public key

**Fix:** Download firmware from official AeroCogito releases only

---

### "Error erasing flash"

**Cause:** STM32CubeProgrammer can't erase memory

**Fix:**
1. Disconnect board
2. Reconnect in DFU mode
3. Try "Full chip erase" before programming

---

### "Upload timeout"

**Cause:** Communication interrupted during upload

**Fix:**
1. Use shorter USB cable (< 3 feet)
2. Connect directly to computer (not hub)
3. Close other programs using serial ports

---

### "Invalid firmware format"

**Cause:** Wrong file type uploaded

**Fix:**
- Method A (Mission Planner) → Use .apj file
- Method B (SD card) → Use .abin file
- Method C (DFU) → Use .hex file

---

## Getting Help

### Before Contacting Support

1. **Check this troubleshooting guide**
2. **Verify firmware checksum** (see [Verification Guide](VERIFICATION_GUIDE.md))
3. **Try different USB cable/port**
4. **Collect information:**
   - Firmware version attempting to install
   - Installation method used (A/B/C)
   - Error messages (screenshots)
   - Operating system and version
   - What you've already tried

### Contact Support

**Email:** support@aerocogito.com

**Include:**
- Description of problem
- Steps to reproduce
- Error messages/screenshots
- System information
- What you've tried from this guide

**Response time:** Usually within 48 hours

---

### Security Issues

**If you suspect signature verification issues or security problems:**

**Email:** security@aerocogito.com
**DO NOT** post in public forums

---

## Additional Resources

- **[Installation Guide](INSTALLATION_GUIDE.md)** - Complete installation instructions
- **[Verification Guide](VERIFICATION_GUIDE.md)** - Firmware verification procedures
- **[Maintainer Guide](MAINTAINER_GUIDE.md)** - Advanced procedures (key rotation, recovery)
- **[ArduPilot Forum](https://discuss.ardupilot.org/)** - Community support
- **[ArduPilot Wiki](https://ardupilot.org/copter/)** - Official documentation

---

**Document Version:** 1.0
**Last Updated:** October 28, 2025
