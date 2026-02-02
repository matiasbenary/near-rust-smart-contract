# Best Practice: Use NearToken for Transfers

Always use the `NearToken` type instead of raw `u128` values when working with NEAR token amounts in transfers and deposits.

## Why It Matters

In near-sdk v5.x, the `Promise::transfer()` method and `env::attached_deposit()` use the `NearToken` type:
- Provides type safety for token amounts
- Prevents confusion between yoctoNEAR and NEAR
- Offers convenient conversion methods
- Makes code more readable and self-documenting

## ❌ Incorrect

```rust
use near_sdk::{env, Promise};

pub fn send_tokens(&mut self, recipient: AccountId, amount: u128) -> Promise {
    // Error: expected `NearToken`, found `u128`
    Promise::new(recipient).transfer(amount)
}

pub fn check_deposit(&self) {
    let deposit = env::attached_deposit();
    // Error: cannot compare NearToken with integer
    if deposit > 0 {
        // ...
    }
}
```

**Problems:**
- `transfer()` expects `NearToken`, not `u128`
- `env::attached_deposit()` returns `NearToken`, not `u128`
- Direct comparison with integers doesn't work

## ✅ Correct

```rust
use near_sdk::{env, near, AccountId, NearToken, Promise};

#[near]
impl Contract {
    pub fn send_tokens(&mut self, recipient: AccountId, amount: u128) -> Promise {
        // Convert u128 to NearToken
        Promise::new(recipient).transfer(NearToken::from_yoctonear(amount))
    }

    #[payable]
    pub fn check_deposit(&mut self) {
        let deposit = env::attached_deposit();

        // Use as_yoctonear() for comparisons
        if deposit.as_yoctonear() > 0 {
            // Process deposit
        }

        // Or compare with NearToken directly
        if deposit > NearToken::from_yoctonear(0) {
            // Process deposit
        }
    }
}
```

## NearToken Conversion Methods

```rust
use near_sdk::NearToken;

// Creating NearToken values
let one_near = NearToken::from_near(1);           // 1 NEAR
let half_near = NearToken::from_millinear(500);   // 0.5 NEAR
let tiny = NearToken::from_yoctonear(1000);       // 1000 yoctoNEAR

// Getting raw values
let yocto: u128 = one_near.as_yoctonear();        // 1_000_000_000_000_000_000_000_000
let near: u128 = one_near.as_near();              // 1
let millinear: u128 = one_near.as_millinear();    // 1000

// Arithmetic operations
let sum = one_near.saturating_add(half_near);
let diff = one_near.saturating_sub(half_near);
let product = one_near.saturating_mul(2);

// Comparisons
let is_greater = one_near > half_near;
let is_equal = one_near == NearToken::from_near(1);
```

## Common Patterns

### Storing Amounts in State

```rust
use near_sdk::{near, AccountId, NearToken};

#[near(contract_state)]
pub struct Contract {
    // Store as u128 for Borsh serialization
    balance: u128,
}

#[near]
impl Contract {
    #[payable]
    pub fn deposit(&mut self) {
        let deposit = env::attached_deposit();
        // Convert to u128 for storage
        self.balance += deposit.as_yoctonear();
    }

    pub fn withdraw(&mut self, amount: u128) -> Promise {
        require!(self.balance >= amount, "Insufficient balance");
        self.balance -= amount;
        // Convert back to NearToken for transfer
        Promise::new(env::predecessor_account_id())
            .transfer(NearToken::from_yoctonear(amount))
    }
}
```

### Validating Deposits

```rust
#[payable]
pub fn purchase(&mut self, item_id: u64) {
    let deposit = env::attached_deposit();
    let price = self.get_price(item_id);

    require!(
        deposit.as_yoctonear() >= price,
        format!("Insufficient deposit. Required: {} yoctoNEAR", price)
    );

    // Process purchase...
}
```

### Refunding Excess

```rust
#[payable]
pub fn exact_purchase(&mut self, item_id: u64) -> Promise {
    let deposit = env::attached_deposit();
    let price = NearToken::from_yoctonear(self.get_price(item_id));

    require!(deposit >= price, "Insufficient deposit");

    let refund = deposit.saturating_sub(price);

    // Process purchase...

    if refund.as_yoctonear() > 0 {
        Promise::new(env::predecessor_account_id()).transfer(refund)
    } else {
        // Return a no-op promise
        Promise::new(env::current_account_id())
    }
}
```

## Migration from u128

If migrating existing code that uses `Balance` (which was `u128`):

```rust
// Old code
type Balance = u128;

pub fn transfer(&mut self, recipient: AccountId, amount: Balance) -> Promise {
    Promise::new(recipient).transfer(amount)  // This no longer works
}

// New code
pub fn transfer(&mut self, recipient: AccountId, amount: u128) -> Promise {
    Promise::new(recipient).transfer(NearToken::from_yoctonear(amount))
}
```

## Unit Constants

```rust
use near_sdk::NearToken;

// Common amounts
const ONE_YOCTO: NearToken = NearToken::from_yoctonear(1);
const ONE_NEAR: NearToken = NearToken::from_near(1);

// For storage deposits (approximately)
const STORAGE_BYTE_COST: u128 = 10_000_000_000_000_000_000; // 0.00001 NEAR per byte
```

## Additional Considerations

- Always use `NearToken` for all token operations in new code
- The `Balance` type alias is no longer exported from near-sdk root
- Use `as_yoctonear()` when you need the raw `u128` value
- Use `from_yoctonear()` when converting from stored `u128` values
- Arithmetic operations use saturating math to prevent overflow

## References

- [NearToken Documentation](https://docs.rs/near-token/latest/near_token/struct.NearToken.html)
- [NEAR SDK Types](https://docs.rs/near-sdk/latest/near_sdk/index.html#types)
- [Understanding Gas and Tokens](https://docs.near.org/concepts/basics/gas)
