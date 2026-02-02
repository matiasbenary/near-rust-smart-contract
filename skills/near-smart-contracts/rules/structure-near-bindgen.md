# Structure: Contract State Macro

Use `#[near(contract_state)]` macro to properly expose contract state and `#[near]` for implementation blocks.

## Why It Matters

The `#[near(contract_state)]` and `#[near]` macros are essential for NEAR smart contracts as they:
- Automatically generate serialization/deserialization code
- Expose methods to the NEAR runtime
- Handle contract state management
- Enable proper initialization

## ❌ Incorrect

```rust
// Missing contract macros - contract won't work
pub struct Contract {
    pub owner: AccountId,
    pub data: IterableMap<AccountId, String>,
}

impl Contract {
    pub fn new(owner: AccountId) -> Self {
        Self {
            owner,
            data: IterableMap::new(b"d"),
        }
    }
}
```

**Problems:**
- Contract methods won't be callable
- State won't be properly serialized
- Runtime won't recognize the contract
- No initialization validation

## ✅ Correct

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

    pub fn get_owner(&self) -> &AccountId {
        &self.owner
    }
}
```

**Benefits:**
- Modern SDK v5.x syntax - cleaner and more maintainable
- Automatic Borsh serialization (no manual derives needed)
- Methods are callable from outside
- State management works correctly
- Initialization is validated with `#[init]`
- Uses `PanicOnDefault` to prevent uninitialized state

## Macro Reference

| Macro | Purpose | Replaces |
|-------|---------|----------|
| `#[near(contract_state)]` | Define contract struct with auto-serialization | `#[near_bindgen]` + Borsh derives |
| `#[near]` | Mark impl block for exposed methods | `#[near_bindgen]` on impl |
| `#[near(serializers = [json, borsh])]` | Custom serialization for types | Manual serde/borsh derives |
| `#[near(ext)]` | Define external contract interface | `#[ext_contract]` |

## Additional Considerations

- Use `#[near(contract_state)]` instead of old `#[near_bindgen]` + derives
- Use `#[init]` for initialization methods
- Derive `PanicOnDefault` to prevent accidental default initialization
- Mark view functions with `&self` (not `&mut self`)
- Use unique prefixes for collections (single byte like `b"d"`)
- Use `near_sdk::store` collections (not deprecated `collections` module)

## References

- [Smart Contract Structure](https://docs.near.org/sdk/rust/contract-structure/near-bindgen)
- [Initialization](https://docs.near.org/sdk/rust/contract-structure/initialization)
- [NEAR SDK Rust v5.x](https://docs.rs/near-sdk)
