# QEMU OP-TEE Trust Platform - Project Plan

## Project Description

Yocto-based distribution for building a hardened OS with OP-TEE (Open Portable Trusted Execution Environment) support. The project targets embedded security use cases: secure key storage, PKCS#11 token access, trusted services, and platform integrity, built on top of OpenEmbedded/Yocto with the kas build tool.

### Author's Vision (from README)

> **TARGET**: Orin NX and QEMU
>
> **FEATURES**:
> 1. OP-TEE - Deep understanding, PKCS#11, key storage, all trust services, Customize OP-TEE
> 2. TBD (security hardening features via meta-secure-core)

---

## Architecture Overview

```
kas configs (kas/)
  |
  +-- base.yml                    Common: bitbake 2.18, oe-core (wrynose), meta-yocto/poky
  |
  +-- openembedded.yml            meta-oe, meta-python, meta-networking, meta-filesystems
  |
  +-- qemuarm64-secureboot.yml    MACHINE=qemuarm64-secureboot, meta-arm (OP-TEE)
  |                               Includes: base.yml
  |
  +-- orin-nx.yml                 MACHINE=p3768-0000-p3767-0000, meta-tegra
  |                               Includes: base.yml, openembedded.yml
  |
  +-- test.yml                    Inherits meta-arm CI configs for qemuarm64-secureboot

All layers (meta-arm, meta-openembedded, meta-tegra, oe-core, etc.) are
fetched automatically by kas based on the repos: sections in the yml configs.
```

---

## Current KAS Build Review

### base.yml
- **Branch**: `wrynose` (Yocto 5.x series)
- **Bitbake**: 2.18 from git.openembedded.org
- **OE-Core**: pinned commit `b3aa1a6419bef653cff710fc3359211daf9f7902`
- **Config**: `rm_work` enabled (saves disk), `perf` in CORE_IMAGE_EXTRA_INSTALL
- **Note**: BB_HASHSERVE=auto for shared state acceleration

### qemuarm64-secureboot.yml
- **Machine**: `qemuarm64-secureboot` (from meta-arm-bsp)
- **Target**: `core-image-base`
- **OP-TEE packages**: `optee-test optee-client optee-os-ta` via IMAGE_INSTALL
- **Tests**: `optee ftpm` test suites configured
- **Init**: systemd
- **meta-arm source**: `github.com/jonmason/meta-arm.git` (Jon Mason's fork)
- **Issue**: Does not include `openembedded.yml` -- may need it for OP-TEE PKCS#11 dependencies

### orin-nx.yml
- **Machine**: `p3768-0000-p3767-0000` (Jetson Orin NX)
- **Target**: `core-image-base`
- **Includes**: base.yml + openembedded.yml
- **Packages**: i2c-tools, opensc, networkmanager
- **Config**: Redundant flash layout, 8 parallel jobs
- **Note**: No OP-TEE integration yet for Orin NX

### test.yml
- Inherits meta-arm CI configs for qemuarm64-secureboot and uefi-secureboot
- Enables root login for testing (empty-root-password)
- Uses systemd init manager

### Observations
1. QEMU target has OP-TEE basics but lacks PKCS#11/trust-service packages
2. Orin NX target has no OP-TEE integration yet
3. No custom OP-TEE configuration (OPTEE_OS_EXTRA_FLAGS, etc.)
4. `qemuarm64-secureboot.yml` does not include `openembedded.yml` which may be needed for extended packages
5. meta-secure-core features (TPM2, IMA, encrypted storage) not yet enabled in any kas config

---

## Feature Roadmap

### Phase 1: OP-TEE Foundation on QEMU (qemuarm64-secureboot)

| # | Feature | Description | Status |
|---|---------|-------------|--------|
| 1.1 | OP-TEE base build verification | Verify `kas build kas/qemuarm64-secureboot.yml` produces a bootable image with OP-TEE | NOT STARTED |
| 1.2 | OP-TEE PKCS#11 TA | Add `optee-os-ta-pkcs11` (or equivalent) to IMAGE_INSTALL, include `openembedded.yml` for dependencies | NOT STARTED |
| 1.3 | OP-TEE key storage | Configure OP-TEE secure storage backend, verify TEE supplicant persistence | NOT STARTED |
| 1.4 | OP-TEE trusted services | Add trusted-services recipes (crypto, internal-trusted-storage, secure-storage) from meta-arm | NOT STARTED |
| 1.5 | Custom OP-TEE configuration | Add OPTEE_OS extra build flags, custom platform config, TA signing keys | NOT STARTED |

### Phase 2: Platform Security with meta-secure-core

| # | Feature | Description | Status |
|---|---------|-------------|--------|
| 2.1 | Integrate meta-secure-core | Create kas config fragment to add meta-secure-core layers to bblayers | NOT STARTED |
| 2.2 | TPM 2.0 support | Enable TPM 2.0 packages and kernel config (swtpm for QEMU) | NOT STARTED |
| 2.3 | IMA (Integrity Measurement) | Enable IMA subsystem with measurement and appraisal policies | NOT STARTED |
| 2.4 | Encrypted storage | Enable dm-crypt based root filesystem or data volume encryption | NOT STARTED |
| 2.5 | UEFI Secure Boot chain | Signed bootloader and kernel with user key store | NOT STARTED |

### Phase 3: Orin NX Target

| # | Feature | Description | Status |
|---|---------|-------------|--------|
| 3.1 | OP-TEE on Orin NX | Integrate OP-TEE with meta-tegra (Jetson OP-TEE support) | NOT STARTED |
| 3.2 | PKCS#11 on Orin NX | Port PKCS#11 TA configuration to Orin NX target | NOT STARTED |
| 3.3 | Hardware security | Leverage Orin NX hardware security features (SE, fuses) | NOT STARTED |

### Phase 4: Testing and CI

| # | Feature | Description | Status |
|---|---------|-------------|--------|
| 4.1 | QEMU automated test | `kas build kas/test.yml` + testimage with OP-TEE test suite | NOT STARTED |
| 4.2 | PKCS#11 test suite | Runtime tests for PKCS#11 token operations | NOT STARTED |
| 4.3 | Security regression tests | IMA, secure boot, encrypted storage validation | NOT STARTED |

---

## Implementation Notes

### Build Commands

```bash
# QEMU ARM64 with secure boot + OP-TEE
kas build kas/qemuarm64-secureboot.yml

# Orin NX
kas build kas/orin-nx.yml

# Run QEMU for testing
kas shell kas/qemuarm64-secureboot.yml -c "runqemu nographic"

# Run automated tests
kas build kas/test.yml
```

### Key Decisions to Make
1. **meta-arm source**: Currently using Jon Mason's fork -- should we pin a release tag or use upstream?
2. **OP-TEE version**: Controlled by meta-arm; may need to override for custom patches
3. **Signing keys**: Need a strategy for dev vs production key management
4. **meta-secure-core compatibility**: Verify compatibility with wrynose branch

---

## Workflow

Each feature will be implemented as follows:
1. Devin creates implementation on a feature branch
2. Devin opens PR for review
3. Author reviews and tests locally
4. Once approved, merge and move to next feature

---

*Document created: 2025-05-21*
*Last updated: 2025-05-21*
