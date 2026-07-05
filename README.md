# Validator-Key-Compromise-Ronin-Bridge-2022-625M-
What happened: A senior engineer was social-engineered via a fake job offer (malicious PDF), granting attackers access that led to compromise of 5 of 9 validator keys — enough to pass Ronin's multisig threshold and approve fraudulent withdrawals, undetected for six days.
fix(validators): raise multisig threshold, remove standing RPC
allowlist exception, add real-time withdrawal anomaly monitoring

Ref: Ronin Bridge exploit (Mar 2022, $625M — largest crypto hack
ever at the time), attributed to Lazarus Group
Root cause: attacker compromised 5 of 9 validator private keys
(4 directly, 1 via a fake job offer / social engineering granting
RPC access) — enough to forge withdrawal approvals.
Root cause: Not a smart contract bug at all — human/operational security. Too few validators, too low a threshold relative to total validators, and no real-time monitoring caught the drain for nearly a week.
Base takeaway: The scariest hacks aren't smart contract bugs — they're social engineering against your own team. Treat validator/multisig key security and employee security training as seriously as your Solidity code. Six days undetected is the real lesson here, not just the compromise itself.
Faulty Message Verification — Nomad Bridge (2022, $190M)
fix(replica): require non-zero root to be explicitly proven, remove
default-trusted 0x00 initialization value

Ref: Nomad Bridge exploit (Aug 2022, $190M — "the crowdsourced hack")
Root cause: a routine upgrade set the trusted root to 0x00 by
default, which meant ANY message with a 0x00 proof was treated as
already-proven/valid — attackers just copy-pasted the first exploit
transaction with new parameters, and hundreds of copycats joined in.
What happened: After a botched upgrade, the bridge's verification defaulted to "trusted" for a specific proof value. Anyone submitting a message with that trivial value could withdraw funds — no real proof needed. It became one of the first fully crowdsourced hacks, with hundreds of unrelated wallets copying the exploit within hours.
Root cause: An uninitialized/default value was mistakenly treated as "already validated" rather than "not yet validated."
Fix: Default state must always fail-closed (unverified/untrusted), never fail-open. Zero/default values should never satisfy a security check by coincidence.
Base takeaway: After ANY upgrade, explicitly test your default/uninitialized states. "What happens if this value is just zero?" should be a mandatory test case for every verification function you ship on Base.
Donation / Rounding Attack — Euler Finance (2023, $197M)
fix(liquidation): add health-check invariant on donateToReserves(),
prevent self-liquidation exploiting undercollateralized state

Ref: Euler Finance exploit (Mar 2023, $197M — later fully returned)
Root cause: a newly-added donateToReserves() function let a user
donate collateral tokens without a matching debt check, artificially
worsening their own health factor, then self-liquidate for
more than they should've been able to extract.
What happened: The attacker exploited a new "donate to reserves" feature to intentionally tank their own account health, then triggered self-liquidation under conditions the liquidation logic didn't correctly account for — extracting far more value than legitimately possible.
Root cause: A newly shipped feature (donation function) interacted with existing liquidation logic in a way that wasn't tested together — each function was "safe" in isolation, but the combination wasn't.
Fix: Any new function that touches collateral/debt accounting needs to be re-audited against ALL existing financial logic, not just tested standalone. Invariant testing (e.g., "health factor can never be manipulated by the position holder to their own benefit") should be automated in CI.
Base takeaway: Most real hacks in 2023-2024 aren't single obvious bugs — they're safe-looking features that break an invariant elsewhere in the system. Every new function shipped to a Base protocol should be tested against your full existing surface, not in isolation.
Oracle Manipulation — Mango Markets (2022, $114M)
fix(collateral): cap borrowable value against illiquid/thin-market
perp positions, add oracle deviation circuit breaker

Ref: Mango Markets exploit (Oct 2022, $114M, "technically legal"
governance-vote drama afterward)
Root cause: attacker used two accounts to pump MNGO perp price
10x+ via low-liquidity spot trades, then used the inflated paper
value as collateral to borrow out nearly all other assets on the
platform.
What happened: The attacker self-traded MNGO token price up massively on thin liquidity, which inflated the paper value of their perp position, which the protocol then accepted at face value as collateral for borrowing everything else.
Root cause: No limit on how much of the protocol's total liquidity could be borrowed against a single, self-referential, illiquid asset's inflated price.
Fix: Cap collateral value contribution per asset, especially thinly-traded/self-listed tokens. Never let a position's own market impact become the basis for borrowing against the rest of the protocol.
Base takeaway: If your Base lending/perp protocol allows a native or low-liquidity token as collateral, hard-cap its influence on total borrowable value regardless of reported price. Illiquid collateral is manipulable collateral.
Governance Flash Loan Attack — Beanstalk (2022, $182M)
fix(governance): add timelock delay between proposal pass and
execution, require sustained token holding (no flash-loanable votes)

Ref: Beanstalk Farms exploit (Apr 2022, $182M)
Root cause: governance proposals could pass AND execute in a single
transaction if the proposer held >2/3 voting power at that instant
— attacker flash-loaned enough governance tokens to self-approve a
malicious proposal that drained the protocol treasury, all in one tx.
What happened: The attacker flash-loaned enough of the governance token to instantly control a supermajority, submitted and passed a malicious proposal, and had it execute — draining the treasury — all within a single transaction, before repaying the flash loan.
Root cause: No time delay between a governance proposal passing and its execution, and voting power based on instantaneous token balance rather than sustained/locked holdings.
Fix: Require a timelock (24-72h minimum) between proposal approval and execution. Use snapshot-based or vote-escrowed (veToken) voting power so flash loans can't manufacture instant governance control.
