Uracil Chain Consensus
Overview
Uracil Chain implements a multi-layer (quad) consensus architecture designed to balance:

Speed (instant confirmations)

Security (cryptographic + economic guarantees)

Decentralization (validators and miners)

Utility (real-world compute via PoUW)

The system combines four complementary mechanisms:

Consensus	Role	Participants	Finality	Status
Proof-of-Stake (PoS)	Primary block production	Validators	~5 seconds	✅
Proof-of-Work (PoW)	Security anchor & anti-spam	Miners	~60 seconds	✅
Proof-of-Authority (PoAuth)	Institutional fast-track finality	Authorized entities	Instant	✅
Proof-of-Useful-Work (PoUW)	Productive compute (ZK protein folding)	Researchers / compute providers	Variable	✅ Working
Difficulty & Block Timing
PoW Difficulty Adjustment
The network dynamically adjusts difficulty to maintain a target block time of 60 seconds.

Formula:

text
avg_time = average block time (last 10 blocks)
ratio = target_time / avg_time
new_difficulty = current_difficulty × ratio
Constraints:

Maximum increase: 2×

Maximum decrease: 0.5×

Difficulty bounds: [1, 20]

rust
pub fn calculate_difficulty(&self, blockchain: &Blockchain) -> u64 {
    let avg_time = total_time / block_count;
    let ratio = 60.0 / avg_time.max(1.0);
    let new_diff = (current_diff as f64 * ratio).round() as u64;
    new_diff.clamp(min_difficulty, max_difficulty)
}
Special Case: PoS blocks have difficulty = 0 (no mining required)

Fork Choice Rule
Uracil Chain uses a deterministic fork choice algorithm to ensure network convergence.

Weight Calculation
text
block_weight = tx_count + total_rewards + difficulty
cumulative_weight = sum(block_weight across chain)
Chain Selection Priority
Higher block height wins

If equal height → higher cumulative weight wins

If equal weight → lexicographically smaller block hash wins

rust
pub fn should_adopt_chain(current: &[Block], candidate: &[Block]) -> bool {
    let current_height = current.last().map(|b| b.index).unwrap_or(0);
    let candidate_height = candidate.last().map(|b| b.index).unwrap_or(0);

    if candidate_height > current_height {
        return true;
    }

    if candidate_height == current_height {
        let current_weight = current.last().unwrap().cumulative_weight;
        let candidate_weight = candidate.last().unwrap().cumulative_weight;

        if candidate_weight > current_weight {
            return true;
        }

        if candidate_weight == current_weight {
            let current_hash = &current.last().unwrap().hash;
            let candidate_hash = &candidate.last().unwrap().hash;
            return candidate_hash < current_hash;
        }
    }

    false
}
Network Attack Protection
Incoming chains are validated against multiple conditions:

1. Validator Verification
Reject chains containing unknown validators.

2. Signature Validation
PoS blocks must have valid proposer signatures

PoW blocks must satisfy difficulty

3. Genesis Consistency
Reject chains with mismatched genesis hash.

4. Weight Sanity Check
text
weight_ratio = candidate_weight / current_weight
Reject if < 0.5 (potential eclipse attack)
1. Proof-of-Stake (PoS)
Validator Selection (VRF-Based)
Validators are selected using a Verifiable Random Function (VRF):

text
selection_point = VRF(seed) % total_stake
Each validator owns a stake range. The selected validator is the one whose range contains the selection point.

rust
pub fn select_validator_vrf(&self, seed: &[u8]) -> Option<&Validator> {
    let active: Vec<&Validator> = self.validators
        .values()
        .filter(|v| v.is_active)
        .collect();

    let random = hash(seed);
    let idx = vrf.select_validator(&random, &active)?;
    Some(active[idx])
}
Validator Requirements
Parameter	Value
Minimum stake	100 URStake
Maximum stake	10,000,000 URStake
Lock period	7 days
Commission	0–30%
Rewards
Validators receive:

70% of transaction fees from their blocks

Epoch staking rewards (every 100 blocks)

Proposer bonus (if selected by VRF)

Slashing Conditions
50 consecutive missed blocks → 10% slash

Offline >1 hour → 10% slash

rust
pub fn should_slash(&self) -> bool {
    self.missed_blocks_in_a_row >= 50 ||
    (now - self.last_heartbeat) > 3600
}
2. Proof-of-Work (PoW)
Purpose
Provides security anchor

Prevents spam and cheap attacks

Adds economic cost to chain rewriting

Reward Model
text
base_reward = (hash_rate × duration) / 1,000,000
user_reward = 85%
total_burn = 15%
Burn Breakdown:

13.5% → URPower burned

1.5% → URCore burned

Commands
bash
mine
qmine
start_mining 100000 bob
Quantum-Resistant PoW
Hash: SHA3-512

Signatures: Falcon

3. Proof-of-Authority (PoAuth)
Purpose
PoAuth enables institutional fast finality using a weighted multi-signature quorum.

Validator Structure
rust
pub struct AuthValidator {
    pub address: String,
    pub weight: u8,        // 1–5
    pub active: bool,
    pub institution_name: String,
    pub public_key: Vec<u8>,
}
Proposal Lifecycle
Proposal created (block hash)

Validators sign using Falcon keys

If total weight ≥ threshold → finalized

Block fast-tracked

Threshold
Default: 5 total weight

Examples:

1 validator (weight 5) → sufficient

5 validators (weight 1 each) → all required

2 validators (weight 3 each) → both required

Commands
bash
auth_init alice
auth_register <addr> 3 "Institute"
auth_propose <block_hash>
auth_sign <proposal_id> <addr>
auth_status
4. Proof-of-Useful-Work (PoUW)
Status: ✅ WORKING — Fully implemented and audited.

Purpose
Validates protein folding computations using zero-knowledge proofs with Ramachandran constraints.

Validation Logic
Each residue must satisfy:

Phi/Psi angle constraints (Ramachandran plot)

Region classification (α-helix, β-sheet, β-turn)

Special cases (Glycine, Proline)

Steric clash avoidance (forbidden zones)

Audit Status (13-Step Audit Complete)
Step	Description	Status
1–2	Sound constraint-based comparisons	✅ Passed
3	No get_value() in logical decisions	✅ Passed
4	Underflow protection	✅ Passed
5	Explicit boolean integrity	✅ Passed
6	Malicious prover test	✅ Passed
7	Gated ranges audit	✅ Passed
8	Constraint count & performance	✅ Passed
9	Production prep (batch proving)	✅ Passed
10	Formalization	✅ Passed
11	Optimization	✅ Passed
12	Integration prep (ZK privacy)	✅ Passed
13	Overall soundness	✅ Passed
Result: Honest proofs verify; malicious proofs rejected (UnSat).

Performance
Residues	Time	Status
3	~4.0s	✅ Working
5	~6.5s (projected)	Ready
10	~13.5s (projected)	Scales linearly
Per residue	~1.35–1.9s	Linear scaling
Reward Model
text
base_reward = sequence_length × 10
quality_multiplier = f(RMSD)  // 0.5 to 1.2
user_reward = base_reward × multiplier
Dynamic Burn (1–10%)
Factor	Adjustment
High quality (RMSD < 2Å)	-2% (lower burn)
Medium quality (RMSD 2–5Å)	-1%
Low quality (RMSD 8–10Å)	+2% (higher burn)
Long sequences (>50 residues)	-1%
Short sequences (<20 residues)	+1%
Compute time >1s	-1%
Optimization Discovery
Pattern analysis of the circuit reveals:

95% pattern repetition across residues

94% theoretical compression possible

7 unique pattern types repeated 26× per residue

Commands
bash
fold_protein AAA alice
mine
balance alice work
Burn System (Global)
Source	Burn	Token	Notes
Transaction Fees	20%	Same as transaction	70% validator, 20% burn, 10% treasury
Mining Rewards	15%	90% URPower, 10% URCore	PurePoW and DualMineTask
Staking Rewards	1% annual	URStake	Epoch-based (100 blocks)
Protein Folding (PoUW)	1–10% dynamic	URWork	Based on RMSD, sequence length, compute time
Burn Tracking
rust
pub struct BurnEvent {
    pub transaction_id: String,
    pub user: String,
    pub token_type: TokenType,
    pub amount: u64,
    pub serials_burned: Vec<u64>,
    pub timestamp: i64,
    pub block_hash: String,
    pub block_height: u64,
    pub reason: String,
}
Guardian Keys (Dual-Factor Security)
Guardian Keys provide a true second factor for account security:

Independent seed (generated once, displayed once, stored locally)

Seed never stored on the blockchain

Auto-rotates every transaction

Both private key and guardian seed required to send

Attacker Has	Can Send?
Private key only	❌ No
Guardian seed only	❌ No
Both	✅ Yes
Commands:

bash
guardian_status alice
Epoch System
Epoch length: 100 blocks

At each epoch:

Staking rewards distributed

Slashing applied

Validator states updated

1% of rewards burned

Summary
Mechanism	Strength	Speed	Role	Status
PoS	High	~5s	Main consensus	✅
PoW	Very High	~60s	Security anchor	✅
PoAuth	Institutional	Instant	Fast-track	✅
PoUW	Utility-driven	Variable (~4s/3 residues)	Real-world compute	✅ Working