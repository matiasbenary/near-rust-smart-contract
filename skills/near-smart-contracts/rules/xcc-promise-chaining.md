# Cross-Contract Calls: Promise Chaining

Chain promises correctly for sequential cross-contract calls and callback handling.

## Why It Matters

Cross-contract calls on NEAR are asynchronous and use promises. Proper promise chaining:
- Ensures correct execution order
- Handles errors gracefully
- Manages gas allocation
- Prevents lost funds or state inconsistencies

## ❌ Incorrect

```rust
#[near]
impl Contract {
    pub fn transfer_and_notify(&mut self, token: AccountId, amount: U128) {
        // Incorrect: Not chaining promises properly
        ext_token::ext(token.clone())
            .with_static_gas(Gas::from_tgas(5))
            .transfer(env::current_account_id(), amount);

        // This will execute immediately, not after transfer completes
        self.mark_transferred(amount);
    }
}
```

**Problems:**
- State changes happen before cross-contract call completes
- No error handling if transfer fails
- Race conditions possible
- Can't rollback if transfer fails

## ✅ Correct

```rust
use near_sdk::{near, env, Promise, Gas, PromiseResult, AccountId};
use near_sdk::json_types::U128;

// Define external contract interface
#[near(ext)]
#[ext_contract(ext_token)]
pub trait ExtToken {
    fn ft_transfer(&mut self, receiver_id: AccountId, amount: U128, memo: Option<String>);
}

const GAS_FOR_FT_TRANSFER: Gas = Gas::from_tgas(30);
const GAS_FOR_CALLBACK: Gas = Gas::from_tgas(20);

#[near]
impl Contract {
    pub fn transfer_and_notify(&mut self, token: AccountId, amount: U128) -> Promise {
        // Chain the promise with a callback using high-level API
        ext_token::ext(token)
            .with_static_gas(GAS_FOR_FT_TRANSFER)
            .with_attached_deposit(1) // 1 yoctoNEAR for ft_transfer
            .ft_transfer(env::current_account_id(), amount, None)
            .then(
                Self::ext(env::current_account_id())
                    .with_static_gas(GAS_FOR_CALLBACK)
                    .on_transfer_complete(amount)
            )
    }

    #[private]
    pub fn on_transfer_complete(&mut self, amount: U128) -> bool {
        // Check if the previous promise succeeded
        match env::promise_result(0) {
            PromiseResult::Successful(_) => {
                self.mark_transferred(amount);
                true
            }
            PromiseResult::Failed => {
                env::log_str("Transfer failed, rolling back");
                false
            }
        }
    }
}
```

**Benefits:**
- Proper promise chaining with callbacks
- Error handling with promise results
- State changes only after verification
- Marked callback as `#[private]` for security
- Explicit gas allocation with constants
- Uses modern high-level API (`ext()` syntax)

## Promise Patterns

### Multiple Sequential Calls
```rust
ext_a::ext(contract_a)
    .with_static_gas(Gas::from_tgas(30))
    .method_a()
    .then(
        ext_b::ext(contract_b)
            .with_static_gas(Gas::from_tgas(30))
            .method_b()
    )
    .then(
        Self::ext(env::current_account_id())
            .with_static_gas(Gas::from_tgas(20))
            .final_callback()
    )
```

### Parallel Calls with Join
```rust
let promise_a = ext_a::ext(contract_a).method_a();
let promise_b = ext_b::ext(contract_b).method_b();

promise_a
    .and(promise_b)
    .then(
        Self::ext(env::current_account_id())
            .callback_for_both()
    )
```

## Additional Considerations

- Always check `env::promise_result()` in callbacks
- Mark callbacks with `#[private]` to prevent external calls
- Allocate sufficient gas for callbacks (at least 20 TGas recommended)
- Handle all promise result cases (`Successful`, `Failed`)
- Note: `PromiseResult::NotReady` was removed in SDK v5.x
- Use `.and()` for parallel independent operations
- Consider partial failures in batch operations
- Use `Gas::from_tgas(30)` for readable gas amounts

## References

- [Cross-Contract Calls](https://docs.near.org/smart-contracts/anatomy/crosscontract)
- [Callbacks](https://docs.near.org/smart-contracts/anatomy/crosscontract#callbacks)
- [Promises API](https://docs.rs/near-sdk/latest/near_sdk/struct.Promise.html)
