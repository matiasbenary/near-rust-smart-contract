# State Management: Legacy Collections Feature

When using `near_sdk::collections` (the older collection types), you must enable the `legacy` feature in near-sdk v5.x+.

## Why It Matters

In near-sdk v5.x, the collections module was moved behind a feature flag:
- The new `near_sdk::store` module is the recommended approach (lazy-loading by default)
- The old `near_sdk::collections` module requires the `legacy` feature
- Without the feature, you'll get a compile error about missing modules

## ❌ Incorrect

```toml
# Cargo.toml - missing legacy feature
[dependencies]
near-sdk = "5.24.0"
```

```rust
// This will fail to compile without the legacy feature
use near_sdk::collections::UnorderedMap;

// Error: could not find `collections` in `near_sdk`
// note: the item is gated behind the `legacy` feature
```

**Problems:**
- `collections` module is not available without `legacy` feature
- Code using `UnorderedMap`, `LookupMap`, `Vector` from `collections` won't compile

## ✅ Correct

### Option 1: Enable Legacy Feature (for existing code)

```toml
# Cargo.toml
[dependencies]
near-sdk = { version = "5.24.0", features = ["legacy"] }
```

```rust
// Now this works
use near_sdk::collections::UnorderedMap;
use near_sdk::collections::LookupMap;
use near_sdk::collections::Vector;
```

### Option 2: Migrate to Store Module (recommended for new code)

```toml
# Cargo.toml - no legacy feature needed
[dependencies]
near-sdk = "5.24.0"
```

```rust
// Use the new store module instead
use near_sdk::store::IterableMap;      // replaces UnorderedMap
use near_sdk::store::LookupMap;        // same name, new implementation
use near_sdk::store::Vector;           // same name, new implementation
use near_sdk::store::IterableSet;      // replaces UnorderedSet
use near_sdk::store::LookupSet;        // same name, new implementation
```

## Collections Comparison

| Legacy (`collections`) | Modern (`store`) | Notes |
|----------------------|------------------|-------|
| `UnorderedMap` | `IterableMap` | Both are iterable |
| `LookupMap` | `LookupMap` | Same name, improved API |
| `Vector` | `Vector` | Same name, improved API |
| `UnorderedSet` | `IterableSet` | Both are iterable |
| `LookupSet` | `LookupSet` | Same name, improved API |
| `TreeMap` | `IterableMap` | Use IterableMap instead |

## Migration Example

### Before (Legacy)

```rust
use near_sdk::collections::UnorderedMap;
use near_sdk::borsh::{BorshDeserialize, BorshSerialize};
use near_sdk::{near_bindgen, AccountId, PanicOnDefault};

#[near_bindgen]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Contract {
    data: UnorderedMap<AccountId, String>,
}

#[near_bindgen]
impl Contract {
    #[init]
    pub fn new() -> Self {
        Self {
            data: UnorderedMap::new(b"d"),
        }
    }

    pub fn get(&self, key: AccountId) -> Option<String> {
        self.data.get(&key)
    }

    pub fn set(&mut self, key: AccountId, value: String) {
        self.data.insert(&key, &value);
    }
}
```

### After (Modern)

```rust
use near_sdk::store::IterableMap;
use near_sdk::{near, AccountId, PanicOnDefault};

#[near(contract_state)]
#[derive(PanicOnDefault)]
pub struct Contract {
    data: IterableMap<AccountId, String>,
}

#[near]
impl Contract {
    #[init]
    pub fn new() -> Self {
        Self {
            data: IterableMap::new(b"d"),
        }
    }

    pub fn get(&self, key: &AccountId) -> Option<&String> {
        self.data.get(key)
    }

    pub fn set(&mut self, key: AccountId, value: String) {
        self.data.insert(key, value);
    }
}
```

## Key Differences

### API Changes
- `store` collections use owned values in `insert()`: `map.insert(key, value)` instead of `map.insert(&key, &value)`
- `get()` returns `Option<&V>` instead of `Option<V>` (no cloning)
- Iteration is more ergonomic

### Performance
- `store` collections have better lazy-loading behavior
- Reduced gas costs for partial reads
- More efficient iteration

## When to Use Legacy

Use the `legacy` feature when:
- Migrating existing contracts that use `collections`
- You need compatibility with older SDK patterns
- Gradual migration is preferred over full rewrite

## Additional Considerations

- New projects should use `store` module
- `legacy` feature also enables other deprecated items
- Both modules can coexist if needed (with different prefixes)
- State migration is NOT needed - the storage format is compatible

## References

- [NEAR SDK Store Module](https://docs.rs/near-sdk/latest/near_sdk/store/index.html)
- [NEAR SDK Collections (Legacy)](https://docs.rs/near-sdk/latest/near_sdk/collections/index.html)
- [Migration Guide](https://docs.near.org/sdk/rust/contract-upgrade)
