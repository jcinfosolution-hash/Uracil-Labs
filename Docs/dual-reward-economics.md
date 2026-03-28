Dual Reward Economics
Overview

Uracil Chain implements a dual-reward economic system that separates:

Computational security → rewarded with URPower
Scientific contribution → rewarded with URWork

This separation ensures that:

Network security is always incentivized
Real-world useful computation is independently rewarded
Economic inflation is controlled and purpose-driven
Core Model
          Compute Contribution
        ┌───────────────────────┐
        │                       │
   Mining (PoW)         Productive Work (PoUW)
   → URPower            → URWork
        │                       │
        └──────────┬────────────┘
                   ▼
            Dual Reward Layer

In advanced cases (e.g., verified computation), a user can earn both reward types simultaneously.

Token System
Token	Purpose	Minting	Supply Model
UR-CORE	Primary currency	Genesis	Fixed
UR-POWER	Mining rewards	Dynamic	Inflationary
UR-WORK	Scientific rewards	Dynamic	Inflationary
UR-STAKE	Validator staking	Genesis + rewards	Bounded
UR-AUTH	Institutional quorum weight	Genesis	Fixed
Reward Mechanisms
1. Mining Rewards (URPower)

Mining rewards are based on contributed compute power and time.

Formula
base_reward = (hash_rate × duration_seconds) / 1,000,000
user_reward = base_reward × 85%
burn = base_reward × 15%
Burn Distribution
90% → URPower burned
10% → URCore burned
Key Properties
Scales with compute contribution
Automatically adjusted via network activity
Includes built-in deflation through burn
2. Productive Work Rewards (URWork)

Rewards are generated from validated scientific computation (e.g., protein folding).

Formula
base_reward = sequence_length × 10
quality_multiplier = f(RMSD)   // Range: 0.5 to 1.2
user_reward = base_reward × quality_multiplier
Key Properties
Rewards quality, not just effort
Encourages accurate scientific results
Independent from mining incentives
3. Staking Rewards (URStake)

Validators earn rewards based on total network stake.

Formula
epoch_reward =
  (total_staked × reward_rate) /
  (blocks_per_year / epoch_length) / 100

validator_share ≈ 83.3%
burn ≈ 16.7%
Key Properties
~6% annual reward rate
Distributed at epoch boundaries
Includes built-in burn for inflation control
Dynamic Burn System

Burning is used to control inflation and maintain economic balance.

Mining Burn (Fixed)
15% of mining rewards burned
Staking Burn
~1% annual burn from staking rewards
Productive Work Burn (Dynamic)

Burn rate adjusts based on computation quality:

Factor	Effect
High accuracy (low RMSD)	Lower burn
Low accuracy	Higher burn
Complex computation	Lower burn
Simple computation	Higher burn

Range: 1% – 10%

Fee System
Fee Model

Fees are always paid in the same token being transferred.

Transaction	Fee Token
UR-CORE transfer	UR-CORE
UR-POWER transfer	UR-POWER
UR-WORK transfer	UR-WORK
Fee Calculation
base_fee = fixed minimum
size_fee = proportional to transaction size
value_fee = % of transfer amount (UR-CORE only)

total_fee = (base_fee + size_fee + value_fee) × congestion_multiplier
Fee Distribution
validator_share = 70%
burn_share = 20%
treasury_share = 10%
Congestion Pricing

Fees scale dynamically with network load:

Mempool Size	Multiplier
Low	1.0×
Medium	1.5× – 2.0×
High	3.0×
Extreme	5.0×

This ensures:

Spam resistance
Fair prioritization
Stable network performance
Dual Reward Execution

Certain operations combine both reward types:

VerifiedPoUW:
  → URWork (scientific validation)
  → URPower (network contribution)

This creates a hybrid incentive model:

Users contribute compute
Users produce useful results
Both are rewarded independently
Economic Invariants

The system enforces strict rules:

Supply Integrity
Total supply = sum of all token ranges
Only minting increases supply
Only burning decreases supply
Fee Integrity
total_fee = validator + burn + treasury
Reward Integrity
All rewards are protocol-calculated
User-defined rewards are rejected
Economic Security
Inflation Control
Mining rewards → dynamic
Staking rewards → fixed rate
Burn mechanisms → continuous

Net inflation remains controlled and predictable

Anti-Spam Protection
Minimum transaction fee
Congestion multiplier
Token-specific fee payment
Fair Distribution
Mining → open to all compute providers
Staking → proportional to stake
Productive work → based on contribution quality

No pre-mine. No privileged reward injection.

Burn Transparency

All burns are recorded and auditable:

pub struct BurnEvent {
    transaction_id: String,
    user: String,
    token_type: TokenType,
    amount: u64,
    timestamp: i64,
    block_height: u64,
    reason: String
}

This ensures:

Full transparency
Verifiable supply changes
Audit-ready economics
Summary
Mechanism	Token	Purpose
Mining	URPower	Network security
Productive Work	URWork	Scientific computation
Staking	URStake	Consensus participation
Fees	Same token	Transaction processing
Final Insight

Uracil Chain’s economic model is built on a clear principle:

Different types of value should be rewarded differently.

Security is not the same as usefulness
Compute is not the same as knowledge
Rewards reflect contribution type

This separation creates a system that is:

More stable
More fair
More aligned with real-world value creation