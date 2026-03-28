Guardian Keys: Auto-Rotating Quantum-Resistant Security (v2)

Overview

Guardian Keys introduce a state-bound, dual-factor security system for Uracil Chain accounts.

Unlike traditional blockchain models where a single private key controls funds, Uracil Chain requires:

A private key (Falcon-based identity)

A guardian seed (independent, user-controlled secret)

Every transaction requires both.

Guardian Keys evolve automatically with account state, transforming account security from static authorization into a dynamic, self-evolving system.

Why We Changed the Guardian Key System
The Original Design (v1)
In the first version, guardian keys were derived directly from the user's private key:

text
guardian_key = SHA256(user_private_key || rolling_hash || domain)
The problem: The guardian key was mathematically linked to the private key. If an attacker obtained the private key, they could compute the guardian key themselves — there was no true second factor.

What was stored on chain: The guardian key (including its scalar) was stored in Account.guardian_key on the blockchain, exposing the secret to anyone running a node.

The New Design (v2)
We introduced an independent guardian seed:

text
guardian_key = SHA256(guardian_seed || rolling_hash || domain)
Key improvements:

The guardian seed is separate from the private key

The seed is never stored on the blockchain

Even if an attacker obtains the private key, they cannot derive the guardian key

The seed is displayed once and must be saved by the user

What changed:

Aspect	v1 (Old)	v2 (New)
Derivation	From private key	From independent seed
On-chain storage	Full guardian key (including scalar)	Only version flag
Second factor	❌ No (derived from private key)	✅ Yes (independent)
User responsibility	None	Save guardian seed
Core Principle
text
guardian_key = SHA256(guardian_seed || rolling_hash || domain)
Where:

Component	Description
guardian_seed	Locally stored, user-controlled secret (256 bits)
rolling_hash	Deterministic fingerprint of account state (on chain)
domain	Context separation (prevents cross-protocol reuse)
Design Goals
Guardian Keys are built to provide:

✅ True dual-factor security

✅ Zero on-chain secret exposure

✅ Automatic key rotation

✅ State-bound authorization

✅ Forward security

✅ Quantum resistance

Why Guardian Keys Exist
Problem: Static Key Vulnerability
Traditional blockchain model:

text
private_key → full control
If compromised:

Funds are immediately and permanently at risk

No built-in rotation or invalidation exists

Security depends entirely on one secret

Solution: Dual-Factor + State Evolution
Uracil Chain model:

text
private_key + guardian_seed
           ↓
combined with rolling_hash
           ↓
guardian_key (state-specific)
This ensures:

Authorization depends on two independent secrets

Every transaction uses a new derived key

Compromise of one component is insufficient

Core Mechanisms
1. Rolling Hash (State Fingerprint)
The rolling hash represents the entire ownership state of an account.

text
rolling_hash = 1
for each serial_range:
    rolling_hash = rolling_hash × hash(range) mod PRIME
Properties:

Deterministic

Order-independent (commutative)

One-way (non-reversible)

Updates on every state change

Binds authorization to exact asset ownership

2. Guardian Seed (Independent Secret)
Guardian Keys introduce a separate, independent seed:

text
guardian_key = SHA256(guardian_seed || rolling_hash || domain)
Properties:

Generated once (on first receive)

Displayed once to the user

Stored only locally (~/.Uracil/wallets/alice.json)

Never transmitted or stored on-chain

Not derived from private key

3. Local Evolution Engine
The wallet maintains a local evolving state:

json
{
  "guardian_seed": "a1b2c3d4e5f6...",
  "guardian_state": {
    "n": 42,
    "a": 3.142,
    "s": 0.891,
    "v": 0.456
  }
}
Evolution Properties:

Deterministic (reproducible)

Irreversible (forward-only)

Unique per account

Advances with every transaction (send or receive)

Key Lifecycle
1. First Receive (Seed Generation)
bash
send user1 alice 100 core
System behavior:

Detects first interaction

Generates a 256-bit random guardian seed

Displays seed once with warning

Stores seed locally

Initializes evolution state

text
🔐 GUARDIAN SEED GENERATED FOR alice
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Guardian Seed: a1b2c3d4e5f67890...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS SEED SECURELY. You will need it to:
   - Sign transactions from this account
   - Recover your wallet
This seed is NOT stored on the blockchain.
If you lose it, you CANNOT access your funds.
⚠️ This seed is not recoverable if lost.

2. Sending Tokens
bash
send alice bob 10 core
Flow:

User enters guardian seed

Wallet verifies seed matches local store

Guardian key is derived internally

Transaction signed (private key + guardian key)

Broadcast to network

Local state evolves (n increments)

Prompt:

text
🔐 GUARDIAN SEED REQUIRED
Enter your guardian seed (the one you saved): _
3. Receiving Tokens (Background)
bash
send user2 alice 50 core
No user interaction required

No seed input needed

State evolves automatically (n increments)

What the User Saves
Item	How to Save	What Happens If Lost
Private key	File (~/.Uracil/user_keys/alice.key)	❌ Cannot recover wallet
Guardian seed	Write down or password manager	❌ Cannot send transactions
The guardian seed is displayed once and never again. There is no recovery mechanism.

Versioning System
Two counters track the system:

Counter	Location	Purpose
guardian_version	On-chain	Tracks state transitions
n (evolution count)	Local wallet	Tracks local evolution
Example:

Event	On-chain Version	Local Evolution (n)
Account created	0	0
First receive	1	1
First send	2	2
Second receive	3	3
Second send	4	4
Evolution Triggers
Event	User Action	Evolution
Send	Enter seed	✅ Advances state
Receive	None (background)	✅ Advances state
Both send and receive transactions evolve the guardian state.

Attack Model & Security Boundaries
Attacker Capability Matrix
Attacker Has	Can They Send Transactions?
Private key only	❌ No (guardian seed required)
Guardian seed only	❌ No (private key required)
Both private key + guardian seed	✅ Yes
Why This Model Is Secure
This follows the same trusted structure as real-world systems:

System	Two Factors
Bank account	Password + OTP
2FA wallet	Private key + 2FA code
Uracil Chain	Private key + Guardian seed
If both factors are compromised, access is possible — this is a known and accepted property of all dual-factor systems, not a weakness unique to Uracil Chain.

What Guardian Keys Protect Against
Attack Scenario	Without Guardian Keys	With Guardian Keys
Private key stolen	❌ Funds lost	✅ Safe (seed required)
Private key brute-forced	❌ Funds lost	✅ Safe (seed required)
Guardian seed stolen only	—	✅ Safe (private key required)
Both compromised	❌ Funds lost	❌ Funds lost
The Real Security Gain
Guardian Keys fundamentally change the threat model:

Before	After
Single secret	Two independent secrets
Single failure point	Dual-factor requirement
File theft = full loss	File theft alone insufficient
This transforms security from single-point failure to multi-layer defense.

Security Properties
1. True Dual-Factor Security
Authorization requires:

Identity (private key)

Authorization (guardian seed)

2. No On-Chain Secrets
Data	On Chain?
Guardian seed	❌ No
Evolution state	❌ No
Rolling hash	✅ Yes
Version counter	✅ Yes
3. Forward Security
Compromised key affects only current state

Future states remain secure

Past states cannot be reconstructed

4. State Binding
Each key is tied to:

Exact serial ownership

Exact moment in time

Any change invalidates previous keys.

5. Quantum Resistance
Falcon signatures (post-quantum)

SHA256-based derivation

No classical cryptographic assumptions

Mathematical Foundation
Field
text
MODULUS = 2²⁵⁶ - 189
Rolling Hash
text
H_new = H_old × hash(range) mod MODULUS
Guardian Key
text
guardian_key = SHA256(guardian_seed || rolling_hash || domain)
Evolution Dynamics
The evolution engine combines:

Oscillatory decay systems

Transaction-driven mutation

Convergence optimization (ALM)

Result:

Non-linear evolution

Non-reversible state path

Unique trajectory per account

Commands
Initialize (Implicit on First Receive)
bash
send user1 alice 100 core
Seed generated and displayed automatically.

Send Transaction
bash
send alice bob 10 core
Prompt:

text
Enter guardian seed: _
Check Status
bash
guardian_status alice
Output:

text
Address: alice
Guardian Version: 4
Guardian System: ACTIVE (v2)
  - Seed generated on first receive
  - Auto-rotates every transaction
  - Seed stored locally (not on chain)
  - Local wallet: ✅ Present
  - Evolution count: 2
System Comparison
Model	Security
Traditional	private_key → full control
Uracil Chain v1	Key derived from private key (no true 2FA)
Uracil Chain v2	private_key + guardian_seed → state-bound key → auto-rotating authorization
Summary
Feature	Benefit
Independent seed	True dual-factor security
No on-chain storage	Zero secret leakage
Auto-rotation	No key reuse
State binding	Prevents replay
Forward security	Limits compromise window
Quantum resistance	Future-proof
Final Note
Guardian Keys transform account security from:

Before	After
Static	Dynamic
Single-factor	Dual-factor
Persistent	Self-evolving
This is not just key rotation — it is state-driven cryptographic authorization with layered defense. The seed is independent, displayed once, and must be saved by the user. It never touches the blockchain, making it a true second factor.