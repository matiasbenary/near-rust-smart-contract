# State: Collections

Use `near_sdk::store` collections for efficient on-chain storage instead of standard Rust collections.

## Why It Matters

Standard Rust collections (HashMap, Vec, etc.) load entire data structures into memory, which:
- Wastes gas on large datasets
- Can exceed gas limits
- Inefficiently uses storage
- Causes performance issues

NEAR SDK `store` collections (IterableMap, Vector, LookupMap, etc.) provide lazy loading and efficient storage.

## ❌ Incorrect

```rust
use std::collections::HashMap;

#[near(contract_state)]
pub struct Contract {
    // Don't use standard Rust collections for contract state
    pub users: HashMap<AccountId, User>,
    pub items: Vec<Item>,
}

impl Contract {
    pub fn add_user(&mut self, user: User) {
        // Entire HashMap is loaded and saved on every operation
        self.users.insert(env::predecessor_account_id(), user);
    }
}
```

**Problems:**
- Entire collection loaded into memory on every access
- High gas costs for large datasets
- Can hit gas limits with moderate data
- Inefficient serialization/deserialization

## ✅ Correct

```rust
use near_sdk::{near, env, AccountId, PanicOnDefault, IntoStorageKey};
use near_sdk::store::{IterableMap, IterableSet, Vector, LookupMap, LookupSet};

#[near(contract_state)]
#[derive(PanicOnDefault)]
pub struct Contract {
    // Use near_sdk::store collections for efficient storage
    pub users: IterableMap<AccountId, User>,
    pub items: Vector<Item>,
    pub quick_lookup: LookupMap<AccountId, UserMetadata>,
}

#[near]
impl Contract {
    #[init]
    pub fn new() -> Self {
        Self {
            // Use simple byte prefixes - must be unique per collection
            users: IterableMap::new(b"u"),
            items: Vector::new(b"i"),
            quick_lookup: LookupMap::new(b"m"),
        }
    }

    pub fn add_user(&mut self, user: User) {
        // Only the specific entry is loaded and saved
        self.users.insert(env::predecessor_account_id(), user);
    }

    pub fn get_user(&self, account_id: &AccountId) -> Option<&User> {
        self.users.get(account_id)
    }
}
```

**Benefits:**
- Lazy loading - only accessed data is loaded
- Lower gas costs for operations
- Scales to large datasets
- Efficient memory usage
- Modern SDK v5.x patterns

## Collection Types Guide (near_sdk::store)

| Collection | Use Case | Iteration | Notes |
|------------|----------|-----------|-------|
| `LookupMap` | Key-value, no iteration | No | Fastest for lookups |
| `IterableMap` | Key-value with iteration | Yes | Use when you need to list entries |
| `LookupSet` | Unique values, no iteration | No | Fastest for membership checks |
| `IterableSet` | Unique values with iteration | Yes | Use when you need to list members |
| `Vector` | Ordered list | Yes | Index access, push/pop |

## Nested Collections

For nested structures, use `near_sdk::store::Lazy` or nested collection patterns:

```rust
use near_sdk::store::{LookupMap, Vector, Lazy};

#[near(contract_state)]
pub struct Contract {
    // Nested collection pattern - each user has their own vector
    user_items: LookupMap<AccountId, Vector<Item>>,

    // Or use Lazy for expensive-to-load data
    large_config: Lazy<Config>,
}
```

## Additional Considerations

- Use unique byte prefixes for all collections (`b"u"`, `b"i"`, etc.)
- Choose the right collection type for your access patterns
- Use `LookupMap`/`LookupSet` when you don't need iteration (more gas efficient)
- Implement pagination with `.iter().skip(offset).take(limit)` for large datasets
- Never mix standard Rust collections with SDK collections for persistent state
- The old `near_sdk::collections` module is deprecated - use `near_sdk::store`

## References

- [Collections](https://docs.near.org/sdk/rust/contract-structure/collections)
- [Storage Management](https://docs.near.org/concepts/storage/storage-staking)
- [near-sdk-rs store module](https://docs.rs/near-sdk/latest/near_sdk/store/index.html)
