# Security: Storage Checks

Always validate storage operations to prevent unauthorized access and data corruption.

## Why It Matters

NEAR smart contracts store data on-chain with associated storage costs. Improper storage management can lead to:
- Unauthorized data access or modification
- Storage staking issues
- Gas inefficiency
- Contract vulnerabilities

## ❌ Incorrect

```rust
#[near]
impl Contract {
    pub fn update_user_data(&mut self, user_id: AccountId, data: String) {
        // No validation - anyone can update any user's data
        self.user_data.insert(user_id, data);
    }
}
```

**Problems:**
- No access control validation
- No storage deposit checks
- Allows unauthorized modifications

## ✅ Correct

```rust
use near_sdk::{near, env, AccountId, NearToken, require};

#[near]
impl Contract {
    #[payable]
    pub fn update_user_data(&mut self, data: String) {
        let user_id = env::predecessor_account_id();

        // Check storage deposit if adding new data
        let initial_storage = env::storage_usage();

        self.user_data.insert(user_id.clone(), data);

        // Calculate storage cost
        let storage_used = env::storage_usage().saturating_sub(initial_storage);
        let storage_cost = NearToken::from_yoctonear(
            storage_used as u128 * env::storage_byte_cost().as_yoctonear()
        );

        require!(
            env::attached_deposit() >= storage_cost,
            format!("Insufficient deposit. Required: {}", storage_cost)
        );

        // Refund excess deposit
        let refund = env::attached_deposit().saturating_sub(storage_cost);
        if refund > NearToken::from_yoctonear(1) {
            Promise::new(user_id).transfer(refund);
        }
    }

    pub fn update_own_data(&mut self, data: String) {
        // User can only update their own data
        let caller = env::predecessor_account_id();
        require!(
            self.user_data.contains_key(&caller),
            "No existing data to update"
        );
        self.user_data.insert(caller, data);
    }

    pub fn admin_update(&mut self, user_id: AccountId, data: String) {
        // Only owner can update any user's data
        require!(
            env::predecessor_account_id() == self.owner,
            "Only owner can perform admin updates"
        );
        self.user_data.insert(user_id, data);
    }
}
```

**Benefits:**
- Validates caller authorization with `require!`
- Ensures proper storage payment with `NearToken`
- Refunds excess deposits
- Clear error messages
- Uses modern SDK patterns

## Storage Cost Pattern

```rust
/// Helper to handle storage deposits with refunds
fn storage_deposit_check<T, F: FnOnce() -> T>(&self, action: F) -> T {
    let initial_storage = env::storage_usage();
    let result = action();

    let storage_used = env::storage_usage().saturating_sub(initial_storage);
    let required = NearToken::from_yoctonear(
        storage_used as u128 * env::storage_byte_cost().as_yoctonear()
    );

    require!(
        env::attached_deposit() >= required,
        format!("Requires {} for storage", required)
    );

    result
}
```

## Additional Considerations

- Use `env::predecessor_account_id()` for access control
- Use `NearToken` type for safer token arithmetic (SDK v5.x)
- Calculate and validate storage costs with `env::storage_byte_cost()`
- Refund excess deposits when appropriate
- Consider using storage management patterns from NEP-145
- Mark functions that accept deposits with `#[payable]`
- Test storage edge cases (add/update/delete)

## References

- [Storage Staking](https://docs.near.org/concepts/storage/storage-staking)
- [NEP-145: Storage Management](https://github.com/near/NEPs/blob/master/neps/nep-0145.md)
- [NearToken API](https://docs.rs/near-sdk/latest/near_sdk/struct.NearToken.html)
