# Gas Optimization: Prepaid Gas for Cross-Contract Calls

Attach appropriate gas amounts for cross-contract calls (recommended: 30 TGas minimum).

## Why It Matters

Proper gas allocation:
- Prevents transaction failures due to insufficient gas
- Avoids wasting gas on overly conservative estimates
- Enables complex cross-contract interactions
- Ensures callbacks execute completely

## ❌ Incorrect

```rust
#[near]
impl Contract {
    pub fn call_external(&mut self) -> Promise {
        // Too little gas - will likely fail
        ext_contract::ext(self.external_contract.clone())
            .with_static_gas(Gas(5_000_000_000_000)) // 5 TGas - too low
            .external_method()
    }
    
    pub fn complex_call(&mut self) -> Promise {
        // No gas specified - uses default
        ext_contract::ext(self.external_contract.clone())
            .external_method()
            .then(
                Self::ext(env::current_account_id())
                    // Callback might not have enough gas
                    .callback()
            )
    }
}
```

**Problems:**
- Insufficient gas causes failures
- Callbacks may not execute
- Wasted transaction costs
- Poor user experience

## ✅ Correct

```rust
use near_sdk::Gas;

// Define gas constants
const GAS_FOR_EXTERNAL_CALL: Gas = Gas(30_000_000_000_000); // 30 TGas
const GAS_FOR_CALLBACK: Gas = Gas(20_000_000_000_000);      // 20 TGas
const GAS_FOR_FT_TRANSFER: Gas = Gas(50_000_000_000_000);   // 50 TGas

#[near]
impl Contract {
    pub fn call_external(&mut self) -> Promise {
        // Specify appropriate gas amount
        ext_contract::ext(self.external_contract.clone())
            .with_static_gas(GAS_FOR_EXTERNAL_CALL)
            .external_method()
    }
    
    pub fn complex_call(&mut self) -> Promise {
        // Allocate gas for call and callback
        ext_contract::ext(self.external_contract.clone())
            .with_static_gas(GAS_FOR_EXTERNAL_CALL)
            .external_method()
            .then(
                Self::ext(env::current_account_id())
                    .with_static_gas(GAS_FOR_CALLBACK)
                    .callback()
            )
    }
    
    pub fn ft_transfer_call(
        &mut self,
        receiver: AccountId,
        amount: U128
    ) -> Promise {
        // FT operations need more gas
        ext_ft::ext(self.ft_contract.clone())
            .with_static_gas(GAS_FOR_FT_TRANSFER)
            .with_attached_deposit(1)
            .ft_transfer(receiver.clone(), amount, None)
            .then(
                Self::ext(env::current_account_id())
                    .with_static_gas(GAS_FOR_CALLBACK)
                    .ft_callback(receiver, amount)
            )
    }
}
```

## Recommended Gas Amounts

| Operation | Recommended Gas | Notes |
|-----------|----------------|-------|
| Simple external call | 30 TGas | Minimum for most calls |
| Callback | 20-30 TGas | Depends on complexity |
| FT transfer | 50-100 TGas | Includes storage checks |
| NFT transfer | 50-100 TGas | Similar to FT |
| Complex operations | 100+ TGas | Multiple state changes |

## Key Points

1. **Use Constants**: Define gas constants for clarity
2. **30 TGas Minimum**: Start with at least 30 TGas for external calls
3. **Test in Sandbox**: Profile actual gas usage in integration tests
4. **Reserve for Callback**: Always allocate gas for callbacks
5. **Max Gas**: Remember max is 300 TGas per transaction

## Additional Resources

- [Gas Documentation](https://docs.near.org/protocol/gas)
- [Cross-Contract Calls](https://docs.near.org/smart-contracts/anatomy/crosscontract)
- [Testing Gas Usage](https://docs.near.org/smart-contracts/testing/introduction)
