# Best Practice: Return Promises

Always return promises from cross-contract calls for proper tracking and error handling.

## Why It Matters

Returning promises:
- Allows callers to wait for results
- Enables proper error tracking
- Helps NEAR Explorer mark transaction chains
- Prevents false-positive success indicators

## ❌ Incorrect

```rust
#[near]
impl Contract {
    pub fn withdraw_100(&mut self, receiver_id: AccountId) {
        // Not returning the promise
        Promise::new(receiver_id).transfer(100);
        // Caller can't track if transfer succeeded
    }
    
    #[payable]
    pub fn ft_transfer_call(&mut self, receiver_id: AccountId, amount: U128) {
        // Promise created but not returned
        let promise = ext_ft::ext(self.ft_contract.clone())
            .with_attached_deposit(1)
            .ft_transfer(receiver_id, amount, None);
        
        // No callback, no return
    }
}
```

**Problems:**
- Caller can't wait for completion
- Failures are hidden
- Explorer shows incomplete transaction status
- No way to handle errors

## ✅ Correct

```rust
#[near]
impl Contract {
    pub fn withdraw_100(&mut self, receiver_id: AccountId) -> Promise {
        // Return the promise for proper tracking
        Promise::new(receiver_id).transfer(100)
    }
    
    #[payable]
    pub fn ft_transfer_call(
        &mut self,
        receiver_id: AccountId,
        amount: U128
    ) -> Promise {
        // Create promise and return it
        ext_ft::ext(self.ft_contract.clone())
            .with_attached_deposit(1)
            .ft_transfer(receiver_id.clone(), amount, None)
            .then(
                // Add callback for proper error handling
                Self::ext(env::current_account_id())
                    .ft_transfer_callback(receiver_id, amount)
            )
    }
    
    #[private]
    pub fn ft_transfer_callback(
        &mut self,
        receiver_id: AccountId,
        amount: U128
    ) -> bool {
        // Handle promise result
        match env::promise_result(0) {
            PromiseResult::Successful(_) => {
                log!("Transfer to {} succeeded", receiver_id);
                true
            }
            _ => {
                log!("Transfer failed, rolling back");
                // Rollback state changes
                false
            }
        }
    }
}
```

## Handling Promises in Unit Tests

In unit tests, Promises don't actually execute (no real NEAR runtime). When calling methods that return `Promise`, use `let _ = ...` to explicitly ignore the return value and avoid compiler warnings.

### ❌ Incorrect (causes warning)

```rust
#[test]
fn test_complete_escrow() {
    // ...setup...

    // Warning: unused `near_sdk::Promise` that must be used
    contract.complete_escrow(escrow_id);

    // ...assertions...
}
```

### ✅ Correct

```rust
#[test]
fn test_complete_escrow() {
    // ...setup...

    // Explicitly ignore the Promise return value in tests
    let _ = contract.complete_escrow(escrow_id);

    // ...assertions...
}
```

**Why `let _ = ...`?**
- Explicitly signals intent to ignore the value
- Suppresses the `unused_must_use` warning
- Cleaner than using `.detach()` in test context
- Makes it clear this is intentional, not an oversight

## Key Points

1. **Always Return**: Return `Promise` from cross-contract calls
2. **Chain Callbacks**: Use `.then()` for callbacks
3. **Handle Results**: Check promise results in callbacks
4. **Track Status**: Enables proper transaction tracking
5. **Unit Tests**: Use `let _ = ...` to ignore Promise returns in tests

## Additional Resources

- [Cross-Contract Calls](https://docs.near.org/smart-contracts/anatomy/crosscontract)
- [Callbacks](https://docs.near.org/smart-contracts/anatomy/crosscontract#callbacks)
- [Best Practices](https://docs.near.org/smart-contracts/anatomy/best-practices#return-promise)
