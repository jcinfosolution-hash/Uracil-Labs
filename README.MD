# 📘 Uracil Chain

Uracil Chain is a production-grade dual-chain blockchain with instant transactions and secure archive consensus. It transforms compute into a tradable resource with built-in security, governance, and adversarial validation.

---

![Uracil Chain Architecture](Screenshots/Full%20Architecture.png)

## 🚀 Download & Run (Windows)

1. Download `uracil-chain.exe` from [Releases](https://github.com/jcinfosolution-hash/Uracil-Labs/releases)
2. Open Command Prompt in the download folder
3. Run:

```bash
uracil-chain.exe 9001
```

> **Note**: Linux and Mac builds will be added soon.

---

## 🎮 Try in 30 Seconds

Once your node is running, try these commands:

```bash
accounts
balance user1 core
send user1 user2 10 core
mine
```

---

## 📘 Uracil Chain v1

### What This Is

Uracil Chain is a complete blockchain implementation designed for:

- Deterministic replay and verifiable state integrity
- Serial-based asset ownership with full forensic tracking
- Adversarially hardened consensus across PoW, PoS, and PoAuth
- Quantum resistance using Falcon signatures
- Dual-factor security with independent Guardian Keys

This version demonstrates a fully working node with all core systems operational and adversarial testing completed across consensus, network, and security layers.

---

### Basic Operations

```bash
# Create accounts (they start with 0 balance)
create alice
create bob
mine

# Check balances (both have 0)
balance alice core
balance bob core

# Fund accounts from user1 (genesis account with tokens)
send user1 alice 100 core
send user1 bob 50 core
mine

# Now check balances
balance alice core   # Should show 100
balance bob core     # Should show 50

# Now alice can send to bob
send alice bob 10 core
mine
balance alice core   # Should show 90
balance bob core     # Should show 60
```

---

### Validator Registration

```bash
# Register alice as validator (needs 100 URStake)
register alice 100
validators
next_validator
generate_vrf alice
```

---

### Guardian Keys (v2 — Dual-Factor Security)

Guardian keys provide true dual-factor security with an independent seed.

**First Receive (Seed Generation):**

```bash
send user1 alice 100 core
mine
```

After mining, the system displays:

```
🔐 GUARDIAN SEED GENERATED FOR alice
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Guardian Seed: a1b2c3d4e5f67890...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS SEED SECURELY. You will need it to:
   - Sign transactions from this account
   - Recover your wallet
This seed is NOT stored on the blockchain.
If you lose it, you CANNOT access your funds.
```

**Sending Tokens (Seed Required):**

```bash
send alice bob 10 core
```

```
🔐 GUARDIAN SEED REQUIRED
Enter your guardian seed (the one you saved): _
[enter saved seed]
✅ Guardian seed verified.

✅ Transaction processed!
```

**Check Guardian Status:**

```bash
guardian_status alice
```

```
Address: alice
Guardian Version: 2
Guardian System: ACTIVE (v2)
  - Seed generated on first receive
  - Auto-rotates every transaction
  - Seed stored locally (not on chain)
  - Local wallet: ✅ Present
  - Evolution count: 1
```

---

### Marketplace - Buy/Sell Compute

```bash
# View available compute plans
compute_plans

# Buy compute power (spends URCore) — prompts for guardian seed if enabled
buy_compute 1000 institution1

# Sell idle compute (earns URCore) — prompts for guardian seed if enabled
dr_sell 100 3600 0.001 alice
list_orders

# Buy from order (spends URCore) — prompts for guardian seed if enabled
buy_order sell_alice_<order_id> 50 bob
mine
```

---

### Mining

```bash
# Start mining with purchased compute power — prompts for guardian seed if enabled
start_mining 100000 bob
mine
stop_mining bob
balance bob power  # Check mining rewards
```

---

### Performance Testing - Large Transfers

```bash
# Test transfer performance with timing metrics
time_transfer institution1 user1 10000 core
```

Provides microsecond-level timing for large transfers:
- `build_ranges`: time to construct serial ranges
- `get_nonce`: time to fetch account nonce
- `create_tx`: time to create transaction
- `sign_time`: Falcon signature time (36–44ms)
- `processing_time`: transaction broadcast and validation
- `total_time`: end-to-end time

> **Note**: For transfers above 10 billion serials (100 tokens), the archive rejects due to MAX_MINT_PER_TX limit. Test with 10 tokens (1 billion serials) for a full success:

```bash
time_transfer institution1 user1 10 core
```

---

### PoAuth - Institutional Quorum

```bash
# Generate auth key for validator
auth_init alice

# Register auth validator (use the generated auth address)
auth_register <auth_address> 3 "Alice Institute"

# Create proposal using a block hash
archive chain  # Get a block hash
auth_propose <block_hash>

# Sign proposal
auth_sign <proposal_id> <auth_address>

# Check status
auth_status
auth_proposals
clean_proposals
```

---

### Protein Folding (ZK Proof)

```bash
fold_protein AAA alice
mine
balance alice work
```

---

### Silent Guardian & Serial Tracking

```bash
# Check silent guardian status
sg status
sg test

# List serials owned by an account
list_serials alice core
list_serials alice all
```

---

## 🌐 3-Node Network Test

**Terminal 1 (Node 1 - Port 9001)**
```bash
cargo run -- 9001
```

**Terminal 2 (Node 2 - Port 9002)**
```bash
cargo run -- 9002
connect 127.0.0.1:9001
```

**Terminal 3 (Node 3 - Port 9003)**
```bash
cargo run -- 9003
connect 127.0.0.1:9001
```

**Test Network Sync**

On Node 1:
```bash
create alice
mine
send user1 alice 100 core
mine
```

On Node 2 & 3:
```bash
archive chain          # Should show same blocks
balance alice core     # Should show 100
```

---

### Debug & Forensic Commands

```bash
# Show hidden debug commands
help --dev

# Debug commands
ghost_status alice           # Show all serials owned
ghost_history <serial>       # Show serial history
check_serial <serial> <owner> # Verify ownership
state_hash                   # Full consensus state hash
```

---

### Available Tokens

| Token | Purpose |
|-------|---------|
| UR-CORE | Main currency |
| UR-STAKE | Staking for validators |
| UR-WORK | Productive work rewards |
| UR-POWER | Mining rewards |
| UR-AUTH | PoAuth weight |

---

## 🧪 Automated Tests

**Layer 1 - Deterministic Replay**
```bash
cargo test --test layer1_deterministic_replay -- --nocapture --test-threads=1
```

**Layer 2 - Supply Invariants**
```bash
cargo test --test layer2_supply_invariants -- --nocapture --test-threads=1
```

**Layer 3 - Guardian Adversarial**
```bash
cargo test --test layer3_guardian_adversarial -- --nocapture --test-threads=1
```

**Layer 4 - Dual Reward Economics**
```bash
cargo test --test layer4_dual_reward_economics -- --nocapture --test-threads=1
```

**Layer 5 - Live Chain Tests**
```bash
cargo test --test layer5_live_chain_test_live_block_reorg -- --nocapture --test-threads=1
cargo test --test layer5_live_chain_validation_consistency_scenarios -- --nocapture --test-threads=1
cargo test --test layer5_live_chain_validation_fork_resolution -- --nocapture --test-threads=1
cargo test --test layer5_live_chain_validation_recovery_scenarios -- --nocapture --test-threads=1
cargo test --test layer5_mempool_consistency -- --nocapture --test-threads=1
cargo test --test layer5_random_flood_with_crash -- --nocapture --test-threads=1
cargo test --test layer5_test_live_overlay_sync_after_reorg -- --nocapture --test-threads=1
cargo test --test layer5_test_simulated_crash_during_ghost_write -- --nocapture --test-threads=1
```

**Layer 6 - Ghost Chain Tests**
```bash
cargo test --test layer6_ghost_as_source_of_truth -- --nocapture --test-threads=1
cargo test --test layer6_ghost_invariants_after_fork -- --nocapture --test-threads=1
```

**Layer 7 - Economic Attacks**
```bash
cargo test --test layer7_economic_attacks -- --nocapture --test-threads=1
```

**Layer 8 - Network Attacks**
```bash
cargo test --test layer8_network_attacks -- --nocapture --test-threads=1
```

**Layer 9 - Adversarial Consensus & Difficulty**
```bash
cargo test --test layer9_adversarial_consensus -- --nocapture --test-threads=1
```

**Layer 10 - PoAuth Attacks**
```bash
cargo test --test layer10_auth_attacks -- --nocapture --test-threads=1
```

**Layer 11 - ZK Proof**
```bash
cargo test --test layer11_zk_protein_consensus -- --nocapture --test-threads=1
```

**Performance Benchmark Test**
```bash
cargo test --test performance_benchmark -- --nocapture
```

---

## 🏗️ Architecture

- **Live Chain** - Instant transaction processing with optimistic overlay
- **Archive Chain** - Secure consensus with PoW/PoS
- **Ghost Chain** - Range-based serial tracking (no expansion, no freeze)
- **PoAuth** - Institutional multi-sig quorum with Falcon signatures
- **Guardian Keys (v2)** - Dual-factor security with independent seed, never stored on chain

---

## 📋 Requirements

- Rust 1.70+
- 2+ CPU cores for parallel verification

---

## 📄 License

MIT
