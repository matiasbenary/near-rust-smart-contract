# Best Practice: Build with cargo-near

Always use `cargo near build` instead of `cargo build` to compile NEAR smart contracts.

## Why It Matters

The NEAR SDK is designed to be compiled with the `cargo-near` tool, which:
- Sets up the correct WASM target (`wasm32-unknown-unknown`)
- Applies necessary optimizations for contract size
- Generates the ABI (Application Binary Interface) file
- Embeds the ABI into the contract for better tooling support
- Validates the build environment

Using `cargo build` directly will result in a compilation error starting from near-sdk v5.x.

## ❌ Incorrect

```bash
# This will fail with near-sdk v5.x+
cargo build

# Error message:
# error: 1. Use `cargo near build` instead of `cargo build` to compile your contract
#        Install cargo-near from https://github.com/near/cargo-near
```

```bash
# Also incorrect - missing target
cargo build --target wasm32-unknown-unknown
```

**Problems:**
- `cargo build` is blocked by the SDK to prevent incorrect compilation
- Missing ABI generation
- No embedded ABI for tooling
- Incorrect optimization settings

## ✅ Correct

### Step 1: Install cargo-near

```bash
# Install cargo-near
cargo install cargo-near

# Verify installation
cargo near --version
```

### Step 2: Build your contract

```bash
# For development (faster, non-reproducible)
cargo near build non-reproducible-wasm

# For production (requires git and reproducible_build config)
cargo near build reproducible-wasm
```

### Step 3: Find your built contract

After building, find your artifacts in:
- `target/near/<contract_name>.wasm` - The compiled contract
- `target/near/<contract_name>_abi.json` - The ABI file
- `target/near/<contract_name>_abi.zst` - Compressed embedded ABI

## Build Modes

| Mode | Use Case | Speed | Reproducible |
|------|----------|-------|--------------|
| `non-reproducible-wasm` | Local development | Fast | No |
| `reproducible-wasm` | Production release | Slower | Yes |

## Non-Interactive Builds

For CI/CD pipelines or non-TTY environments:

```bash
# Use the subcommand directly to avoid TTY requirement
cargo near build non-reproducible-wasm
```

## Cargo.toml Requirements

For reproducible builds, add to your `Cargo.toml`:

```toml
[package.metadata.near.reproducible_build]
# Optional: specify docker image for reproducible builds
image = "sourcescan/cargo-near:latest"
```

## Running Tests

Tests require the `unit-testing` feature:

```toml
[dev-dependencies]
near-sdk = { version = "5.x", features = ["unit-testing"] }
```

```bash
# Run tests (this works with cargo test)
cargo test
```

## Common Issues

### TTY Error
```
Error: The input device is not a TTY
```
**Solution:** Use `cargo near build non-reproducible-wasm` instead of just `cargo near build`

### Rust Version Mismatch
```
error: cannot install package `cargo-near X.Y.Z`, it requires rustc X.Y.Z
```
**Solution:** Install a compatible version: `cargo install cargo-near@<compatible-version>`

## Additional Considerations

- `cargo check` can be used for syntax checking: `cargo check --target wasm32-unknown-unknown`
- Always commit your changes before using `reproducible-wasm` mode
- The ABI is automatically generated and embedded in the contract

## References

- [cargo-near GitHub](https://github.com/near/cargo-near)
- [NEAR SDK Documentation](https://docs.near.org/sdk/rust/building-contracts)
- [NEP-330: Contract Source Metadata](https://github.com/near/NEPs/blob/master/neps/nep-0330.md)
