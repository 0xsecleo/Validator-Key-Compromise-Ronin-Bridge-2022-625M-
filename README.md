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
