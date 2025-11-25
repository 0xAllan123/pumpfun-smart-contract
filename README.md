## Pump.fun Liquidity & Raydium Pool Program (Solana / Anchor)

This repository contains the on-chain **Pump.fun** smart contract implemented with the **Anchor** framework on **Solana**.  
The program focuses on configurable swap fees, basic AMM-style liquidity management, and **Raydium pool creation via CPI**.

The code is suitable as a reference implementation for:
- **Solana DeFi engineers** integrating with Pump.fun–style liquidity flows.
- **Auditors** reviewing liquidity and swap logic.
- **Contributors** extending the pool logic or Raydium integration.

## 📌 Program Overview

- **Program name**: `pump`
- **Program ID (devnet)**: `7wUQXRQtBzTmyp9kcrmok9FKcc4RSYXxPYN9FGDLnqxb`
- **Framework**: [Anchor](https://www.anchor-lang.com/) on Solana

Core capabilities:
- **Configurable fee curve** via `CurveConfiguration` account.
- **Liquidity provision** to a `LiquidityPool` (add/remove liquidity).
- **Token/SOL swaps** using a constant‑product style formula with fee application.
- **Raydium pool creation** via CPI (`create_raydium_pool` instruction).

## 📋 Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Build & Deploy](#build--deploy)
- [Program Instructions](#program-instructions)
- [Accounts & Architecture](#accounts--architecture)
- [Development & Testing](#development--testing)
- [Support](#support)
- [Contributing](#contributing)
- [License](#license)

## ✅ Requirements

- **Rust** 1.70+
- **Solana CLI** 1.16+ (or compatible with your Anchor toolchain)
- **Anchor** 0.29.0+
- **Node.js** 18+ (for tests / TypeScript clients)
- **Yarn** or **npm**

## 🛠 Installation

```bash
# Clone the repository
git clone https://github.com/0xAllan123/Pumpfun-Smart-Contract.git
cd Pumpfun-Smart-Contract

# Install TypeScript / JS dependencies (for tests & clients)
yarn install        # or: npm install
```

## 🚀 Build & Deploy

The project is configured with Anchor workspaces (`Cargo.toml`) and `Anchor.toml` for devnet deployment.

```bash
# Build the on-chain program
anchor build

# Deploy to devnet (uses provider settings from Anchor.toml)
anchor deploy
```

You can also use `solana config get` and Anchor’s `--provider.cluster` flag to target a different cluster if required.

## 📡 Program Instructions

The `pump` program exposes the following primary instructions (see `programs/pump/src/lib.rs` and `programs/pump/src/instructions`):

- **`initialize`**
  - Initializes the global `CurveConfiguration` account and sets the swap fee (as `f64`).
  - Signature:  
    `pub fn initialize(ctx: Context<InitializeCurveConfiguration>, fee: f64) -> Result<()>`

- **`add_liquidity`**
  - Adds liquidity to a `LiquidityPool`, mints provider shares, and updates reserves.
  - Signature:  
    `pub fn add_liquidity(ctx: Context<AddLiquidity>, amount_one: u64, amount_two: u64) -> Result<()>`

- **`remove_liquidity`**
  - Burns provider shares and returns a proportional amount of underlying assets.
  - Signature:  
    `pub fn remove_liquidity(ctx: Context<RemoveLiquidity>, nonce: u8, init_pc_amount: u64) -> Result<()>`

- **`swap`**
  - Executes a token/SOL swap using the configured fee and AMM formula.
  - Signature:  
    `pub fn swap(ctx: Context<Swap>, amount: u64, style: u64) -> Result<()>`

- **`create_raydium_pool`**
  - Creates a Raydium pool via CPI, using initial token/PC amounts.
  - Signature:  
    `pub fn create_raydium_pool(ctx: Context<CreateRaydiumPool>, nonce: u8, init_pc_amount: u64, init_coin_amount: u64) -> Result<()>`

### Example (TypeScript / Anchor client)

```ts
import { Program } from '@coral-xyz/anchor';
import { Connection, PublicKey } from '@solana/web3.js';

// Example: swap
const connection = new Connection('https://api.devnet.solana.com');
const program = new Program(idl, new PublicKey('7wUQXRQtBzTmyp9kcrmok9FKcc4RSYXxPYN9FGDLnqxb'), { connection });

await program.methods
  .swap(new anchor.BN(amount), new anchor.BN(style))
  .accounts({
    // fill in accounts as defined in the Anchor IDL / instruction context
  })
  .rpc();
```

## 🧱 Accounts & Architecture

Key modules (under `programs/pump/src`):

- **`state.rs`**
  - `CurveConfiguration` – stores global fee configuration.
  - `LiquidityPool` – tracks token mints, total supply, reserves, and bump.
  - `LiquidityProvider` – stores LP share balances per provider.
  - `LiquidityPoolAccount` trait – encapsulates liquidity and swap logic, as well as SOL/token transfer helpers.

- **`instructions/`**
  - `initialize.rs`, `add_liquidity.rs`, `remove_liquidity.rs`, `swap.rs`, and Raydium-related logic for `create_raydium_pool`.

- **`errors.rs`**
  - `CustomError` enum with expressive error messages (e.g., `InsufficientShares`, `InvalidAmount`, `OverflowOrUnderflowOccurred`).

- **`consts.rs`**
  - Core configuration constants such as `INITIAL_PRICE`.

The swap implementation uses a constant‑product style formula with fee application derived from `CurveConfiguration.fees`, combined with SPL Token and system program CPI calls for token and SOL transfers.

## 🧪 Development & Testing

Anchor test configuration is defined in `Anchor.toml` (`[scripts]` and `[test]` sections).

Typical local/dev workflow:

```bash
# Run the Anchor test suite
anchor test

# (Optional) TypeScript tests
yarn test     # or: npm test
```

Tests use `ts-mocha` as configured in `Anchor.toml`:

```toml
[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

## 📞 Support

- **Telegram**: `@oxgoldenledger`  
- **Discord**: `goldenledger`  
- **GitHub Issues**: use the repository’s *Issues* tab for bug reports and feature requests.

## 🤝 Contributing

Contributions, audits, and review feedback are welcome.

- Please read `CONTRIBUTING.md` for guidelines.
- Open an issue to discuss substantial changes before raising a PR.

## 📄 License

This project is licensed under the **MIT License**. See `LICENSE` for details.
