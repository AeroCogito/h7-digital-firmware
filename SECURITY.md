# Security Policy

## 🛡️ Compliance & Standards

### NDAA Section 848 Compliance

This firmware pipeline is designed to meet **NDAA Section 848** (FY2020) requirements:

| Requirement | Status | Implementation |
|------------|--------|----------------|
| **No Covered Foreign Entities** | ✅ COMPLIANT | Built from official ArduPilot open-source repository |
| **Supply Chain Transparency** | ✅ COMPLIANT | Complete build provenance via GitHub Actions with SLSA attestations |
| **Firmware Integrity Protection** | ✅ COMPLIANT | Ed25519 cryptographic signatures on all firmware |
| **Secure Boot Chain** | ✅ COMPLIANT | Bootloader verifies signatures before execution |
| **No Binary Blobs** | ✅ COMPLIANT | 100% built from source code |
| **Traceability** | ✅ COMPLIANT | GitHub Actions logs retained for 90 days minimum |
| **Update Security** | ✅ COMPLIANT | Only signed firmware accepted by bootloader |

**Overall Status:** ✅ **FULLY COMPLIANT**

### DIU Blue UAS Framework (PENDING)

Compliant with Defense Innovation Unit cybersecurity requirements:

| Requirement | Status | Implementation |
|------------|--------|----------------|
| **Cryptographic Code Signing** | ✅ COMPLIANT | Ed25519 digital signatures via Monocypher 3.1.3.2 |
| **Secure Boot Chain** | ✅ COMPLIANT | Hardware-verified boot process |
| **Build Traceability** | ✅ COMPLIANT | Automated GitHub Actions pipeline with full audit logs |
| **Key Management** | ✅ COMPLIANT | Offline key generation, secure GitHub Secrets storage |
| **Update Security** | ✅ COMPLIANT | Signed updates only, bootloader signature verification |
| **Source Transparency** | ✅ COMPLIANT | Public GitHub repository with all build scripts |
| **No Backdoors** | ✅ COMPLIANT | Open-source ArduPilot base, auditable source code |
| **Tamper Protection** | ✅ COMPLIANT | Signature verification at boot, hardware rejects modified firmware |
| **Supply Chain Security** | ✅ COMPLIANT | SLSA L2 attestations, Dependabot monitoring |

**Overall Status:** ✅ **FULLY COMPLIANT** (PENDING)

### SLSA Framework Compliance

Meets [**SLSA Level 2**](https://slsa.dev) requirements for supply chain security:

#### SLSA Level 1

| Requirement | Status | Implementation |
|------------|--------|----------------|
| **Build Scripted** | ✅ MET | Fully automated WAF build system (ArduPilot WAF + GitHub Actions) |
| **Provenance Exists** | ✅ MET | Build metadata generated for every release |

#### SLSA Level 2

| Requirement | Status | Implementation |
|------------|--------|----------------|
| **Version Control** | ✅ MET | GitHub version control for all source with commit history |
| **Hosted Build Service** | ✅ MET | GitHub Actions hosted runners (ubuntu-22.04) |
| **Authenticated Provenance** | ✅ MET | Sigstore-signed SLSA attestations |
| **Service-Generated Provenance** | ✅ MET | GitHub Actions generates attestations automatically |
| **Provenance Non-Falsifiable** | ✅ MET | Cryptographically signed by GitHub's Sigstore identity |

**Overall Status:** ✅ **SLSA Level 2 FULLY COMPLIANT**

**Future (SLSA Level 3):** Source provenance verification, hermetic builds, comprehensive SBOM (planned Q1-Q2 2026)

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
- **Implementation**: [GitHub Attestations](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/use-artifact-attestations)

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

### Quick Verification

**Before installing firmware, always verify checksums:**

```bash
# Download and verify (Linux/macOS)
sha256sum -c arducopter-[TAG]-SHA256SUMS.txt
# Expected: arducopter-[TAG]-signed.apj: OK
```

**Complete verification instructions:** [docs/VERIFICATION_GUIDE.md](docs/VERIFICATION_GUIDE.md)

The verification guide covers:
- Quick verification (all platforms: Linux, macOS, Windows)
- Enhanced verification (SHA-512, public key fingerprint)
- Advanced verification (SLSA attestations for compliance audits)
- Bootloader verification
- Troubleshooting verification issues

**Installation and troubleshooting:**
- [Installation Guide](docs/INSTALLATION_GUIDE.md) - Complete installation procedures for all methods
- [Troubleshooting Guide](docs/TROUBLESHOOTING.md) - Common issues and solutions

### Automated Verification

The secure bootloader **automatically verifies** firmware signatures on every boot before executing firmware, rejecting unsigned or tampered firmware with no user intervention required.

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

### Coordinated Vulnerability Disclosure
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
- ✅ **Build environment verification**: SLSA attestation includes runner details (see [Advanced Verification](docs/VERIFICATION_GUIDE.md#advanced-verification-compliance--auditing))
- ✅ **Cryptographic proof**: Both firmware signing (Ed25519) and build attestation (Sigstore)
- ✅ **Transparency logs**: Sigstore provides immutable audit trail
- ✅ **Reproducible builds**: Same source + toolchain = bit-identical output (see [Automated Build Pipeline](#automated-build-pipeline))

**Contact for Security Audits:** security@aerocogito.com

---

**Last Updated:** October 28, 2025
**Policy Version:** 1.0
**Next Review:** April 28, 2026