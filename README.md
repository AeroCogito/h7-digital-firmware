# AeroCogito H7 Digital Flight Controller

[![Latest Release](https://img.shields.io/github/v/release/AeroCogito/h7-digital-firmware?label=Latest%20Firmware)](../../releases/latest)
[![Build Status](https://img.shields.io/github/actions/workflow/status/AeroCogito/h7-digital-firmware/build-signed-firmware.yml?branch=main)](../../actions)
[![License](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](LICENSE)
[![NDAA Compliant](https://img.shields.io/badge/NDAA%20848-Compliant-green.svg)](SECURITY.md)
[![SLSA 2](https://slsa.dev/images/gh-badge-level2.svg)](SECURITY.md)

Secure, **NDAA** / **Blue UAS** compliant ArduPilot firmware for the [**AeroCogito H7 Digital Flight Controller**](https://aerocogito.com/store/p/h7-digital)

![AeroCogito H7 Digital Flight Controller](docs/images/H7-Digital_pinout.jpg)

## 🚁 Hardware Overview

**STM32H743-based flight controller** designed for professional UAS applications requiring high performance and **secure, compliant firmware**.
- Full ArduPilot/BetaFlight compatibility
- [Hardware specifications & purchase info](https://aerocogito.com/store/p/h7-digital)

## 🔐 Security & Compliance

- **Secure Boot:** Ed25519 signed firmware with hardware bootloader verification
- **NDAA & Blue UAS Compliant:** Meets **NDAA Section 848 Operating Software** compliance and **DMCA Blue UAS Framework** requirements
- **Built from Source:** Complete transparency with automated builds from official ArduPilot releases
- **Cryptographic Verification:** SHA-256 and SHA-512 checksums for all releases
- **🇺🇸 Made in the USA:** Hardware and firmware pipeline designed and maintained in the United States

[Full compliance details in SECURITY.md](SECURITY.md)

## 📦 Downloads

Get the latest signed firmware from [Releases](../../releases/latest)

### File Types

| File | Purpose |
|------|---------|
| `*-signed.apj` | Firmware updates (requires secure bootloader already installed) |
| `*-signed.abin`| Firmware updates via SD Card (requires secure bootloader already installed) |
| `*-with-bootloader-signed.hex` | Complete system (secure bootloader + signed firmware for new installations) |

## 🚀 Quick Start

**Always verify checksums before installing** (see [Verification Guide](docs/VERIFICATION_GUIDE.md))

### Option A: Firmware Update via Mission Planner/QGC

1. Download `arducopter-*-signed.apj` from [Releases](../../releases/latest)
2. Mission Planner → Setup → Install Firmware → Load custom firmware
3. Select downloaded `.apj` file

### Option B: Firmware Update via SD Card

1. Download `arducopter-*-signed.abin` from [Releases](../../releases/latest)
2. **Rename to exactly `ardupilot.abin`** (critical requirement)
3. Copy to SD card root directory (not in any folder)
4. Insert SD card and power cycle board
5. Update completes in ~1 minute (file renamed to `ardupilot-flashed.abin`)

### Option C: Fresh Installation (DFU Mode)

1. Download `arducopter-*-with-bootloader-signed.hex` from [Releases](../../releases/latest)
2. Put board in DFU mode (hold BOOT button while connecting USB)
3. Flash using STM32CubeProgrammer or BetaFlight Configurator
4. Power cycle board

**Detailed instructions:** See [Installation Guide](docs/INSTALLATION_GUIDE.md)

## 🔑 Public Key

Firmware signing public key fingerprint ([`keys/FINGERPRINT.txt`](keys/FINGERPRINT.txt)):
```
e3:37:0c:eb:9f:e4:78:de:fd:82:ba:a7:79:8c:4e:dc:95:be:3e:3b:95:fd:ca:ac:07:06:22:f2:f6:1b:16:91
```

The secure bootloader contains this public key and only boots firmware with valid Ed25519 signatures. Full key file: [`keys/AeroCogito_public_key.dat`](keys/AeroCogito_public_key.dat)

## 🏗️ How It Works

Automated builds check for new ArduPilot releases daily and produce cryptographically signed firmware with SLSA Level 2 attestations. The secure bootloader (built with embedded public key) verifies Ed25519 signatures before executing firmware.

**Build logs:** [GitHub Actions](../../actions)
**Build workflow:** [build-signed-firmware.yml](.github/workflows/build-signed-firmware.yml)
**Build process details:** [Maintainer Guide](docs/MAINTAINER_GUIDE.md#release-process)

## 📚 Documentation

### For Users
- **[Verification Guide](docs/VERIFICATION_GUIDE.md)** - How to verify firmware authenticity
- **[Installation Guide](docs/INSTALLATION_GUIDE.md)** - Detailed installation for all methods
- **[Troubleshooting](docs/TROUBLESHOOTING.md)** - Common issues and solutions
- **[Security Policy](SECURITY.md)** - Compliance details and vulnerability reporting

### For Maintainers
- **[Maintainer Guide](docs/MAINTAINER_GUIDE.md)** - Key generation, secrets setup, release process

### External Resources
- **[ArduPilot Documentation](https://ardupilot.org/copter/)** - Official ArduPilot docs
- **[Hardware Specifications](https://aerocogito.com/store/p/h7-digital)** - AeroCogito H7 Digital details

## 🆘 Support

- **Email:** support@aerocogito.com
- **Security:** security@aerocogito.com (for vulnerabilities and compliance auditing)

## ⚖️ License

This project follows ArduPilot's GPL-3.0 license.

Firmware is based on [ArduPilot](https://github.com/ArduPilot/ardupilot) © ArduPilot Dev Team.
