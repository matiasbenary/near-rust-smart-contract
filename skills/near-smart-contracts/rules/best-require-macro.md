# Best Practice: Use `require!` Macro

Use `require!` macro for early validation instead of `assert!` for better error handling.

## Why It Matters

The `require!` macro:
- Provides cleaner error messages
- Validates input early to save gas
- Exits immediately on failure
- More idiomatic in NEAR contracts

## ❌ Incorrect

```rust
#[near]
impl Contract {
    pub fn set_fee(&mut self, new_fee: Fee) {
        // Using assert! provides less clear errors
        assert!(
            env::predecessor_account_id() == self.owner_id,
            "Owner's method"
        );
        new_fee.assert_valid();
        self.internal_set_fee(new_fee);
    }
}
```

**Problems:**
- Less descriptive error format
- Older pattern (pre SDK 4.0)
- No debug information in error

## ✅ Correct

```rust
#[near]
impl Contract {
    pub fn set_fee(&mut self, new_fee: Fee) {
        // Validate early with require!
        require!(
            env::predecessor_account_id() == self.owner_id,
            "Only owner can set fee"
        );
        require!(new_fee > 0, "Fee must be positive");
        
        self.internal_set_fee(new_fee);
    }
    
    pub fn transfer(&mut self, receiver: AccountId, amount: U128) {
        // Multiple require! statements for early validation
        require!(!receiver.is_empty(), "Receiver cannot be empty");
        require!(amount.0 > 0, "Amount must be greater than 0");
        
        let sender = env::predecessor_account_id();
        let balance = self.balances.get(&sender).unwrap_or(0);
        require!(balance >= amount.0, "Insufficient balance");
        
        // Only proceed after all validations pass
        self.internal_transfer(sender, receiver, amount.0);
    }
}
```

## Key Points

1. **Early Validation**: Place `require!` statements at the beginning
2. **Clear Messages**: Provide descriptive error messages
3. **Gas Efficiency**: Fail fast to save gas
4. **Multiple Checks**: Chain multiple `require!` statements for comprehensive validation

## Additional Resources

- [NEAR Best Practices](https://docs.near.org/smart-contracts/anatomy/best-practices)
- [Environment Module](https://docs.near.org/smart-contracts/anatomy/environment)
