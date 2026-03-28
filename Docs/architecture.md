Uracil Chain Architecture
Overview
Uracil Chain implements a dual-chain architecture that separates:

Low-latency execution (Live Chain)

Secure finality (Archive Chain)

This design allows:

⚡ Instant transaction feedback (<1ms)

🔒 Strong consensus guarantees (PoW/PoS/PoAuth/PoUW)

🔁 Deterministic replay and recovery

🔐 Dual-factor security with auto-rotating keys

System Flow
text
┌─────────────────────────────────────────────────────────────────────────────┐
│                           USER TRANSACTION                                 │
│                      (Falcon-signed, guardian-verified)                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              LIVE CHAIN                                     │
│                          (Instant Execution)                               │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  1. Validation                                                      │   │
│  │     • Falcon signature verification                                 │   │
│  │     • Nonce check                                                   │   │
│  │     • Balance check (via Ghost)                                     │   │
│  │     • Guardian seed verification (if initialized)                   │   │
│  │     • Fee validation                                                │   │
│  └───────────────────────────────┬─────────────────────────────────────┘   │
│                                  ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  2. Optimistic Overlay                                              │   │
│  │     • Apply to in-memory state immediately                          │   │
│  │     • User sees balance update instantly                            │   │
│  │     • Reversible (if transaction fails later)                       │   │
│  └───────────────────────────────┬─────────────────────────────────────┘   │
│                                  ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  3. Mempool                                                         │   │
│  │     • Queue for archive inclusion                                   │   │
│  │     • Broadcast to network                                          │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────┬───────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             ARCHIVE CHAIN                                   │
│                         (Secure Finality)                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  4. Consensus                                                       │   │
│  │     • PoS: VRF-based validator selection (~5s)                      │   │
│  │     • PoW: Dynamic difficulty mining (~60s)                         │   │
│  │     • PoAuth: Institutional quorum (instant)                        │   │
│  │     • PoUW: Protein folding ZK proofs (variable)                    │   │
│  └───────────────────────────────┬─────────────────────────────────────┘   │
│                                  ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  5. Block Application                                               │   │
│  │     • Fee deduction (same token)                                    │   │
│  │     • Serial range transfer (Ghost)                                 │   │
│  │     • Nonce update                                                  │   │
│  │     • Guardian key auto-rotation (seed never on chain)              │   │
│  │     • Reward minting (URPower, URWork, URStake)                     │   │
│  │     • Burn recording                                                │   │
│  └───────────────────────────────┬─────────────────────────────────────┘   │
│                                  ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  6. Checkpoint                                                      │   │
│  │     • State root hash                                               │   │
│  │     • Block hash                                                    │   │
│  │     • Timestamp                                                     │   │
│  │     • Height                                                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────┬───────────────────────────────────────┘
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PERSISTENT STORAGE                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │    Blocks       │  │    Accounts     │  │   Ghost Chain   │             │
│  │  (Full history) │  │  (State, flags) │  │ (Serial ranges) │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │   Validators    │  │   Sell Orders   │  │   Auth Quorum   │             │
│  │  (Staking info) │  │  (Marketplace)  │  │  (PoAuth keys)  │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
└─────────────────────────────────────────────────────────────────────────────┘
Core Components
1. ⚡ Live Chain
Purpose: Instant transaction processing with optimistic execution.

Characteristics:

In-memory overlay

Sub-millisecond validation (<1ms)

Reversible state updates

Syncs with archive via checkpoints

No persistent storage

Key Operations:

Operation	Description
validate_transaction()	Checks signature, nonce, balance, guardian seed, fee
apply_transaction_optimistic()	Updates overlay immediately (user sees change)
apply_checkpoint()	Syncs with archive final state
Data Structure:

rust
pub struct LiveChain {
    base: LiveBaseState,       // Last checkpoint state (from archive)
    overlay: LiveOverlay,      // Pending optimistic updates (unconfirmed)
    mempool: Vec<Transaction>, // Pending for archive inclusion
}
2. 🔒 Archive Chain
Purpose: Final consensus and permanent state.

Characteristics:

Disk-persistent

Fully deterministic

Consensus-driven (PoW / PoS / PoAuth / PoUW)

Canonical source of truth

Atomic state persistence

Key Operations:

Operation	Description
add_transaction()	Adds to mempool with validation
mine_block()	PoW mining with dynamic difficulty
propose_pos_block()	PoS validator block proposal
apply_block()	State transition with rewards and burns
Data Structure:

rust
pub struct Blockchain<S, M, E> {
    chain: Vec<Block>,                       // Full block history
    accounts: HashMap<String, Account>,      // Account state (nonce, flags)
    ghost: GhostChain,                       // Serial ownership (source of truth)
    pending_transactions: Vec<Transaction>,  // Mempool
    validator_selector: ValidatorSelector,   // PoS validators
    auth_quorum: AuthQuorum,                 // PoAuth quorum
    serial_manager: SerialManager,           // Next serial tracking
}
3. 👻 Ghost Chain (Source of Truth)
Purpose: Canonical tracking of serial ownership and history.

Key Features:

Range-based serial storage (no expansion)

Full ownership history (every serial tracked)

Deterministic rebuild from blocks

Forensic commands for auditability

Operations:

Operation	Description	Complexity
transfer_ranges()	Move serials, split/merge ranges	O(ranges)
get_balance()	Sum range sizes	O(ranges)
get_serial_history()	Look up serial in index	O(log n)
Data Structure:

rust
pub struct GhostChain {
    // Current ownership (source of truth)
    owner_ranges: HashMap<String, BTreeMap<TokenType, Vec<SerialRange>>>,
    
    // Historical tracking (forensics)
    serial_index: BTreeMap<(TokenType, u64), GhostRecord>,
}
SerialRange:

rust
pub struct SerialRange {
    pub start: u64,           // First serial in range
    pub end: u64,             // Last serial in range
    pub token_type: TokenType,
    pub owner: String,
    pub created_at: i64,
    pub last_move: i64,
    pub parent_hash: String,  // Links to block
    pub transfer_count: u64,
}
4. 🔐 Guardian Keys (v2)
Purpose: Dual-factor security with independent seed, auto-rotating.

Core Mechanism:

text
rolling_hash = product of hash(all serial ranges) mod PRIME
guardian_key = SHA256(guardian_seed || rolling_hash || domain)
Key Properties:

Seed generated once on first receive

Seed displayed once, stored locally (~/.Uracil/wallets/alice.json)

Seed never stored on chain

Key auto-rotates every transaction

Both private key and guardian seed required to send

Security Model:

Attacker Has	Can Send?
Private key only	❌ No
Guardian seed only	❌ No
Both	✅ Yes
Local Wallet Structure:

json
{
  "address": "alice",
  "private_key": "...",
  "guardian_seed": "a1b2c3d4e5f67890...",
  "guardian_state": { ... },
  "created_at": 1700000000
}
5. 🔁 Checkpoints
Checkpoints synchronize Live Chain with Archive Chain after each block.

rust
pub struct Checkpoint {
    pub height: u64,
    pub state_root: String,
    pub timestamp: i64,
    pub block_hash: String,
}
Purpose:

Fast state sync for new nodes

Live Chain rollback on reorg

Light client verification

6. 🧬 ZK Protein Folding (PoUW)
Purpose: Validate protein folding computations with zero-knowledge proofs.

Working Implementation:

Nova-based SNARK circuits

Ramachandran constraint validation

13-step audit passed

3 residues: ~4.0s, 5 residues: ~6.5s (projected)

Validation Flow:

text
For each residue:
  ├── Phi/Psi angle normalization
  ├── Strict limit enforcement (0–360°)
  ├── Residue classification (Glycine, Proline, Standard)
  ├── Region claims (α-helix, β-sheet, β-turn)
  ├── Gated range enforcement
  ├── Special case handling
  └── Steric clash detection
Data Flow
Transaction Lifecycle
text
1. User submits transaction (with guardian seed if required)
        │
        ▼
2. Live Chain validates:
   • Falcon signature
   • Nonce
   • Balance (via Ghost)
   • Guardian seed (if initialized)
   • Fee sufficiency
        │
        ▼
3. Applied optimistically to overlay (user sees instant update)
        │
        ▼
4. Broadcast to network (P2P)
        │
        ▼
5. Added to archive mempool
        │
        ▼
6. Block mined/proposed (PoW/PoS/PoAuth)
        │
        ▼
7. Block applied to archive state:
   • Fee deduction
   • Serial range transfer (Ghost)
   • Nonce update
   • Guardian key auto-rotation
   • Reward minting (URPower, URWork, URStake)
   • Burn recording
        │
        ▼
8. Checkpoint published
        │
        ▼
9. Live Chain syncs to checkpoint (overlay cleared)
Security Layers
Layer	Protection	Details
Quantum Signatures	Falcon-512 + SHA3-512	NIST PQC finalist
Consensus	PoW + PoS + PoAuth + PoUW	Quad-consensus hardening
Fork Choice	Weight-based + eclipse detection	Cumulative weight, validator verification
Ghost Chain	Cryptographic serial tracking	Atomic transfers, full history
Guardian Keys (v2)	Dual-factor, auto-rotating	Independent seed, never on chain
PoAuth	Multi-signature quorum	Falcon signatures, 5/7 threshold
Slashing	10% stake penalty	50 missed blocks or 1 hour offline
Performance Characteristics
Operation	Time	Notes
Live validation	<1 ms	Optimistic execution
Falcon signing	36–44 ms	Quantum-resistant
Block mining (diff 1)	<1 s	PoW with difficulty adjustment
Range transfer	~1 µs	Ghost chain operation
Balance query	~10 ns	Range size sum
Serial lookup	1.26 µs avg	O(log n) complexity
Ghost rebuild (10k ranges)	548 µs	Deterministic
Protein folding (3 residues)	~4.0 s	Nova ZK proof
Live chain throughput	16,753 TPS	Validation only
Transfer throughput	5,708 TPS	Including block mining
Directory Structure
text
src/
├── blockchain/      # Archive chain core (state machine, consensus)
├── node/            # Live chain (optimistic execution, overlay)
├── ghost/           # Serial tracking (source of truth, ranges, history)
├── mining/          # Block production (PoW mining, PoS proposal)
├── economics/       # Rewards & fees (dual reward, burns, staking)
├── auth/            # PoAuth quorum (institutional signatures)
├── crypto/          # Falcon signatures, SHA3-512 hashing
├── storage/         # Persistence layer (disk, memory)
├── staking/         # Validator management (VRF selection, slashing)
├── guardian_key/    # Dual-factor security (v2) with local wallet
├── wallet/          # Local wallet storage (guardian seeds, private keys)
├── zk/              # ZK protein folding (working, 3 residues ~4s)
├── serials/         # Serial manager (next serial tracking)
└── types/           # Common types (burn events, etc.)
Configuration
Loaded via config.toml:

toml
[auth]
threshold = 5

[consensus]
min_stake = 1000
max_stake = 10000000
reward_rate = 600
validator_share = 500
burn_share = 100
epoch_length = 100
slash_fraction = 10
blocks_per_year = 525600
idle_mining_factor = 0.1
pouw_reward_multiplier = 1.0
pow_reward_multiplier = 1.0
dual_mine_divisor = 20
min_pos_reward = 1

[consensus.fee]
base_fee = 1
fee_per_byte = 1
value_fee_basis_points = 10
min_fee = 1
max_fee = 1000
State Persistence
All state is stored at: ~/.Uracil/blockchain/blockchain.json

rust
pub struct BlockchainState {
    pub chain: Vec<Block>,
    pub accounts: Vec<Account>,
    pub validator_selector: ValidatorSelector,
    pub sell_orders: Vec<SellOrder>,
    pub auth_quorum: AuthQuorum,
    pub pending_transactions: Vec<Transaction>,
    pub serial_manager: SerialManager,  // Preserves next serials
}
Guarantees:

Atomic writes (rename-based)

Crash-safe persistence

Full replay capability

Serial manager state preserved across restarts

Summary
Uracil Chain introduces a modular, deterministic, and adversarially hardened blockchain architecture with:

Feature	Implementation
Dual-chain execution	Live Chain (instant) + Archive Chain (finality)
Serial-based ownership	Ghost Chain with range-based tracking
Full forensic reconstruction	Every serial has complete history
Multi-layer consensus	PoW / PoS / PoAuth / PoUW
Dual-factor security	Guardian Keys v2 (independent seed, auto-rotating)
ZK protein folding	Working with Ramachandran validation
Strong crash recovery	Atomic persistence, deterministic replay