# Passkey Auth Kit — Core

> Rust/Soroban Secp256r1 verification contracts and cryptographic library for WebAuthn-based biometric transaction signing on Stellar.

This repository contains the smart contract suite and cryptographic backend for Passkey Auth Kit — implementing native Secp256r1 signature verification, biometric multi-sig structures, and key-rotation policy enforcement directly on Soroban.

> **Related repository:** [`passkey-auth-kit-frontend`](https://github.com/rafikace/passkey-auth-kit) — TypeScript wallet SDK and Next.js demo wallet.

---

## Why This Exists

Standard public-private key cryptography introduces significant UX friction for retail and institutional web3 adoption:

- **Secret phrase and private key management** is a primary vector for user error and phishing attacks.
- **Mobile dApp users** must frequently copy and paste private keys, exposing credentials to clipboard exploits.
- **Existing smart contract wallets** struggle to verify WebAuthn signatures natively without incurring high on-chain costs.
- **Developers lack modular, ready-to-use contract templates** to implement biometric multi-sig structures with customizable key-rotation policies.

---

## Repository Structure

```
.
├── packages/
│   └── contracts-secp256r1/           # Rust/Soroban Secp256r1 verification contract suite
│       ├── src/
│       │   ├── lib.rs                 # Contract entry point and public interface
│       │   ├── verify.rs              # Core Secp256r1 / WebAuthn signature verification logic
│       │   ├── keyset.rs              # Biometric keyset registration and key-rotation policies
│       │   └── errors.rs              # Typed contract error definitions
│       ├── tests/
│       │   ├── webauthn_vectors.rs    # Test vectors derived from WebAuthn Level 3 specification
│       │   └── integration.rs         # Full transaction signing and verification integration tests
│       └── Cargo.toml
├── docs/                              # WebAuthn payload decoding, key rotation strategies,
│                                      # and cryptographic security assumptions
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                     # Lint, Clippy, and full test suite on every push and PR
│   │   ├── deploy-testnet.yml         # Deploy verified WASM contract to Stellar Testnet on merge to main
│   │   └── audit.yml                  # Nightly cryptographic regression tests against WebAuthn vectors
│   └── SECURITY.md
├── .husky/
│   ├── pre-commit                     # Runs cargo fmt --check and cargo clippy before every commit
│   └── pre-push                       # Runs full cargo test --release before pushing to remote
├── Dockerfile                         # Reproducible WASM build environment for contract compilation
├── docker-compose.yml                 # Local orchestration: Stellar Quickstart node and contract deployment
├── .env.example
└── README.md
```

---

## Quick Start

### Prerequisites

- Rust stable + Cargo
- `wasm32-unknown-unknown` target
- Stellar CLI configured for Testnet or Mainnet
- Docker + Docker Compose

### Environment Setup

```bash
cp .env.example .env
```

```env
# Stellar network
STELLAR_NETWORK=testnet                          # or mainnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
STELLAR_SOURCE_ACCOUNT=your_deployer_account_alias

# Contract
CONTRACT_ADMIN_ADDRESS=your_admin_stellar_address
```

---

## Development

### Install Husky Git Hooks

Husky is initialised via a thin Node.js dev dependency to manage hook scripts:

```bash
npm install
npm run prepare
```

| Hook | Actions |
|---|---|
| `pre-commit` | `cargo fmt --check`, `cargo clippy -- -D warnings` |
| `pre-push` | `cargo test --release --all-targets` |

---

### Run Cryptographic Verification Tests

The test suite validates the Secp256r1 implementation against official WebAuthn Level 3 specification test vectors:

```bash
cd packages/contracts-secp256r1
cargo test --release
```

To run only the WebAuthn vector tests:

```bash
cargo test --release webauthn_vectors
```

### Build Contract WASM

```bash
cd packages/contracts-secp256r1
cargo build --target wasm32-unknown-unknown --release
```

### Deploy to Testnet

```bash
stellar contract deploy \
  --wasm packages/contracts-secp256r1/target/wasm32-unknown-unknown/release/contracts_secp256r1.wasm \
  --network testnet \
  --source-account $STELLAR_SOURCE_ACCOUNT
```

---

## Running Locally with Docker

The Docker setup provides a reproducible WASM build environment and a local Stellar Quickstart node for end-to-end contract testing without connecting to Testnet:

```bash
docker compose up --build
```

| Service | Description | Local Port |
|---|---|---|
| `stellar-quickstart` | Local Stellar + Soroban RPC node | `8000` |
| `contract-builder` | Reproducible Rust/WASM build environment | — |

Deploy the contract against the local node:

```bash
docker compose run contract-builder stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/contracts_secp256r1.wasm \
  --network standalone \
  --rpc-url http://stellar-quickstart:8000/soroban/rpc \
  --source-account root
```

Tear down and remove volumes:

```bash
docker compose down -v
```

---

## CI/CD Workflows

| Workflow | Trigger | Actions |
|---|---|---|
| `ci.yml` | Every push and pull request | `cargo fmt`, Clippy, WASM build, full test suite |
| `deploy-testnet.yml` | Merge to `main` | Builds and deploys verified WASM to Stellar Testnet |
| `audit.yml` | Nightly schedule | Cryptographic regression tests against WebAuthn Level 3 vectors |

---

## Security

To report a cryptographic or logic vulnerability, open an issue.

The Secp256r1 verification implementation is extensively tested using test vectors from the WebAuthn specification to guarantee transaction safety. All contract deployments use reproducible WASM builds to ensure the deployed bytecode is auditable and matches the published source.

---

## License

This project is currently **unlicensed**. Usage, redistribution, and modification restrictions are in place while the cryptographic implementation undergoes review. To request access or discuss licensing, please [open an issue](../../issues).
