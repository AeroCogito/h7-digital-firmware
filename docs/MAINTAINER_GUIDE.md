# Maintainer Guide
## AeroCogito H7 Digital Flight Controller - Secure Firmware Pipeline

**Audience:** Repository maintainers and administrators
**Last Updated:** October 28, 2025

---

## Overview

This guide covers the technical setup and maintenance procedures for the secure firmware build pipeline. If you're a user looking to install firmware, see the [main README](../README.md).

---

## Table of Contents

1. [Initial Setup](#initial-setup)
2. [Key Generation](#key-generation)
3. [GitHub Secrets Configuration](#github-secrets-configuration)
4. [Release Process](#release-process)
5. [Key Rotation](#key-rotation)
6. [Testing Procedures](#testing-procedures)
7. [Disaster Recovery](#disaster-recovery)

---

## Initial Setup

### Prerequisites

- Administrative access to this GitHub repository
- Air-gapped system for key generation (recommended)
- ArduPilot source code checkout
- Python 3.7+ with pip

### Repository Fork (Optional)

If creating a new secure firmware repository:

```bash
# Fork the repository
git clone https://github.com/AeroCogito/h7-digital-firmware.git
cd h7-digital-firmware

# Review and update board name if needed
# Edit .github/workflows/build-signed-firmware.yml
# Change BOARD_NAME environment variable
```

---

## Key Generation

### Offline Key Generation (Recommended)

**IMPORTANT:** Generate keys on an air-gapped system that has never been connected to the internet.

#### Step 1: Prepare Air-Gapped System

```bash
# On an offline computer with ArduPilot source:
cd /path/to/ardupilot

# Install required cryptographic library
python3 -m pip install pymonocypher==3.1.3.2
```

#### Step 2: Generate Key Pair

```bash
# Generate keys using ArduPilot's official signing script
Tools/scripts/signing/generate_keys.py AeroCogito

# This creates two files in the current directory:
#   - AeroCogito_private_key.dat (32 bytes + header)
#   - AeroCogito_public_key.dat  (57 bytes with PUBLIC_KEYV1 header)
```

#### Step 3: Verify Key Generation

```bash
# Check file sizes
ls -lh AeroCogito_*.dat

# Expected output:
# -rw------- 1 user user   57 Oct 28 AeroCogito_public_key.dat
# -rw------- 1 user user   32 Oct 28 AeroCogito_private_key.dat

# Verify public key format (should start with "PUBLIC_KEYV1:")
head -n1 AeroCogito_public_key.dat
```

#### Step 4: Generate Fingerprint

```bash
# Create fingerprint for documentation
sha256sum AeroCogito_public_key.dat | awk '{print $1}' | fold -w2 | paste -sd':' - > AeroCogito_fingerprint.txt

# Display fingerprint
cat AeroCogito_fingerprint.txt
```

#### Step 5: Secure Private Key Storage

```bash
# CRITICAL: Store private key securely

# Option 1: Hardware Security Module (HSM)
# - Recommended for production use
# - Store encrypted on HSM

# Option 2: Encrypted USB drive
# - Keep offline in secure location
# - Use strong encryption (LUKS, VeraCrypt)

# Option 3: Password manager (encrypted vault)
# - Use dedicated security vault
# - Enable 2FA and audit logging

# Never store private key in:
# - Cloud storage
# - Email
# - Git repository
# - Unencrypted disk
```

#### Step 6: Prepare for GitHub Secrets

```bash
# Encode private key for GitHub Secrets (base64)
base64 -w 0 AeroCogito_private_key.dat > AeroCogito_private_key_base64.txt

# On macOS (no -w flag):
base64 -i AeroCogito_private_key.dat > AeroCogito_private_key_base64.txt

# Transfer base64 file to internet-connected computer via USB drive
# Delete base64 file after uploading to GitHub Secrets
```

#### Step 7: Commit Public Key

```bash
# On internet-connected computer, copy public key to repository
mkdir -p keys
cp AeroCogito_public_key.dat keys/
cp AeroCogito_fingerprint.txt keys/FINGERPRINT.txt

# Update fingerprint in README.md and SECURITY.md
# Commit to repository
git add keys/
git commit -m "Add AeroCogito signing public key"
git push
```

---

## GitHub Secrets Configuration

### Required Secrets

Navigate to: `Settings → Secrets and variables → Actions → New repository secret`

#### 1. SIGNING_PRIVATE_KEY

| Field | Value |
|-------|-------|
| **Name** | `SIGNING_PRIVATE_KEY` |
| **Secret** | Contents of `AeroCogito_private_key_base64.txt` |

**Format:** Base64-encoded Ed25519 private key in ArduPilot `.dat` format

**Validation:**
```bash
# The base64 string should be approximately 44 characters
# Example (DO NOT USE - generate your own):
# YXNkZmFzZGZhc2RmYXNkZmFzZGZhc2RmYXNkZmFzZGZhc2RmYXNkZmFzZGY=
```

#### 2. DISPATCH_TOKEN

| Field | Value |
|-------|-------|
| **Name** | `DISPATCH_TOKEN` |
| **Secret** | GitHub Personal Access Token (Classic) |

**Requirements:**
- Token type: Personal Access Token (Classic)
- Scopes: `repo` (full repository access)
- Expiration: 1 year (set calendar reminder)

**To create:**
1. Go to: `Settings → Developer settings → Personal access tokens → Tokens (classic)`
2. Click "Generate new token (classic)"
3. Name: "AeroCogito Firmware Build Dispatcher"
4. Expiration: 1 year
5. Scopes: Check ✅ `repo`
6. Click "Generate token"
7. **Copy immediately** (won't be shown again)
8. Add as `DISPATCH_TOKEN` secret

---

## Release Process

### Automated Releases (Recommended)

Automated builds trigger daily at UTC midnight via `check-ardupilot-releases.yml`:

```yaml
schedule:
  - cron: '0 0 * * *'  # Daily at midnight UTC
```

**How it works:**
1. Workflow checks for new ArduPilot Copter releases
2. Compares against baseline version (`Copter-4.6.2`) when the board support was first added
3. Checks if release already built (looks for `{tag}-signed` in git tags)
4. If new release found, triggers `build-signed-firmware.yml` via repository dispatch
5. Build workflow compiles, signs, and publishes firmware

**Monitoring:**
- View workflow runs: `Actions → Check ArduPilot Releases`
- Check for failures: Email notifications (if enabled)
- Review releases: `Releases` tab

### Manual Releases

To build a specific ArduPilot version manually:

1. Go to `Actions → Build Signed Firmware → Run workflow`
2. Select branch: `main`
3. Enter ArduPilot release tag (e.g., `Copter-4.5.7`)
4. Click "Run workflow"

**Build steps:**
1. Clone ArduPilot at specified tag
2. Install build dependencies (official ArduPilot script)
3. Build secure bootloader with embedded public key
4. Configure WAF with `--signed-fw` and `--private-key`
5. Build firmware (ArduPilot automatically signs during build)
6. Generate checksums (SHA-256, SHA-512)
7. Create SLSA build attestations (Sigstore)
8. Upload artifacts
9. Create GitHub release with tag `{original-tag}-signed`

**Build time:** Approximately 20-30 minutes

### Post-Release Verification

After each release:

```bash
# 1. Download release artifacts
wget https://github.com/AeroCogito/h7-digital-firmware/releases/download/Copter-X.Y.Z-signed/arducopter-Copter-X.Y.Z-signed.apj

# 2. Verify checksum
wget https://github.com/AeroCogito/h7-digital-firmware/releases/download/Copter-X.Y.Z-signed/arducopter-Copter-X.Y.Z-SHA256SUMS.txt
sha256sum -c arducopter-Copter-X.Y.Z-SHA256SUMS.txt

# 3. Verify SLSA attestation (requires GitHub CLI)
gh attestation verify arducopter-Copter-X.Y.Z-signed.apj --owner AeroCogito

# 4. Test on hardware (if available)
# - Flash to test board
# - Verify bootloader accepts signed firmware
# - Check firmware boots correctly
# - Verify functionality
```

---

## Key Rotation

### When to Rotate Keys

- **Scheduled:** Annually (recommended)
- **Compromise:** Immediately if private key suspected compromised
- **Policy change:** If security policy requires different key

### Key Rotation Procedure

#### Phase 1: Generate New Key Pair

```bash
# On air-gapped system
cd /path/to/ardupilot
Tools/scripts/signing/generate_keys.py AeroCogito_v2

# Store new private key securely
# Generate fingerprint
sha256sum AeroCogito_v2_public_key.dat | awk '{print $1}' | fold -w2 | paste -sd':' -
```

#### Phase 2: Update Bootloader

**CRITICAL:** The bootloader supports up to 10 public keys. Add new key while keeping old key for backward compatibility.

```bash
# Build bootloader with BOTH keys (old + new)
cd ardupilot
Tools/scripts/build_bootloaders.py AeroCogito-H7Digital \
  --signing-key=../keys/AeroCogito_public_key.dat \
  --signing-key=../keys/AeroCogito_v2_public_key.dat

# This creates bootloader with both keys embedded
```

#### Phase 3: Update GitHub Secret

1. Encode new private key: `base64 -w 0 AeroCogito_v2_private_key.dat`
2. Update `SIGNING_PRIVATE_KEY` secret in GitHub
3. **Keep old key backed up** until all devices updated

#### Phase 4: Build and Release New Firmware

```bash
# Update workflow to use new key
# The --private-key flag in workflow now uses new key
# But bootloader accepts BOTH keys

# Trigger manual build to test
# Actions → Build Signed Firmware → Run workflow
```

#### Phase 5: Deprecation Timeline

**Recommended timeline:**

| Timeframe | Action |
|-----------|--------|
| **T+0 (Day 1)** | New key active, both keys in bootloader |
| **T+90 days** | Announce old key deprecation |
| **T+180 days** | Final firmware release with dual-key bootloader |
| **T+365 days** | Remove old key from bootloader (new builds) |

**Communication:**
- Add deprecation notice to release notes
- Email notification to known users
- Update SECURITY.md with deprecation timeline

---

## Testing Procedures

### Pre-Release Testing Checklist

Before each release (automated or manual):

- [ ] **Workflow syntax validated** (tests.yml runs automatically)
- [ ] **Build completes successfully** (check Actions logs)
- [ ] **Artifacts generated** (.apj, .hex, .abin files present)
- [ ] **Checksums correct** (SHA-256 and SHA-512 files generated)
- [ ] **Attestations created** (SLSA provenance present)
- [ ] **File sizes reasonable** (firmware 1-2 MB, bootloader ~50 KB)

### Hardware Testing (If Available)

Recommended for major releases:

```bash
# Test 1: APJ update on device with bootloader
# - Flash .apj via Mission Planner
# - Verify bootloader accepts signature
# - Verify firmware boots and runs

# Test 2: Full DFU flash
# - Flash .hex via STM32CubeProgrammer
# - Verify bootloader installed correctly
# - Verify firmware boots and runs

# Test 3: SD card update
# - Rename .abin to ardupilot.abin
# - Copy to SD card root
# - Power cycle
# - Verify update completes
```

### Signature Verification Test

```bash
# ArduPilot doesn't provide a separate signature verification script
# Verification happens at runtime by the bootloader

# To test signature format, check .apj file:
unzip -l arducopter-*.apj
# Should contain firmware.bin and signature metadata in JSON
```

---

## Disaster Recovery

### Lost Private Key

**Impact:** Cannot sign new firmware, but existing firmware still works

**Recovery procedure:**

1. **Generate new key pair** (follow Key Generation section)
2. **Build new bootloader** with new public key
3. **Create recovery firmware** with new bootloader included
4. **Distribute recovery instructions** to all users:
   - Must flash new bootloader via DFU (cannot update via .apj)
   - Download new `*-with-bootloader-signed.hex` file
   - Flash via STM32CubeProgrammer
5. **Update documentation** with new public key fingerprint
6. **Revoke old key** in SECURITY.md

**Prevention:**
- Maintain encrypted backups in multiple locations
- Use hardware security module (HSM) for production
- Document key backup procedure
- Test recovery annually

### Compromised Private Key

**Impact:** CRITICAL - Unauthorized parties can sign malicious firmware

**Immediate actions:**

1. **Revoke key immediately**
   - Update SECURITY.md with compromise notice
   - Remove `SIGNING_PRIVATE_KEY` from GitHub Secrets
   - Notify all users via email/website

2. **Generate new key pair**
   - Follow Key Generation procedure
   - Use different key name (e.g., `AeroCogito_emergency`)

3. **Release emergency firmware**
   - Build with new key ASAP
   - Include new bootloader
   - Mark as "SECURITY UPDATE - CRITICAL"

4. **Investigate compromise**
   - Review GitHub Actions logs
   - Check for unauthorized workflow runs
   - Audit GitHub access logs
   - Report to GitHub Security if workflow compromise

5. **Post-incident**
   - Update security procedures
   - Implement additional controls (key rotation automation)
   - Document incident in SECURITY.md

### Build Workflow Failure

**Common causes:**

1. **ArduPilot source unavailable**
   - GitHub API rate limit → Wait 1 hour or use authenticated requests
   - ArduPilot repository down → Monitor https://github.com/ArduPilot/ardupilot

2. **Build errors**
   - Toolchain version mismatch → Pinned to official ArduPilot script
   - Dependency changes → Review ArduPilot release notes

3. **Signing failure**
   - Private key secret corrupted → Re-add secret
   - Private key format wrong → Verify base64 encoding

4. **Attestation failure**
   - GitHub Actions permissions → Check workflow permissions
   - Sigstore service down → Retry later

**Debug procedure:**

```bash
# View workflow logs
# Go to Actions → Failed workflow → View logs

# Common errors and solutions:
# "Private key not found" → Check SIGNING_PRIVATE_KEY secret
# "Bootloader not generated" → Board not supported in ArduPilot version
# "Signature invalid" → Private/public key mismatch
```

---

## Maintenance Schedule

| Task | Frequency | Procedure |
|------|-----------|-----------|
| **Review workflow runs** | Weekly | Check Actions tab for failures |
| **Update dependencies** | Weekly (automated) | Review Dependabot PRs |
| **Test firmware release** | Monthly (optional) | Flash to test hardware |
| **Rotate DISPATCH_TOKEN** | Annually | Generate new PAT, update secret |
| **Key rotation review** | Annually | Assess if rotation needed |
| **Security audit** | Quarterly | Review SECURITY.md compliance |
| **Disaster recovery drill** | Annually | Test key recovery procedure |

---

## Additional Resources

- **ArduPilot Secure Boot & Signing Documentation:** https://github.com/ArduPilot/ardupilot/tree/master/Tools/scripts/signing
- **SLSA Framework:** https://slsa.dev
- **GitHub Actions Security:** https://docs.github.com/en/actions/security-guides
- **Sigstore Documentation:** https://docs.sigstore.dev

---

## Contact

For maintainer-specific questions:

- **Security Issues:** security@aerocogito.com
- **Build Issues:** security@aerocogito.com
- **Key Management:** security@aerocogito.com (encrypted email preferred)

---

**Document Version:** 1.0
**Last Updated:** October 28, 2025
**Next Review:** January 28, 2026
