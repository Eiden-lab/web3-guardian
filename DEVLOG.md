# Development log

Newest first. One entry per meaningful step.

## 2026-10-29
- Node hardened: RPC addresses never appear in error logs.
- Interactive walkthrough of the Sepolia attack proof.

## 2026-10-28
- **Public attack proof on Sepolia:** Bybit-style rewrite, approval drain and off-chain Permit2 attack all refused
  on-chain; an ordinary payment still executed.
- Continuous integration, security policy, gas cost measurements (about +18,000 gas per Safe transaction).
- Signature firewall: off-chain signatures only count for messages the owners approved on-chain.
- 25 theft scenarios on a mainnet fork with real USDC, Permit2 and Safe contracts.

## 2026-10-16
- Alerting node: tested polling loop, chain check at startup, deployment files.

## 2026-10-11
- Safe 1.5.0 support, including a guard for modules (modules can no longer bypass the delays).

## 2026-10-08
- Hardened guard: approved Safe upgrades, fast lane for removing a compromised signer, stronger detection of a
  rewritten Safe.

## 2026-10-04
- Configurable delays and a testnet variant with minute-long delays for demos.
- First alerting node with Telegram messages.

## 2026-10-03
- Batched transactions (MultiSend) decoded call by call, recursively.
- Real WazirX attack replayed; verified identifiers for the Bybit and WazirX incidents.

## 2026-10-02
- Queueing requires the owners' full quorum of signatures.
- Real Bybit attack replayed: refused.
