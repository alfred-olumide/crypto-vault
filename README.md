# 📦 CryptoVault Pro — Advanced Collateralized Lending Engine

**Version:** `v1.0.0`
**Language:** [Clarity](https://docs.stacks.co/docs/write-smart-contracts/clarity-overview)
**Network:** Stacks Blockchain

---

## 📘 Overview

**CryptoVault Pro** is a next-generation decentralized finance (DeFi) protocol designed for capital-efficient, Bitcoin-backed lending. Built with institutional standards in mind, the protocol enables secure, automated, and transparent asset-collateralized borrowing while integrating dynamic risk management and portfolio protection mechanisms.

The platform leverages **BTC as primary collateral**, incorporating real-time monitoring, dynamic collateral ratios, multi-oracle price feeds, and liquidation protection to ensure systemic resilience and user confidence.

---

## 🚀 Key Features

* **Dynamic Collateral Management:** Adjusts collateral requirements based on market volatility and user risk profiles.
* **Predictive Liquidation Engine:** Monitors loan health and automatically initiates liquidation to minimize risk.
* **Multi-Oracle Price Aggregation:** Prevents manipulation by sourcing data from multiple trusted feeds.
* **Flexible Interest Rate Model:** Supports adaptive interest calculations based on protocol parameters.
* **Cross-Chain Asset Support (extensible):** Initially supports `BTC` and `STX`, with room for future asset types.
* **Institutional-Grade Security:** Immutable smart contract logic with role-based access control and strict validations.

---

## 🧠 System Overview

| Component         | Responsibility                                                                       |
| ----------------- | ------------------------------------------------------------------------------------ |
| **Loan Engine**   | Handles loan origination, repayment, interest accrual, and collateral validation     |
| **Risk Module**   | Monitors real-time collateral ratios and performs liquidation if thresholds breached |
| **Oracle Module** | Maintains price feeds for BTC (and future assets) via admin-set trusted inputs       |
| **Admin Control** | Enables platform configuration such as thresholds, fees, and price updates           |
| **User Vaults**   | Manages per-user active loans and tracks status and balances                         |

---

## 📚 Contract Architecture

### Contract Name: `CryptoVaultPro`

#### 🏗️ Platform Constants

```clarity
CONTRACT-OWNER                 ;; Admin authority for sensitive updates
VALID-ASSETS                  ;; ["BTC", "STX"]
```

#### 🧩 Core Variables

| Variable                   | Type   | Description                            |
| -------------------------- | ------ | -------------------------------------- |
| `platform-initialized`     | `bool` | Flag to prevent re-initialization      |
| `minimum-collateral-ratio` | `uint` | e.g., 150% required collateral ratio   |
| `liquidation-threshold`    | `uint` | e.g., 120% triggers liquidation        |
| `platform-fee-rate`        | `uint` | Reserved for platform fees (e.g., 1%)  |
| `total-btc-locked`         | `uint` | Tracks BTC collateral held by protocol |
| `total-loans-issued`       | `uint` | Global loan counter                    |

#### 🧾 Data Structures

```clarity
(loans { loan-id }) => {
  borrower,
  collateral-amount,
  loan-amount,
  interest-rate,
  start-height,
  last-interest-calc,
  status,
}

(user-loans { user }) => {
  active-loans: (list 10 uint)
}

(collateral-prices { asset }) => {
  price: uint
}
```

---

## 🔄 Core Data Flow

### 🏦 1. Deposit Collateral

Users lock BTC collateral via `deposit-collateral(amount)`. Collateral is tracked globally and associated with borrower at loan creation.

### 📝 2. Request Loan

When calling `request-loan(collateral, loan-amount)`:

* Current BTC price is fetched via oracle
* Required collateral is computed: `loan-amount * minimum-collateral-ratio`
* Loan struct is created with unique `loan-id` and stored in `loans`
* Loan ID is added to the borrower’s `user-loans`

### 💥 3. Automatic Liquidation

A private function `check-liquidation(loan-id)` monitors active loans:

* Compares real-time collateral ratio to `liquidation-threshold`
* If undercollateralized, triggers `liquidate-position`
* Marks loan as `"liquidated"` and removes it from user mapping

### 💰 4. Repayment

Users repay full debt via `repay-loan(loan-id, amount)`:

* Interest is calculated based on blocks passed since last update
* Total due is checked against `amount`
* On success, collateral is released and loan status updated to `"repaid"`

---

## ⚙️ Admin & Governance Functions

| Function                                      | Description                             |
| --------------------------------------------- | --------------------------------------- |
| `initialize-platform()`                       | Initializes the contract (one-time)     |
| `update-collateral-ratio(new-ratio)`          | Updates minimum required collateral     |
| `update-liquidation-threshold(new-threshold)` | Sets liquidation trigger ratio          |
| `update-price-feed(asset, new-price)`         | Updates asset price from trusted oracle |

---

## 🔍 Read-Only Interfaces

| Function                    | Output                              |
| --------------------------- | ----------------------------------- |
| `get-loan-details(loan-id)` | Full loan metadata                  |
| `get-user-loans(user)`      | Active loan IDs for a user          |
| `get-platform-stats()`      | Protocol-wide metrics               |
| `get-valid-assets()`        | Lists whitelisted collateral assets |

---

## 🔒 Security Considerations

* **Authorization Controls**: All sensitive updates require caller to be `CONTRACT-OWNER`.
* **Input Validations**: Defensive checks across all public entry points prevent invalid or malicious state changes.
* **Immutable Parameters**: Loan terms (rate, collateral, etc.) are immutable once issued.
* **Oracle Trust**: Only the admin can set price feeds—multi-oracle integration recommended at frontend layer.

---

## 📈 Future Improvements

* Support for dynamic interest rates via DAO-driven governance
* Integration with Chainlink/BTC price feeds (via off-chain relay)
* Multi-asset collateralization engine (`USDC`, `wETH`, etc.)
* Platform fee streaming or staking module

---

## 📎 Dependencies

* **Clarity Language SDK**
* **Stacks Blockchain CLI**
* **Stacks Devnet/Testnet/Mainnet Environment**

---

## 🧪 Testing & Deployment

To test or deploy:

```bash
# Run tests
clarinet test

# Deploy on Devnet
clarinet deploy
```

---

## 📄 License

MIT License — use and modify freely, but attribution is appreciated.
