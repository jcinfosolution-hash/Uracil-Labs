# Idle Compute Marketplace

## Overview

The Idle Compute Marketplace enables participants in Uracil Chain to **trade computational power using UR-CORE tokens**.

* **Sellers** monetize unused compute capacity
* **Buyers** acquire compute power for mining or other workloads

This creates a decentralized, on-demand compute economy integrated directly into the blockchain.

---

## System Flow

```
Seller (Compute Provider)              Buyer (Compute Consumer)
        │                                      │
        ▼                                      ▼
   dr_sell command                        buy_order command
        │                                      │
        └───────────────┬──────────────────────┘
                        ▼
                  Sell Order Created
                        │
                        ▼
        UR-CORE transferred from buyer to seller
                        │
                        ▼
             Compute allocated to buyer
```

---

## Compute Plans (Direct Purchase)

Users can purchase compute directly without interacting with marketplace orders.

### Command

```bash
compute_plans
```

### Example Output

```
AVAILABLE COMPUTE PLANS:
• Basic:      1000 units  → 1000 UR-CORE
• Standard:   5000 units  → 4800 UR-CORE (5% discount)
• Premium:    10000 units → 9000 UR-CORE (10% discount)
• Enterprise: 50000 units → 40000 UR-CORE (20% discount)
```

These plans are optional. Users may also purchase arbitrary amounts using `buy_compute`.

---

## Core Commands

### 1. Buy Compute (Direct Purchase)

Purchase compute power directly using UR-CORE.

```bash
buy_compute <amount> <institution_address>
```

#### Example

```bash
buy_compute 1000 institution1
```

#### Process

* Deducts UR-CORE from buyer
* Adds compute units to buyer’s account
* Enables mining using purchased compute

---

### 2. Sell Compute (Create Order)

Create a marketplace listing for idle compute.

```bash
dr_sell <amount> <duration_seconds> <price_per_unit> <seller_address>
```

#### Parameters

| Parameter        | Description            | Example |
| ---------------- | ---------------------- | ------- |
| amount           | Compute units for sale | 100     |
| duration_seconds | Order lifetime         | 3600    |
| price_per_unit   | Price in UR-CORE       | 0.001   |
| seller_address   | Owner of compute       | alice   |

#### Example

```bash
dr_sell 100 3600 0.001 alice
```

#### Behavior

* Creates a sell order
* Order remains active until filled or expired
* Seller retains compute ownership until purchased

---

### 3. List Active Orders

```bash
list_orders
```

Displays all active marketplace orders including:

* Order ID
* Seller
* Remaining compute
* Price
* Expiry time

---

### 4. Buy from Order

Purchase compute from an existing sell order.

```bash
buy_order <order_id> <quantity> <buyer_address>
```

#### Example

```bash
buy_order sell_alice_1774411304 50 bob
```

#### Process

* Validates order availability
* Transfers UR-CORE from buyer to seller
* Allocates compute units to buyer
* Updates remaining quantity

---

## Order Data Structure

```rust
pub struct SellOrder {
    pub id: String,
    pub seller: String,
    pub amount: u64,
    pub remaining: u64,
    pub duration: u64,
    pub price: u64,
    pub created_at: i64,
    pub expires_at: i64,
    pub active: bool,
    pub token_type: TokenType,
}
```

---

## Order Lifecycle

### 1. Creation

* Seller submits `dr_sell`
* Order initialized with full amount
* Status: Active

### 2. Active Trading

* Buyers can purchase partially or fully
* Remaining amount updates dynamically

### 3. Partial Fill

* Only part of compute is purchased
* Order remains active

### 4. Full Fill

* Remaining becomes zero
* Order is deactivated

### 5. Expiry

* Order expires after duration
* Automatically removed from active listings

---

## Fee Model

### Buyer Fees

```
total_cost = quantity × price
fee = base_fee + value_fee
```

* Fees are paid in **UR-CORE**
* Deducted from buyer account

### Seller Receives

```
seller_receives = total_cost
```

* Seller receives full payment (fees handled separately)

---

## Integration with Mining

Purchased compute can be immediately used for mining.

```bash
start_mining <compute_units> <address>
mine
stop_mining <address>
balance <address> power
```

### Compute Tracking

```rust
pub struct Account {
    pub compute_balance: u64,
}
```

Compute is separate from token balances and only used for mining.

---

## Automatic Cleanup

Expired orders are removed automatically during block processing.

```rust
pub fn clean_expired_orders(&mut self, current_time: i64) -> usize
```

No manual intervention is required.

---

## Security Model

### Anti-Fraud

* Orders validated against seller capacity
* Cannot oversell compute
* Partial fills strictly enforced

### Double-Spend Protection

* Uses range-based transfers via Ghost Chain
* Atomic state updates prevent inconsistencies

### Anti-Spam

* Order creation requires transaction fees
* High-frequency spam becomes costly

### Market Integrity

* No centralized pricing
* Buyers choose which orders to fill
* Fully market-driven

---

## Example Workflow

### Seller (Alice)

```bash
dr_sell 100 3600 0.001 alice
list_orders
```

---

### Buyer (Bob)

```bash
buy_order sell_alice_1774411304 50 bob
```

---

### After Purchase

```bash
list_orders
# Remaining: 50
```

---

### Bob Uses Compute

```bash
start_mining 50 bob
mine
stop_mining bob
balance bob power
```

---

## Summary

| Command       | Purpose                          |
| ------------- | -------------------------------- |
| compute_plans | View predefined compute packages |
| buy_compute   | Purchase compute directly        |
| dr_sell       | Create marketplace sell order    |
| list_orders   | View active listings             |
| buy_order     | Purchase compute from seller     |
| start_mining  | Use compute for mining           |

---

## Key Insight

The Idle Compute Marketplace transforms unused computational resources into a **liquid, tradable asset**, enabling:

* Efficient resource utilization
* Decentralized compute access
* Seamless integration with mining and rewards

It bridges infrastructure and economics—turning raw compute into a native on-chain economy.