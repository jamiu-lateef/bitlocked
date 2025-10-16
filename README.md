# **BitLocked Protocol**

### **Transform Idle Bitcoin into Productive Capital**

*Trustless, Bitcoin-backed collateralization via Stacks Layer 2 smart contracts.*

---

## **Overview**

BitLocked is a **Bitcoin-collateralized lending protocol** built on the **Stacks blockchain**, enabling users to **unlock liquidity without selling their BTC**. By leveraging Bitcoin’s security through Stacks’ settlement guarantees, BitLocked merges **Bitcoin’s sound money properties** with **DeFi utility**, enabling stablecoin loans, automated liquidation, and decentralized risk management.

Users can deposit BTC as collateral, mint stablecoins or borrow assets, and maintain exposure to BTC price movements—all while ensuring that every critical state transition settles on Bitcoin’s base layer.

---

## **Key Features**

* ⚡ **Bitcoin-Collateralized Loans** — Lock BTC as collateral to mint stablecoin loans.
* 🧱 **Bitcoin Settlement Layer Security** — All contract state transitions ultimately anchor to Bitcoin.
* 📈 **Dynamic Risk Management** — Algorithmic collateral ratio enforcement and liquidation thresholds.
* 🔒 **Trustless Collateral Custody** — No intermediaries; fully on-chain enforcement via Clarity contracts.
* 🪙 **Price Oracles** — Decentralized feed for BTC and STX asset valuations.
* 🧮 **Transparent Accounting** — Real-time access to system stats and loan registry.

---

## **System Overview**

At a high level, the BitLocked Protocol consists of **three functional layers**:

1. **Core Smart Contracts (Clarity)**

   * Define loan logic, collateral ratios, liquidation mechanics, and administrative governance.
   * All operations are deterministic and verifiable on-chain.

2. **Oracle Layer**

   * Provides asset price data (BTC/STX).
   * Can be integrated with decentralized oracle networks or verified multisig feeds.

3. **Frontend / Middleware Interface (Optional)**

   * Web or API layer for user-friendly loan management.
   * Communicates with Stacks nodes and invokes Clarity functions securely.

---

## **Contract Architecture**

### **1. Core Constants & Configuration**

* **`CONTRACT-OWNER`** — Administrator of the protocol configuration.
* **Collateral Parameters:**

  * `minimum-collateral-ratio` (default: 150%)
  * `liquidation-threshold` (default: 120%)
  * `platform-fee-rate` (default: 1%)

These constants govern the safety and solvency of the system.

---

### **2. State Variables**

| Variable               | Description                                    |
| ---------------------- | ---------------------------------------------- |
| `platform-initialized` | Indicates if the protocol is active.           |
| `total-btc-locked`     | Tracks aggregate BTC collateral in the system. |
| `total-loans-issued`   | Counter for all loans ever created.            |

---

### **3. Data Maps**

| Map                 | Key       | Value                   | Purpose                                      |
| ------------------- | --------- | ----------------------- | -------------------------------------------- |
| `loans`             | `loan-id` | Loan record             | Registry of all active and historical loans. |
| `user-loans`        | `user`    | List of active loan IDs | Index for user-specific loan access.         |
| `collateral-prices` | `asset`   | Price                   | Oracle feed for collateral valuation.        |

---

### **4. Core Public Functions**

#### **Administrative**

* `initialize-platform()` — One-time setup to activate the protocol.
* `update-collateral-ratio(new-ratio)` — Adjust system-level collateral requirements.
* `update-liquidation-threshold(new-threshold)` — Modify liquidation sensitivity.
* `update-price-feed(asset, new-price)` — Update on-chain price oracle data.

#### **User Operations**

* `deposit-collateral(amount)` — Lock BTC into the protocol.
* `request-loan(collateral, loan-amount)` — Create a new collateralized debt position (CDP).
* `repay-loan(loan-id, amount)` — Repay loan principal and accrued interest.

#### **Read-Only Views**

* `get-loan-details(loan-id)` — Fetch full loan data structure.
* `get-user-loans(user)` — Retrieve user’s active loan list.
* `get-platform-stats()` — Inspect global metrics.
* `get-valid-assets()` — Supported asset identifiers.

---

## **Risk Management & Liquidation Logic**

BitLocked continuously enforces **collateral adequacy** using oracle-driven price feeds.

* **Collateral Ratio Calculation:**

  ```
  ratio = (collateral_value / loan_amount) * 100
  ```

* **Automatic Liquidation Trigger:**
  When a position’s ratio ≤ `liquidation-threshold`, the loan is marked “liquidated,” and collateral is released back into protocol reserves.

The liquidation function (`check-liquidation`) ensures solvency and protects lenders from under-collateralized positions.

---

## **Security Considerations**

1. **Deterministic Logic:**

   * All core operations are deterministic and auditable on-chain.
   * No hidden randomness or mutable external dependencies.

2. **Oracle Integrity:**

   * Only the `CONTRACT-OWNER` (or future DAO governance) can update price feeds.
   * Future iterations may integrate decentralized oracles (e.g., Chainlink, Hiro’s SIP-010 oracles).

3. **Access Control:**

   * Strict enforcement via `(asserts! (is-eq tx-sender CONTRACT-OWNER) ...)`.
   * Prevents unauthorized configuration or asset manipulation.

4. **Fail-Safe Design:**

   * Loan state transitions (`active → repaid → liquidated`) are irreversible and validated.
   * Protocol rejects invalid or inconsistent state transitions.

---

## **Data Flow Overview**

```
 ┌──────────────────────────────┐
 │        User Wallet           │
 │  (Stacks Address + BTC Key)  │
 └──────────────┬───────────────┘
                │
                ▼
       deposit-collateral(amount)
                │
                ▼
 ┌──────────────────────────────┐
 │     BitLocked Smart Contract │
 │ ────────────┬─────────────── │
 │  Validate inputs             │
 │  Record loan / collateral    │
 │  Fetch BTC price (oracle)    │
 │  Compute ratios              │
 │  Issue loan token            │
 └──────────────┬───────────────┘
                │
                ▼
         User Receives Loan
                │
                ▼
        repay-loan() or liquidation
```

---

## **Deployment & Configuration**

1. **Compile and Deploy:**

   ```bash
   clarity-cli launch bitlocked.clar --network mainnet
   ```

2. **Initialize Protocol:**

   ```clarity
   (contract-call? .bitlocked initialize-platform)
   ```

3. **Set Oracle Prices:**

   ```clarity
   (contract-call? .bitlocked update-price-feed "BTC" u6800000)
   ```

4. **Start Operations:**

   * Deposit BTC collateral.
   * Request loans based on configured collateral ratios.

---

## **Future Extensions**

* DAO governance for parameter updates.
* Integration with decentralized BTC oracles.
* Cross-chain collateralization via sBTC.
* On-chain liquidation auctions.
