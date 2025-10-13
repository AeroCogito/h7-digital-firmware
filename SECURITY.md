# Security Policy

## 🛡️ Compliance & Standards

### NDAA Section 848 Compliance
This firmware pipeline is designed to meet **NDAA Section 848** (FY2020) requirements:
- ✅ **No covered foreign entities**: Built from official ArduPilot open-source repository
- ✅ **Supply chain transparency**: Complete build provenance via automated GitHub Actions
- ✅ **Firmware integrity**: Cryptographically signed with Ed25519 signatures
- ✅ **Secure boot**: Hardware verification prevents unauthorized code execution

### DIU Blue UAS Framework
Compliant with Defense Innovation Unit cybersecurity requirements:
- ✅ **Cryptographic code signing**: Ed25519 algorithm
- ✅ **Secure boot chain**: Bootloader verifies signatures before execution
- ✅ **Build traceability**: Automated pipeline with full audit logs
- ✅ **Update security**: Only signed firmware accepted by bootloader
- ✅ **No binary blobs**: 100% built from auditable open-source

### SLSA Framework Compliance
Meets [**SLSA Level 2**](https://slsa.dev) requirements for supply chain security:
- ✅ **Build provenance**: Machine-readable attestations for every release
- ✅ **Source integrity**: Verifiable link between source code and built artifacts
- ✅ **Build integrity**: Cryptographically signed by GitHub Actions
- ✅ **Tamper protection**: Sigstore transparency log prevents post-build modification

## 🔐 Security Architecture

### Secure Boot Chain
1. **Bootloader** (secure, signature-verified)
   - Contains public key: `keys/AeroCogito_public_key.dat`
   - Verifies firmware signature on every boot
   - Rejects unsigned or tampered firmware
   
2. **Signed Firmware** (signed with private key)
   - Ed25519 signature appended to `.apj` file
   - Signature verified before execution
   - Prevents unauthorized modifications

3. **Update Mechanism**
   - Mission Planner validates `.apj` signature
   - Bootloader performs final verification
   - Rollback protection via secure boot

### Cryptographic Standards

**Firmware Signing (Runtime Security):**
- **Signing Algorithm**: Ed25519 (EdDSA over Curve25519), implemented via Monocypher
- **Hash Algorithm**: SHA-256/512 for release checksums
- **Key Length**: 256-bit Ed25519 keys
- **Implementation**: [ArduPilot signing framework](https://github.com/ArduPilot/ardupilot/tree/master/Tools/scripts/signing)

**Build Attestation (Supply Chain Security):**
- **Signing Algorithm**: Sigstore (ECDSA P-256)
- **Purpose**: Proves artifact was built by official GitHub Actions workflow
- **Implementation**: [GitHub Attestations](https://docs.github.com/en/actions/security-guides/using-artifact-attestations)

## 🔑 Signing Key Security

Our firmware signing private key is:
- ✅ Generated offline on air-gapped system
- ✅ Stored as GitHub Secrets (AES-256 encrypted at rest and transmitted over TLS in transit)
- ✅ Never committed to repository
- ✅ Backed up in secure offline cold storage
- ✅ Access limited to authorized security personnel only
- ✅ Key rotation policy: Annual review, rotation on compromise

### Key Management
| Key Type | Location | Purpose |
|----------|----------|---------|
| Private Key | GitHub Secret (encrypted) | Firmware signing in CI/CD |
| Private Key Backup | Offline Cold Storage | Disaster recovery |
| Public Key | `keys/AeroCogito_public_key.dat` (repo) | Embedded in bootloader |

**Public Key Fingerprint (SHA-256):** 
```
e3:37:0c:eb:9f:e4:78:de:fd:82:ba:a7:79:8c:4e:dc:95:be:3e:3b:95:fd:ca:ac:07:06:22:f2:f6:1b:16:91
```

Also available at: [`keys/FINGERPRINT.txt`](keys/FINGERPRINT.txt)

## 🏗️ Supply Chain Security

### Build Provenance
Every firmware release includes:
- ✅ ArduPilot source commit SHA
- ✅ Build timestamp (ISO 8601 format)
- ✅ GitHub Actions workflow logs (full audit trail)
- ✅ Toolchain version information
- ✅ SHA-256/512 release checksums
- ✅ Ed25519 digital signature
- ✅ Complete build metadata file
- ✅ **SLSA build attestation** (machine-verifiable provenance)

### Automated Build Pipeline
Our GitHub Actions pipeline ensures:
1. **Source Verification**: Clone from official ArduPilot repository only
2. **Reproducible Builds**: Deterministic compilation environment
3. **Isolated Signing**: Private key used only in secure GitHub environment
4. **Audit Trail**: Complete logs retained for compliance
5. **Build Attestation**: Metadata file with cryptographic details
6. **SLSA Provenance**: Machine-readable attestation signed by GitHub/Sigstore

### Supply Chain Attack Mitigations
- 🔒 Dependabot security alerts enabled
- 🔒 Branch protection rules enforced
- 🔒 Signed commits required from maintainers
- 🔒 GitHub Secret access limited to repository administrators
- 🔒 Security updates monitored via Dependabot

## ✅ Verification

### Verify Firmware Authenticity

**Before installing any firmware, always verify:**

```bash
# 1. Download firmware and checksums
wget https://github.com/AeroCogito/H7-Digital-Flight-Controller/releases/download/[TAG]/arducopter-[TAG]-signed.apj
wget https://github.com/AeroCogito/H7-Digital-Flight-Controller/releases/download/[TAG]/arducopter-[TAG]-SHA256SUMS.txt

# 2. Verify SHA-256 checksum
sha256sum -c arducopter-[TAG]-SHA256SUMS.txt
# Expected output: arducopter-[TAG]-signed.apj: OK

# 3. (Optional) Verify with SHA-512
wget https://github.com/AeroCogito/H7-Digital-Flight-Controller/releases/download/[TAG]/arducopter-[TAG]-SHA512SUMS.txt
sha512sum -c arducopter-[TAG]-SHA512SUMS.txt

# 4. (Optional) Verify public key fingerprint
sha256sum keys/AeroCogito_public_key.dat | awk '{print $1}' | fold -w2 | paste -sd':' -
# Compare with fingerprint above
```

### Advanced Verification - Build Provenance Attestation

For compliance audits and automated verification, we provide [**SLSA build attestations**](https://slsa.dev) signed by GitHub:

**Prerequisites:**
```bash
# Install GitHub CLI
# macOS
brew install gh

# Ubuntu/Debian
sudo apt install gh

# Windows
winget install GitHub.cli
```

**Verify attestation:**
```bash
gh attestation verify arducopter-[TAG]-signed.apj --owner AeroCogito
```

This verifies the artifact was built in GitHub Actions from a specific source commit, with an auditable build process, and is cryptographically signed by GitHub/Sigstore (SLSA Level 2 compliance).

### Bootloader Verification
The secure bootloader **automatically verifies** firmware signatures:
- ✅ On every boot
- ✅ Before executing firmware
- ✅ Rejects unsigned/tampered firmware
- ✅ No user intervention required

## 🚨 Reporting a Vulnerability

**DO NOT** create public issues for security vulnerabilities.

### Contact
**Email:** security@aerocogito.com

### What to Include
- Description of vulnerability
- Affected versions/components
- Steps to reproduce
- Potential impact assessment
- Suggested fix (if any)
- Your contact information for follow-up

### Response Timeline
- **Acknowledgment**: Within 48 hours
- **Initial Assessment**: Within 7 days
- **Status Updates**: Every 14 days until resolved
- **Fix Timeline**:
  - Critical: 30 days
  - High: 60 days
  - Medium: 90 days

### Coordinated Disclosure
We follow responsible disclosure practices:
1. Private notification and verification (you → us)
2. Fix development and testing (us)
3. Patch release (us)
4. Public disclosure (coordinated, 90 days after fix)

## 🔍 Audit & Attestation

### Third-Party Verification
Our firmware and build process are available for independent security audits:
- **Source Code**: https://github.com/ArduPilot/ardupilot
- **Build Pipeline**: `.github/workflows/` (this repository)
- **Public Key**: `keys/AeroCogito_public_key.dat`(this repository)
- **Build Metadata**: Included in every release
- **SLSA Attestations**: Machine-verifiable via GitHub API or attestation bundles

**Audit Capabilities:**
- ✅ **Source-to-binary traceability**: Every release links to exact source commit (see [Build Provenance](#build-provenance))
- ✅ **Build environment verification**: SLSA attestation includes runner details (see [Advanced Verification](#advanced-verification---build-provenance-attestation))
- ✅ **Cryptographic proof**: Both firmware signing (Ed25519) and build attestation (Sigstore)
- ✅ **Transparency logs**: Sigstore provides immutable audit trail
- ✅ **Reproducible builds**: Same source + toolchain = bit-identical output (see [Automated Build Pipeline](#automated-build-pipeline))

**Contact for Security Audits:** security@aerocogito.com

---

**Last Updated:** October 13, 2025  
**Policy Version:** 1.0  
**Next Review:** April 13, 2026