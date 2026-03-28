Ghost Chain: The Source of Truth
Overview

The Ghost Chain is the canonical source of truth for all token ownership and balances in Uracil Chain.

Instead of storing balances as simple numbers, the system tracks every token as a unique serial, grouped efficiently into ranges. This allows:

Deterministic state reconstruction
Full historical traceability
Strong protection against double-spending
Efficient storage at scale
┌──────────────────────────────────────────────────────────────┐
│                       Ghost Chain                            │
│                   (Source of Truth)                          │
├──────────────────────────────────────────────────────────────┤
│ owner_ranges: address → token → [ranges]                     │
│ serial_index: (token, serial) → full history                 │
├──────────────────────────────────────────────────────────────┤
│ • All balances derived from ranges                           │
│ • All transfers validated via range ownership                │
│ • Full history preserved for every serial                    │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                    Archive Chain                             │
│             (Consensus + Checkpoints)                        │
└──────────────────────────────────────────────────────────────┘
Why “Ghost Chain”?

The system compresses ownership into ranges while preserving full history.

Like a “ghost,” the detailed past is not always visible—but it is always there when needed.

Storage Model	Memory Usage	Query Speed	History
Individual Serials	O(n)	O(log n)	Full
Ghost Ranges	O(log n)	O(log n)	Full
Core Data Structures
1. SerialRange

Represents a contiguous block of owned serials.

pub struct SerialRange {
    pub start: u64,
    pub end: u64,
    pub token_type: TokenType,
    pub owner: String,
    pub created_at: i64,
    pub last_move: i64,
    pub parent_hash: String,
    pub transfer_count: u64,
}
2. Owner Ranges (Primary Storage)
owner_ranges: HashMap<String, BTreeMap<TokenType, Vec<SerialRange>>>

Maps:

address → token_type → list of ranges

This is the primary source of truth for balances.

3. Serial Index (Historical Layer)
serial_index: BTreeMap<(TokenType, u64), GhostRecord>

Maps:

(token_type, serial) → full ownership history

Used for:

Forensics
Debugging
Audit trails
Range Operations
Transfer (Core Mechanism)

Every transfer is implemented as a range movement.

Steps:

Take serials from the end of sender ranges (edge-based spending)
Split range if partially used
Assign transferred portion to receiver
Merge adjacent ranges for both parties
Record history in serial_index
pub fn transfer_ranges(
    &mut self,
    from: &str,
    to: &str,
    token_type: TokenType,
    amount: u64,
    timestamp: i64,
    block_hash: String,
) -> Result<Vec<SerialRange>, String>
Split Example
Before:
  Sender → [1–100]

Transfer:
  30 serials

After:
  Sender   → [1–70]
  Receiver → [71–100]
Merge Example
Before:
  [1–50] + [51–100]

After:
  [1–100]
Balance Calculation
get_balance()

All balance queries across the entire system rely on Ghost Chain.

pub fn get_balance(&self, owner: &str, token_type: TokenType) -> u64 {
    let mut total = 0;
    if let Some(by_token) = self.owner_ranges.get(owner) {
        if let Some(ranges) = by_token.get(&token_type) {
            for range in ranges {
                total += range.size();
            }
        }
    }
    total
}

Important:
There is no separate balance storage.
Balances are always derived from ranges.

Forensic History
get_serial_history()
pub fn get_serial_history(
    &self,
    token_type: TokenType,
    serial: u64
) -> Option<&GhostRecord>

Returns:

Current owner
Previous owner
First seen timestamp
Last movement
Parent block hash
Transfer count
Example
ghost_history 100060000000001
Token Type: URPower
Current Owner: institution1
First Seen: 2026-03-25T04:21:55Z
Last Move: 2026-03-25T04:21:55Z
Transfer Count: 0
Parent Block: 0000874ed6e617b6...
Genesis Range Initialization

Each token is initialized with a dedicated serial space:

URCore   → 1 to 115,940,000,000,000
URStake  → 1 to 290,000,000,000
URWork   → 1 to 1,750,000,000,000
URPower  → 1 to 100,060,000,000,000
URVote   → 1 to 11,000,000,000
URAuth   → 1 to 5,500,000,000
URTime   → 1 to 3,300,000,000
URId     → 1 to 1,100,000,000

Each token type has isolated serial space (no overlap).

Performance Characteristics
Operation	Complexity	Typical Time
Balance query	O(ranges)	~10 ns
Transfer	O(ranges)	~1 µs
Merge	O(r log r)	~2 µs
History lookup	O(log n)	~50 ns
Storage	O(ranges)	Very compact
Debug & Forensic Commands
ghost_status alice
ghost_history <serial>
check_serial <serial> <owner>
state_hash
Security Properties
1. Cryptographic Lineage

Each range includes parent_hash, linking it to the block where it was created or modified.

2. Double-Spend Prevention
Ranges are removed from sender before assignment
Transfers are atomic
No serial can exist in two places
3. Deterministic State

Given the same block sequence:

Ghost state is always identical

This is critical for consensus safety.

4. Range Invariants
No overlapping ranges per owner/token
All ranges remain contiguous
Total supply equals sum of all ranges
Why Ghost Chain Matters
Without Ghost	With Ghost
Balances are numbers	Tokens have identity
No history	Full forensic trace
O(n) storage	O(log n) storage
Weak verification	Strong ownership proof

Ghost Chain transforms Uracil Chain into a true serial-based blockchain, where:

Every unit is uniquely identifiable
Every transfer is traceable
Every state is verifiable
Integration with Archive Chain

The Archive Chain relies entirely on Ghost Chain for correctness.

pub fn get_balance(&self, address: &str, token_type: TokenType) -> u64 {
    self.ghost.get_balance(address, token_type)
}

pub fn apply_block(&mut self, block: Block) -> Result<(), String> {
    for tx in block.transactions {
        self.ghost.transfer_ranges(...)?;
    }
}
Checkpoints

Each checkpoint includes a state root derived from Ghost data, ensuring:

Network-wide consistency
Deterministic replay
Consensus integrity
Final Insight

Ghost Chain is not just a storage optimization.

It is the foundation of correctness, security, and auditability in Uracil Chain.