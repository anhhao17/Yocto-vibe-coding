# QEMU OP-TEE Trust Platform

Yocto-based distribution for building a hardened OS with OP-TEE (Open Portable Trusted Execution Environment) support. Built with the [kas](https://kas.readthedocs.io/) build tool on top of OpenEmbedded/Yocto (wrynose branch).

## Targets

| Target | Machine | KAS Config |
|--------|---------|------------|
| NVIDIA Jetson Orin NX | `p3768-0000-p3767-0000` | `kas/orin-nx.yml` |
| QEMU ARM64 Secure Boot | `qemuarm64-secureboot` | `kas/qemuarm64-secureboot.yml` |

## Features

### OP-TEE
- Deep understanding of TEE architecture
- PKCS#11 trusted application for cryptographic token interface
- Secure key storage via OP-TEE secure storage
- All trusted services (crypto, internal-trusted-storage, secure-storage)
- Custom OP-TEE configuration and TA development

### Platform Security (via meta-secure-core) -- TBD
- TPM 2.0 support
- IMA (Integrity Measurement Architecture)
- Encrypted storage (dm-crypt)
- UEFI Secure Boot chain

## Quick Start

```bash
# Build QEMU ARM64 with secure boot + OP-TEE
kas build kas/qemuarm64-secureboot.yml

# Build for Orin NX
kas build kas/orin-nx.yml
```

## Project Plan

See [docs/PLAN.md](docs/PLAN.md) for the detailed implementation roadmap and status.

## License

MIT License - see [LICENSE](LICENSE) for details.
