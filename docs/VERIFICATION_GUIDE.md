# Firmware Verification Guide
## AeroCogito H7 Digital Flight Controller

**Purpose:** Verify firmware authenticity and integrity before installation
**Last Updated:** October 28, 2025

---

## Why Verify Firmware?

**Verifying firmware before installation ensures:**
- ✅ Firmware was built by AeroCogito (not modified by attackers)
- ✅ Firmware wasn't corrupted during download
- ✅ Firmware came from official GitHub repository
- ✅ Build process is auditable and traceable

**Security principle:** Never trust, always verify.

---

## Quick Verification (Recommended for All Users)

### Linux / macOS

```bash
# 1. Download firmware and checksum file
cd ~/Downloads
# (firmware file already downloaded from GitHub Releases)

# 2. Download checksum file
wget https://github.com/AeroCogito/h7-digital-firmware/releases/download/Copter-X.Y.Z-signed/arducopter-Copter-X.Y.Z-SHA256SUMS.txt

# 3. Verify checksum
sha256sum -c arducopter-Copter-X.Y.Z-SHA256SUMS.txt

# Expected output:
# arducopter-Copter-X.Y.Z-signed.apj: OK
```

✅ **If you see "OK" - firmware is verified and safe to install**

❌ **If you see "FAILED" - DO NOT INSTALL - file may be corrupted or tampered**

### Windows PowerShell

```powershell
# 1. Download firmware and checksum file from GitHub Releases
# Save both to same folder (e.g., Downloads)

# 2. Open PowerShell in that folder
cd $HOME\Downloads

# 3. Calculate SHA-256 hash
Get-FileHash .\arducopter-Copter-X.Y.Z-signed.apj -Algorithm SHA256

# 4. Open the SHA256SUMS.txt file in Notepad
notepad .\arducopter-Copter-X.Y.Z-SHA256SUMS.txt

# 5. Compare the hash values manually
# They should match exactly
```

✅ **If hashes match - firmware is verified and safe to install**

❌ **If hashes don't match - DO NOT INSTALL - file may be corrupted or tampered**

### Windows (Alternative: CertUtil)

```cmd
REM Calculate SHA-256 hash
certutil -hashfile arducopter-Copter-X.Y.Z-signed.apj SHA256

REM Compare with SHA256SUMS.txt manually
```

---

## Enhanced Verification (Recommended for Security-Conscious Users)

### SHA-512 Verification

SHA-512 provides additional security over SHA-256:

```bash
# Download SHA-512 checksum file
wget https://github.com/AeroCogito/h7-digital-firmware/releases/download/Copter-X.Y.Z-signed/arducopter-Copter-X.Y.Z-SHA512SUMS.txt

# Verify with SHA-512
sha512sum -c arducopter-Copter-X.Y.Z-SHA512SUMS.txt
```

### Public Key Fingerprint Verification

Verify the public key used to sign the firmware:

```bash
# 1. Clone or download repository
git clone https://github.com/AeroCogito/h7-digital-firmware.git
cd h7-digital-firmware

# 2. Calculate public key fingerprint
sha256sum keys/AeroCogito_public_key.dat | awk '{print $1}' | fold -w2 | paste -sd':' -

# 3. Compare with documented fingerprint
cat keys/FINGERPRINT.txt

# Expected fingerprint:
# e3:37:0c:eb:9f:e4:78:de:fd:82:ba:a7:79:8c:4e:dc:95:be:3e:3b:95:fd:ca:ac:07:06:22:f2:f6:1b:16:91
```

✅ **If fingerprints match - public key is authentic**

---

## Advanced Verification (Compliance & Auditing)

### SLSA Build Provenance Attestation

For compliance audits, regulatory requirements, or automated verification, we provide **SLSA Level 2 build attestations** signed by GitHub.

#### What are Attestations?

Attestations are cryptographic proofs that:
- Firmware was built by GitHub Actions (not on a developer's laptop)
- Specific source commit was used (traceable to ArduPilot)
- Build process is auditable (view logs for 90 days)
- Signed by GitHub/Sigstore (tamper-proof)

#### Prerequisites

Install GitHub CLI:

```bash
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Windows
winget install GitHub.cli

# Or download from: https://cli.github.com
```

#### Verify Attestation

```bash
# Verify firmware was built by official GitHub Actions workflow
gh attestation verify arducopter-Copter-X.Y.Z-signed.apj --owner AeroCogito

# Expected output:
# Loaded digest sha256:abc123... for file://arducopter-Copter-X.Y.Z-signed.apj
# Loaded 1 attestation from GitHub API
# ✓ Verification succeeded!
#
# sha256:abc123... was attested by:
# REPO                    PREDICATE_TYPE                  WORKFLOW
# AeroCogito/H7-Digital-  https://slsa.dev/provenance/v1  .github/workflows/build-signed-firmware.yml@refs/heads/main
```

✅ **"Verification succeeded" - Build provenance is authentic**

#### What Attestation Proves

When verification succeeds, you have cryptographic proof that:

1. **Built in GitHub Actions** - Not on an untrusted computer
2. **From specific commit** - Traceable to exact source code
3. **Workflow auditable** - View build logs on GitHub
4. **Signed by GitHub** - Tamper-proof via Sigstore transparency log
5. **SLSA L2 compliant** - Meets supply chain security standards

#### Attestation Metadata

View detailed attestation information:

```bash
# Download attestation JSON
gh attestation download arducopter-Copter-X.Y.Z-signed.apj --owner AeroCogito

# Inspect attestation (requires jq)
cat attestation.json | jq

# Key fields:
# - subject: Firmware file hash
# - predicate: Build details (repo, commit, workflow)
# - signature: GitHub/Sigstore signature
```

---

## Bootloader Verification

### Automatic Verification

The AeroCogito secure bootloader **automatically verifies** firmware signatures:

- ✅ On every boot
- ✅ Before executing firmware
- ✅ Rejects unsigned firmware
- ✅ Rejects firmware with invalid signatures
- ✅ Rejects firmware signed with unknown keys

**No user action required** - verification is automatic.

### How to Tell Bootloader is Verifying

**Visual indicators:**

1. **USB Device Name** (when in bootloader mode):
   - Secure bootloader: `AeroCogito-H7Digital-Secure-BL-v10`
   - Non-secure bootloader: `AeroCogito-H7Digital-BL-v10` (no "Secure")

2. **Boot Behavior**:
   - Valid signed firmware: Boots normally
   - Invalid/unsigned firmware: Stays in bootloader forever (LED pattern: slow blink)

**Check on Linux:**

```bash
# Connect board in bootloader mode
dmesg | grep "Product:"

# Expected output with secure bootloader:
# Product: AeroCogito-H7Digital-Secure-BL-v10
```

**Check on Windows:**

1. Open Device Manager
2. Find the flight controller device
3. Right-click → Properties → Details → Bus reported device description
4. Should contain "Secure-BL"

### Manual Firmware Signature Test

To test if a firmware file is signed (advanced users):

```bash
# APJ files are ZIP archives containing firmware + metadata
unzip -l arducopter-Copter-X.Y.Z-signed.apj

# Look for these entries:
# - firmware.bin (actual firmware)
# - firmware.apj.json (metadata including signature)

# Extract and inspect JSON
unzip arducopter-Copter-X.Y.Z-signed.apj firmware.apj.json
cat firmware.apj.json | jq

# Should contain "signature" field with Ed25519 signature
```

**Note:** ArduPilot doesn't provide a separate signature verification script. Verification happens at boot time by the bootloader.

---

## Verification Troubleshooting

### Problem: Checksum Verification Failed

**Error message:**
```
arducopter-Copter-X.Y.Z-signed.apj: FAILED
```

**Possible causes:**
1. File corrupted during download
2. File modified after download
3. Downloaded wrong file

**Solutions:**
1. **Re-download firmware** from official GitHub Releases
2. **Try different browser** or download method
3. **Check download completed** - verify file size matches release notes
4. **Disable antivirus temporarily** - some AV software modifies files
5. **If still fails** - report to security@aerocogito.com

### Problem: Attestation Verification Failed

**Error message:**
```
✗ Verification failed
```

**Possible causes:**
1. File not from official GitHub Releases
2. Attestation not yet published (wait a few minutes after release)
3. GitHub CLI not authenticated

**Solutions:**
1. **Verify download source** - Must be from `github.com/AeroCogito/h7-digital-firmware/releases`
2. **Authenticate GitHub CLI:**
   ```bash
   gh auth login
   ```
3. **Check file is correct:**
   ```bash
   file arducopter-Copter-X.Y.Z-signed.apj
   ```
4. **Wait and retry** - Attestations may take 5-10 minutes to propagate
5. **Check repository owner spelling** - Must be exactly `AeroCogito`

### Problem: Public Key Fingerprint Doesn't Match

**Possible causes:**
1. Repository was forked/cloned from wrong source
2. Public key was modified
3. Using old version of repository

**Solutions:**
1. **Verify repository URL:**
   ```bash
   git remote -v
   # Should show: https://github.com/AeroCogito/h7-digital-firmware.git
   ```
2. **Update repository:**
   ```bash
   git pull origin main
   ```
3. **Check key file integrity:**
   ```bash
   ls -l keys/AeroCogito_public_key.dat
   # Should be exactly 57 bytes
   ```
4. **Re-clone from official repo:**
   ```bash
   git clone https://github.com/AeroCogito/h7-digital-firmware.git
   ```

### Problem: Bootloader Not Verifying Signatures

**Symptoms:**
- Unsigned firmware boots successfully
- USB device name doesn't show "Secure-BL"

**Cause:** Non-secure bootloader installed

**Solution:** Install secure bootloader via DFU:
1. Download `*-with-bootloader-signed.hex` from latest release
2. Put board in DFU mode
3. Flash via STM32CubeProgrammer
4. See [Installation Guide](INSTALLATION_GUIDE.md) for details

---

## Verification Best Practices

### For Users

1. ✅ **Always verify checksums** before installation
2. ✅ **Download only from official GitHub Releases**
3. ✅ **Check release notes** for version compatibility
4. ✅ **Keep verification tools updated** (sha256sum, GitHub CLI)
5. ✅ **Report suspicious files** to security@aerocogito.com

### For Compliance/Auditing

1. ✅ **Use attestation verification** for audit trail
2. ✅ **Document verification results** in compliance reports
3. ✅ **Maintain verification logs** for regulatory requirements
4. ✅ **Verify public key fingerprint** annually
5. ✅ **Review build logs** on GitHub Actions (available 90 days)

### For Enterprises

1. ✅ **Implement automated verification** in deployment pipeline
2. ✅ **Require attestation verification** before production deployment
3. ✅ **Maintain air-gapped verification systems** for critical deployments
4. ✅ **Archive verification results** for compliance
5. ✅ **Subscribe to security advisories** (GitHub Watch → Custom → Releases)

---

## Verification Checklist

Before installing firmware, complete this checklist:

- [ ] Downloaded from official GitHub Releases (not third-party)
- [ ] SHA-256 checksum verified (or SHA-512)
- [ ] Checksum verification **passed** (shows "OK")
- [ ] (Optional) Attestation verified with `gh attestation verify`
- [ ] (Optional) Public key fingerprint matches documentation
- [ ] Release notes reviewed for compatibility
- [ ] Ready to proceed with installation

✅ **All checks passed → Safe to install**

---

## Additional Resources

- **Installation Guide:** [INSTALLATION_GUIDE.md](INSTALLATION_GUIDE.md)
- **Troubleshooting:** [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- **Security Policy:** [../SECURITY.md](../SECURITY.md)
- **SLSA Framework:** https://slsa.dev
- **Sigstore Documentation:** https://docs.sigstore.dev
- **GitHub CLI:** https://cli.github.com

---

## Support

**Questions about verification:**
- General questions: support@aerocogito.com
- Security concerns: security@aerocogito.com

---

**Document Version:** 1.0
**Last Updated:** October 28, 2025
