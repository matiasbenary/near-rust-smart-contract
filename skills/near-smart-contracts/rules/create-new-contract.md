# Best Practice: Creating a New NEAR Contract

Always use the latest stable versions of packages and follow proper project setup when creating new NEAR smart contracts.

## CRITICAL: Always Check Latest Versions

**MANDATORY**: Before creating ANY Cargo.toml file, you MUST run:

```bash
cargo search near-sdk
```

This will show the current latest version. NEVER use hardcoded versions from examples or documentation - they become outdated quickly.

## Why It Matters

Using outdated dependencies can lead to:
- **Security vulnerabilities** - Older versions may have known exploits
- **Missing features** - New SDK versions include improved APIs and macros
- **Compatibility issues** - Older packages may not work with current tooling
- **Performance problems** - Newer versions often include gas optimizations
- **Deprecated patterns** - Old code patterns may stop working in future updates

## ❌ Incorrect

```toml
# Cargo.toml with outdated/hardcoded versions
[package]
name = "my-contract"
version = "0.1.0"
edition = "2018"

[lib]
crate-type = ["cdylib"]

[dependencies]
near-sdk = "4.0.0"  # Outdated version!

[profile.release]
opt-level = "z"
# Missing overflow-checks!
```

```rust
// Using deprecated patterns from old SDK
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};
use near_sdk::collections::UnorderedMap;
use near_sdk::{near_bindgen, AccountId};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize)]
pub struct Contract {
    data: UnorderedMap<AccountId, String>,
}
```

**Problems:**
- Using old SDK version with potential security issues
- Missing `overflow-checks = true` in profile
- Using deprecated `collections` module instead of `store`
- Missing `PanicOnDefault` derive
- Using old macro syntax

## ✅ Correct

### Step 1: Check Latest Versions Before Starting

Always verify the latest versions:

```bash
# Check latest near-sdk version
cargo search near-sdk

# Or visit crates.io
# https://crates.io/crates/near-sdk

# Check NEAR documentation for recommended versions
# https://docs.near.org/sdk/rust/introduction
```

### Step 2: Use Proper Cargo.toml

```toml
[package]
name = "my-contract"
version = "1.0.0"
edition = "2026"
authors = ["Your Name <your@email.com>"]
description = "Description of your contract"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
# MANDATORY: Run `cargo search near-sdk` to get the latest version
# NEVER use hardcoded versions from documentation - check crates.io!
near-sdk = "X.Y.Z"  # Replace with result from `cargo search near-sdk`
borsh = "lastest"  # Optional, if using Borsh serialization

[dev-dependencies]
near-sdk = { version = "X.Y.Z", features = ["unit-testing"] }  # Same version as above
tokio = { version = "1", features = ["full"] }

[profile.release]
codegen-units = 1
opt-level = "z"
lto = true
debug = false
panic = "abort"
# CRITICAL: Prevent integer overflow vulnerabilities
overflow-checks = true
```

### Step 3: Use Modern SDK Patterns

```rust
use near_sdk::{near, env, AccountId, NearToken, PanicOnDefault};
use near_sdk::store::IterableMap;

#[near(contract_state)]
#[derive(PanicOnDefault)]
pub struct Contract {
    owner: AccountId,
    data: IterableMap<AccountId, String>,
}

#[near]
impl Contract {
    #[init]
    pub fn new(owner: AccountId) -> Self {
        Self {
            owner,
            data: IterableMap::new(b"d"),
        }
    }

    pub fn get_owner(&self) -> &AccountId {
        &self.owner
    }
}
```

**Benefits:**
- Latest security patches and features
- Modern `#[near(contract_state)]` macro syntax
- Using `store` module with lazy-loading collections
- `PanicOnDefault` prevents uninitialized state access
- Proper release profile with overflow protection

## Package Version Guidelines

| Package | Purpose | Where to Check |
|---------|---------|----------------|
| `near-sdk` | Core SDK | https://crates.io/crates/near-sdk |
| `near-workspaces` | Testing | https://crates.io/crates/near-workspaces |
| `cargo-near` | Build tool | `cargo install cargo-near` |
| `near-cli-rs` | CLI tool | https://github.com/near/near-cli-rs |

## Version Checking Commands

```bash
# Update cargo-near to latest
cargo install cargo-near --force

# Check for outdated dependencies in your project
cargo outdated

# Update dependencies (review changes before committing)
cargo update

# Verify your contract builds with latest tooling
cargo near build
```

## Modern SDK Features (v5.x+)

The latest SDK includes significant improvements:

```rust
// New contract state macro (replaces #[near_bindgen] + derives)
#[near(contract_state)]
#[derive(PanicOnDefault)]
pub struct Contract { /* ... */ }

// Simplified method macro
#[near]
impl Contract { /* ... */ }

// Flexible serialization options
#[near(serializers = [json, borsh])]
pub struct MyStruct { /* ... */ }

// Modern collections (lazy-loading by default)
use near_sdk::store::{IterableMap, IterableSet, Vector, LookupMap, LookupSet};

// NearToken type for safer token handling
use near_sdk::NearToken;
let amount = NearToken::from_near(1);
```

## Project Initialization Checklist

- [ ] Check latest `near-sdk` version on crates.io
- [ ] Use Rust edition 2021
- [ ] Set `overflow-checks = true` in release profile
- [ ] Include both `cdylib` and `rlib` crate types
- [ ] Use `#[near(contract_state)]` macro
- [ ] Derive `PanicOnDefault`
- [ ] Use `store` collections (not deprecated `collections`)
- [ ] Add unit testing dependencies
- [ ] Configure proper release optimizations

## Additional Considerations

- **Run `cargo update` regularly** to get patch updates
- **Review changelogs** before major version upgrades
- **Test thoroughly** after updating dependencies
- **Use `cargo audit`** to check for security advisories
- **Pin exact versions** in production only after testing

## References

- [NEAR SDK Changelog](https://github.com/near/near-sdk-rs/blob/master/CHANGELOG.md)
- [NEAR SDK Documentation](https://docs.near.org/sdk/rust/introduction)
- [Crates.io - near-sdk](https://crates.io/crates/near-sdk)
- [NEAR Examples](https://github.com/near-examples)
- [Cargo Book - Dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html)
