---
name: near-smart-contracts
description: NEAR Protocol smart contract development in Rust. Use when writing, reviewing, or deploying NEAR smart contracts. Covers contract structure, state management, cross-contract calls, testing, security, and optimization patterns.
license: MIT
metadata:
  author: near
  version: "1.0.0"
---

# NEAR Smart Contracts Development

Comprehensive guide for developing secure and efficient smart contracts on NEAR Protocol using Rust and the NEAR SDK.

## When to Apply

Reference these guidelines when:
- Writing new NEAR smart contracts in Rust
- Reviewing existing contract code for security and optimization
- Implementing cross-contract calls and callbacks
- Managing contract state and storage
- Testing and deploying NEAR contracts
- Optimizing gas usage and performance

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Security & Safety | CRITICAL | `security-` |
| 2 | Build & Tooling | HIGH | `build-` |
| 3 | Contract Structure | HIGH | `structure-` |
| 4 | State Management | HIGH | `state-` |
| 5 | Cross-Contract Calls | MEDIUM-HIGH | `xcc-` |
| 6 | Gas Optimization | MEDIUM | `gas-` |
| 7 | Testing | MEDIUM | `testing-` |
| 8 | Best Practices | MEDIUM | `best-` |

## Quick Reference

### 1. Security & Safety (CRITICAL)

- `security-storage-checks` - Always validate storage operations and check deposits
- `security-access-control` - Implement proper access control using `predecessor_account_id`
- `security-reentrancy` - Protect against reentrancy attacks (update state before external calls)
- `security-overflow` - Use `overflow-checks = true` in Cargo.toml to prevent overflow
- `security-callback-validation` - Validate callback results and handle failures
- `security-private-callbacks` - Mark callbacks as `#[private]` to prevent external calls
- `security-yoctonear-validation` - Validate attached deposits with `#[payable]` functions
- `security-sybil-resistance` - Implement minimum deposit checks to prevent spam

### 2. Build & Tooling (HIGH)

- `build-cargo-near` - Use `cargo near build` instead of `cargo build` for compilation

### 3. Contract Structure (HIGH)

- `structure-near-bindgen` - Use `#[near(contract_state)]` macro for contract struct
- `structure-initialization` - Implement proper initialization with `#[init]` patterns
- `structure-versioning` - Plan for contract upgrades with versioning mechanisms
- `structure-events` - Use `env::log_str()` and structured event logging (NEP-297)
- `structure-standards` - Follow NEAR Enhancement Proposals (NEPs) for standards
- `structure-derive-macros` - Use `#[near(serializers = [json, borsh])]` appropriately
- `structure-panic-default` - Use `#[derive(PanicOnDefault)]` to require initialization

### 4. State Management (HIGH)

- `state-collections` - Use `IterableMap`, `Vector`, `LookupMap`, `UnorderedSet`, etc.
- `state-legacy-collections` - Enable `legacy` feature for `near_sdk::collections` (deprecated)
- `state-serialization` - Use Borsh for state, JSON for external interfaces
- `state-lazy-loading` - Use SDK collections for lazy loading to save gas
- `state-pagination` - Implement pagination with `.skip()` and `.take()` for large datasets
- `state-migration` - Plan state migration strategies using versioning
- `state-storage-cost` - Remember: 1 NEAR ≈ 100kb storage, contracts pay for their storage
- `state-unique-prefixes` - Use unique prefixes for all collections (avoid collisions)

### 5. Cross-Contract Calls (MEDIUM-HIGH)

- `xcc-promise-chaining` - Chain promises correctly
- `xcc-callback-handling` - Handle all callback scenarios (success, failure)
- `xcc-gas-management` - Allocate appropriate gas for cross-contract calls
- `xcc-error-handling` - Implement robust error handling
- `xcc-result-unwrap` - Never unwrap promise results without checks

### 6. Gas Optimization (MEDIUM)

- `gas-batch-operations` - Batch operations to reduce transaction costs
- `gas-minimal-state-reads` - Minimize state reads and writes (cache in memory)
- `gas-efficient-collections` - Choose appropriate collection types (LookupMap vs IterableMap)
- `gas-view-functions` - Mark read-only functions as view (`&self` in Rust)
- `gas-avoid-cloning` - Avoid unnecessary cloning of large data structures
- `gas-early-validation` - Use `require!` early to save gas on invalid inputs
- `gas-prepaid-gas` - Attach appropriate gas for cross-contract calls (recommended: 30 TGas)

### 7. Testing (MEDIUM)

- `testing-unit-tests` - Write comprehensive unit tests with mock contexts
- `testing-integration-tests` - Use `workspaces-rs` or `workspaces-js` for integration tests
- `testing-simulation-tests` - Test with sandbox environment before testnet/mainnet
- `testing-edge-cases` - Test boundary conditions, overflow, and empty states
- `testing-gas-profiling` - Profile gas usage in integration tests
- `testing-cross-contract` - Test cross-contract calls and callbacks thoroughly
- `testing-failure-scenarios` - Test promise failures and timeout scenarios

### 8. Best Practices (MEDIUM)

- `best-create-new-contract` - Use latest templates and SDK features for new contracts, remember run cargo near new
- `best-neartoken-transfers` - Use `NearToken` type instead of raw `u128` for transfers
- `best-panic-messages` - Provide clear, actionable panic messages
- `best-logging` - Use `env::log_str()` for debugging and event emission
- `best-documentation` - Document public methods, parameters, and complex logic
- `best-error-types` - Define custom error types or use descriptive strings
- `best-constants` - Use constants for magic numbers and configuration
- `best-require-macro` - Use `require!` instead of `assert!` for better error messages
- `best-promise-return` - Return promises from cross-contract calls for proper tracking
- `best-sdk-crates` - Reuse SDK-exported crates (borsh, serde, base64, etc.)
- `best-collection-cloning` - Clone values from `IterableMap::get()` before mutation

### Important: Collection Get Methods

When using `IterableMap::get()`, `LookupMap::get()`, etc., remember these methods return **references**, not owned values:

```rust
// ❌ WRONG - cannot mutate reference
let mut escrow = self.escrows.get(&id).expect("Not found");
escrow.status = NewStatus;  // ERROR: cannot assign to immutable

// ✅ CORRECT - clone before mutation
let mut escrow = self.escrows.get(&id).expect("Not found").clone();
escrow.status = NewStatus;
self.escrows.insert(id, escrow);  // Update storage

// ✅ CORRECT - extract needed values before insert
let mut escrow = self.escrows.get(&id).expect("Not found").clone();
let recipient = escrow.buyer.clone();  // Clone before insert
self.escrows.insert(id, escrow);
Promise::new(recipient).transfer(amount)
```

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/create-new-contract.md
rules/build-cargo-near.md
rules/security-storage-checks.md
rules/structure-near-bindgen.md
rules/state-legacy-collections.md
rules/best-neartoken-transfers.md
rules/best-require-macro.md
rules/gas-prepaid-gas.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
- Additional context and NEAR-specific considerations

## Latest Tools & Versions

> **CRITICAL REQUIREMENT**: Before creating ANY Cargo.toml, you MUST run `cargo search near-sdk` to get the actual latest version. NEVER use hardcoded versions from this document or any other documentation.

```bash
# ALWAYS run this FIRST before creating a new contract
cargo search near-sdk
```

### Rust Toolchain Version

> **CRITICAL**: NEAR VM currently requires Rust 1.86.0 or earlier for WASM compilation

**ALWAYS create a `rust-toolchain` file in your project root:**

```
1.86.0
```

This ensures consistent builds across environments and prevents compatibility issues.

### Development Tools
- **cargo-near**: Build and deploy contracts (`cargo install cargo-near`)
- **near-cli-rs**: Command-line interface (`cargo install near-cli-rs`)
- **rustc**: Rust compiler (1.86.0 for WASM compilation)
- **near-workspaces**: Integration testing

### Build Command

```bash
# Use this command to build NEAR contracts
cargo near build non-reproducible-wasm

# For production/reproducible builds (requires all git changes committed):
cargo near build reproducible-wasm
```

### SDK Versions
- **near-sdk**: Latest Rust SDK with improved macros
- **near-sdk-js**: Latest - TypeScript/JavaScript SDK
- **near-sdk-py**: Experimental - Python SDK

> **MANDATORY**: When starting a new project:
> 1. Run `cargo search near-sdk` to get the current version
> 2. Use that exact version in your Cargo.toml
> 3. Create a `rust-toolchain` file with version `1.86.0`
> 4. NEVER copy versions from examples or documentation - they become outdated

### Testing Setup

For unit tests, add to `Cargo.toml`:

```toml
[dev-dependencies]
near-sdk = { version = "5.24.0", features = ["unit-testing"] }
time = "=0.3.36"  # Lock to version compatible with Rust 1.86
```

The `unit-testing` feature is required to use near-sdk in test environments, and the `time` crate must be locked to avoid Rust version incompatibilities.

### Key Features (Latest SDK)
- Improved serialization macros: `#[near(serializers = [json, borsh])]`
- Better contract state macro: `#[near(contract_state)]`
- Enhanced collections: `IterableMap`, `Vector`, `LookupMap`, `UnorderedSet`
- Simplified cross-contract calls with high-level API
- Built-in support for NEP standards (FT, NFT, etc.)

## Common Patterns

### Contract Template
```rust
use near_sdk::{near, env, AccountId, PanicOnDefault};
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
    
    #[payable]
    pub fn do_something(&mut self, param: String) -> String {
        require!(
            env::predecessor_account_id() == self.owner,
            "Only owner"
        );
        // Implementation
        param
    }
}
```

## Essential Links

- [NEAR Docs](https://docs.near.org)
- [Smart Contract Quickstart](https://docs.near.org/smart-contracts/quickstart)
- [NEAR Examples](https://github.com/near-examples)
- [SDK Reference](https://docs.rs/near-sdk)
- [Playground](https://nearplay.app)
- [Security Best Practices](https://docs.near.org/smart-contracts/security/welcome)

## Resources

- NEAR Documentation: https://docs.near.org
- NEAR SDK Rust: https://docs.near.org/sdk/rust/introduction
- NEAR Examples: https://github.com/near-examples
- NEAR Standards (NEPs): https://github.com/near/NEPs
- Security Best Practices: https://docs.near.org/smart-contracts/security/welcome
- SDK API Reference: https://docs.rs/near-sdk

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
