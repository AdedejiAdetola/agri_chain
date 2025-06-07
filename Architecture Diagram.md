# 🐔 AgriChain Protocol — Fiat-Crypto Settlement Vault

---

# 🧱 Architecture Diagram

````mermaid
flowchart TD
    A[User Wallet<br/>(Naira / Fiat Payment)] --> B[On-Ramp Provider<br/>(Naira → USDC on Solana)]
    B --> C[ChickenVault Smart Contract (Solana)<br/>• Holds USDC<br/>• Maps Order ID to Farmer<br/>• Emits audit events]
    C --> D1[Delivery Oracle System<br/>(Confirms Delivery)]
    C --> D2[Off-Ramp Provider<br/>(USDC → Naira to Farmer Bank)]
    D1 --> E[Trigger USDC Release<br/>to Farmer’s Solana Wallet]


---

## 📜 Protocol POC Requirements for ChickChain

### 🔐 Admin + Configuration

- The protocol shall allow an **admin wallet** to manage global parameters including timeout, off-ramp providers, and delivery oracles.
- The protocol shall allow the admin to **pause/resume** vault operations.

### 🛒 Order Creation & Vaulting

- The protocol shall allow users to initiate orders by paying with Naira through an on-ramp service.
- The protocol shall allow the system to **receive USDC** from on-ramp and map it to an order.
- The protocol shall **vault USDC** and await delivery confirmation.
- On-chain event logs shall be emitted when vaulting occurs.

### 🚚 Delivery & Oracle Confirmation

- The protocol shall allow a **designated oracle address** to confirm delivery.
- Confirmation may be triggered manually or programmatically by trusted sources.

### 💸 Fund Release & Settlement

- Upon delivery confirmation, the protocol shall **release USDC** to the mapped farmer wallet.
- The protocol shall emit **release events** for transparency.
- The protocol shall allow **integration with off-ramp APIs** for converting USDC → Naira to farmer’s bank account.

### 🔄 Refund / Reversal Mechanism

- If delivery is unconfirmed within a timeout window, users may trigger a **refund flow**.
- USDC is returned to the buyer’s wallet if no valid confirmation occurs.

### 📑 Traceability & Auditing

- A hash of the order metadata (e.g., product type, farm ID, timestamp) shall be stored on-chain.
- Status queries and transaction history shall be publicly accessible.

### 💱 Token Conversion & Pricing

- The protocol shall interface with **Solana price oracles (e.g., Pyth)** to compute Naira ↔ USDC rates.
- FX updates shall be dynamic and reflected in real-time conversions.

---

## ⚙️ Smart Contract Features (Instruction Set)

### 🛠️ Core Functions

```rust
createOrder(order_id, farmer_wallet, amount_usdc)
````

- Maps USDC vault to order ID and target wallet

```rust
confirmDelivery(order_id)
```

- Oracle updates order status → triggers USDC release

```rust
releaseFunds(order_id)
```

- Transfers escrowed USDC to farmer upon delivery

```rust
refundOrder(order_id)
```

- Refunds buyer after timeout if delivery unconfirmed

```rust
getOrderStatus(order_id)
```

- Returns vault status, timestamp, and release data

### 🔧 Admin Functions

```rust
setOracleAddress(address)
```

- Sets the address allowed to confirm delivery

```rust
setOffRampProvider(provider_id, is_enabled)
```

- Manages whitelist of active fiat settlement providers

```rust
setTimeoutPeriod(seconds)
```

- Sets global timeout window for refund eligibility

---

## 🔌 External Modules

| Module               | Role                                           |
| -------------------- | ---------------------------------------------- |
| **On-Ramp Gateway**  | Convert fiat → USDC and fund smart contract    |
| **Delivery Oracle**  | GPS or QR code-triggered delivery confirmation |
| **Off-Ramp Gateway** | Convert USDC → Naira and send to bank          |
| **Admin Dashboard**  | Web UI for vault monitoring, refunds, status   |

---

## 📦 Example Order Flow

1. Buyer initiates chicken order worth ₦6,000
2. On-ramp converts to \~\$4 USDC and deposits into smart contract
3. Smart contract logs vault and waits for delivery confirmation
4. Rider delivers chicken → GPS event or manual input triggers oracle
5. Smart contract releases USDC to farmer
6. Off-ramp API sends ₦6,000 to farmer’s bank

---

## 🛡️ Security Considerations

- Vault operations must be non-custodial and protected against reentrancy
- Only verified oracles should trigger delivery confirmation
- Refund logic must include time checks and dispute channels
- All token transfers and logs should be tamper-proof and auditable
