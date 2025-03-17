# 💊 Crowbar Sonic SVM Smart Contract

The **Crowbar Smart Contract forking** is an innovative platform  designed to allow users to create tokens, markets, and pools on Sonic SVM. This comprehensive project offers not only the bonding curve but also more for managing token authorities, customizing token properties, white list and handling liquidity pools with advanced functionalities.


## Development progress

| **Core bonding curve features**  | **TO DO** | **In Progress** | PoC | **Refined** | **Audited** |
|------------------------------|-----------|-----------------|-----|-------------|-------------|
| Quote with Sol               |           |                 |✅    |             |             |
| Quote with SPL token         |           | 🚧              |     |             |             |
| Migration on Completion      |           |                 | ✅   |             |             |
| Token Management             |           |                 | ✅   |             |             |
| Market Creation              |           |                 | ✅   |             |             |
| Pool Management              |           |                 | ✅   |             |             |
| Creation/Trading/Listing fees|           |                 |     |               |            |

## Contracts

### Sonic SVM Mainnet

// To be updated

### Sonic SVM Testnet

- 8mYkiWJF23Znv77Erfck3cQvUPqttdGPxu2FmBAsfBS

## Bonding Curve Math
```
(x + x0) * (y + y0) = k
x: quote reserve
x0: quote virtual reserve
y: base reserve
y0: base virtual reserve
```
## Main Components

### States

#### `LiquidityProvider`
**Purpose:**  
Represents a **liquidity provider's** share in a **bonding curve liquidity pool**.

##### Fields:
- **`shares: u64`** –  
  - The number of **shares** this provider **owns** in the liquidity pool.  
  - Determines the provider’s **proportional claim** on the pool’s assets.  

##### Key Considerations:
- **Shares represent ownership**: More shares = **higher claim** on liquidity pool reserves.
- **Used in bonding curve mechanics**: Share balances **change dynamically** based on deposits and withdrawals.
- **Not yet added to the contract**: The logic for handling shares **needs to be implemented**.

This structure helps track **liquidity providers' stakes** in the **automated market maker (AMM)**! 🔄📈  

#### `LiquidityPool`
**Purpose:**  
Represents a **bonding curve liquidity pool**, tracking reserves and ownership.

##### Fields:
- **`creator: Pubkey`** –  
  - **Public key** of the liquidity pool **creator**.  
  - The **owner** or **initial deployer** of the pool.  

- **`token: Pubkey`** –  
  - **Public key** of the token **being traded** in the liquidity pool.  
  - Defines the **asset type** managed by the pool.  

- **`total_supply: u64`** –  
  - **Total supply** of **liquidity tokens** issued by the pool.  
  - Represents **ownership shares** distributed to liquidity providers.  

- **`reserve_token: u64`** –  
  - The **amount of tokens** held in the pool’s reserves.  
  - Used for **swaps** and liquidity management.  

- **`reserve_sol: u64`** –  
  - The **amount of SOL** held in the pool’s reserves.  
  - Supports **SOL-token pair** trading.  

- **`bump: u8`** –  
  - **Nonce** for the **program-derived address (PDA)**.  
  - Ensures **security** and **uniqueness** of the pool’s account.  

##### Key Considerations:
- **Manages liquidity reserves**: Ensures smooth **token-SOL swaps**.  
- **Uses bonding curve pricing**: Price changes dynamically based on reserves.  
- **Tracks ownership**: Liquidity providers receive **shares** in exchange for deposits.  

This structure forms the **core state** of the **bonding curve AMM**! ⚖️📊  

#### `LiquidityPoolAccount` Trait  
**Purpose:**  
Defines core **liquidity pool operations** for a **bonding curve AMM**.  

---

#### Methods:

1. 🔄 **update_reserves**  
**Updates** the **token and SOL reserves** of the liquidity pool.  
```rust
fn update_reserves(&mut self, reserve_token: u64, reserve_sol: u64) -> Result<()>;
```
Adjusts reserve balances after swaps or liquidity changes.

2. ➕ **add_liquidity**
Deposits both token and SOL into the pool to receive shares.
```rust
fn add_liquidity(
    &mut self,
    token_accounts: (
        &mut Account<'info, Mint>,
        &mut Account<'info, TokenAccount>,
        &mut Account<'info, TokenAccount>,
    ),
    pool_sol_vault: &mut AccountInfo<'info>,
    authority: &Signer<'info>,
    token_program: &Program<'info, Token>,
    system_program: &Program<'info, System>,
) -> Result<()>;
```
Mints liquidity tokens based on deposited amounts.
Ensures proportional deposits of token and SOL.
3. ➖ **remove_liquidity**
Burns liquidity pool shares to redeem underlying assets.

```rust
fn remove_liquidity(
    &mut self,
    token_accounts: (
        &mut Account<'info, Mint>,
        &mut Account<'info, TokenAccount>,
        &mut Account<'info, TokenAccount>,
    ),
    pool_sol_account: &mut AccountInfo<'info>,
    authority: &Signer<'info>,
    bump: u8,
    token_program: &Program<'info, Token>,
    system_program: &Program<'info, System>,
) -> Result<()>;
```
Returns SOL and tokens in proportion to the shares burned.
Ensures equitable withdrawals from the pool.
4. 🛒 **buy**
Purchases tokens from the pool using SOL, following the bonding curve.

```rust
fn buy(
    &mut self,
    token_accounts: (
        &mut Account<'info, Mint>,
        &mut Account<'info, TokenAccount>,
        &mut Account<'info, TokenAccount>,
    ),
    pool_sol_vault: &mut AccountInfo<'info>,
    amount: u64,
    authority: &Signer<'info>,
    token_program: &Program<'info, Token>,
    system_program: &Program<'info, System>,
) -> Result<()>;
```
Uses pool reserves to determine pricing.
Updates reserves after purchase.
5. 💰 **sell**
Sells tokens to the pool in exchange for SOL, using the bonding curve.

```rust
fn sell(
    &mut self,
    token_accounts: (
        &mut Account<'info, Mint>,
        &mut Account<'info, TokenAccount>,
        &mut Account<'info, TokenAccount>,
    ),
    pool_sol_vault: &mut AccountInfo<'info>,
    amount: u64,
    bump: u8,
    authority: &Signer<'info>,
    token_program: &Program<'info, Token>,
    system_program: &Program<'info, System>,
) -> Result<()>;
```
Adjusts pool reserves based on trade size.
Ensures fair price determination using the bonding curve.

6. 🔄 **transfer_token_from_pool**
Transfers tokens out of the liquidity pool.

```rust
fn transfer_token_from_pool(
    &self,
    from: &Account<'info, TokenAccount>,
    to: &Account<'info, TokenAccount>,
    amount: u64,
    token_program: &Program<'info, Token>,
) -> Result<()>;
```

7. 🔄 **transfer_token_to_pool**
Transfers tokens into the liquidity pool.

```rust
fn transfer_token_to_pool(
    &self,
    from: &Account<'info, TokenAccount>,
    to: &Account<'info, TokenAccount>,
    amount: u64,
    authority: &Signer<'info>,
    token_program: &Program<'info, Token>,
) -> Result<()>;
```

8. 🔄 **transfer_sol_to_pool**
Transfers SOL into the liquidity pool.

```rust
fn transfer_sol_to_pool(
    &self,
    from: &Signer<'info>,
    to: &mut AccountInfo<'info>,
    amount: u64,
    system_program: &Program<'info, System>,
) -> Result<()>;
```
9. 🔄 **transfer_sol_from_pool**
Transfers SOL out of the liquidity pool.

```rust
fn transfer_sol_from_pool(
    &self,
    from: &mut AccountInfo<'info>,
    to: &Signer<'info>,
    amount: u64,
    bump: u8,
    system_program: &Program<'info, System>,
) -> Result<()>;
```

#### Key Features
✅ Liquidity Management – Allows users to add/remove liquidity seamlessly.
✅ Bonding Curve Pricing – Ensures dynamic, algorithmic pricing.
✅ Efficient Asset Swaps – Supports SOL-token trading.
✅ Permissioned Operations – Uses Solana PDAs for security.

This trait powers the bonding curve AMM, enabling automated liquidity provisioning and trustless swaps! ⚡📈

