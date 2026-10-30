# Web3 Guardian

**Stopping the multisig thefts that signatures alone cannot stop.**

> **Status (October 2026):** working prototype, tested against replays of real attacks and proven on a public testnet.
> **Not audited.** The source code is private while the audit and licensing are prepared. This repository is the
> project's public overview, proof and development log.

## The problem

The largest multisig losses did not break any cryptography. In **Bybit** (February 2025, about $1.5B), **WazirX**
(July 2024, about $230M) and **Radiant** (October 2024, about $50M), honest signers approved a transaction that their
screen misrepresented. The signatures were valid, so the Safe executed it immediately. Monitoring tools saw the attack
happen and could not stop it.

The weak point is not the keys. It is the moment between "the owners signed" and "the funds are gone": today, that
moment does not exist.

## The idea

Web3 Guardian is an on-chain guard for [Safe](https://safe.global) multisigs. It gives that moment back:

1. **Every transaction waits in proportion to its risk.** An ordinary payment waits about an hour; rewriting the Safe,
   or moving most of the treasury, waits two days. Owners have time to see what they really signed and cancel it.
2. **It checks what a transaction actually does, not only what it says.** A transaction that moves more than it
   declared, or quietly rewrites the Safe, is refused when it runs, even after its delay.
3. **It closes the doors that no transaction guard sees**, such as off-chain signatures that drain a treasury without
   any Safe transaction.
4. **It explains each transaction in plain language** before it can run, so a tricked signer gets a second look.

The delays are enforced on-chain. Nothing off-chain can unlock a transaction early.

## Proof

### Public attack proof on Sepolia (28 October 2026)

Attacks were queued with valid owner signatures, as tricked signers would, and attempted after their delay with
nobody cancelling. Every outcome is recorded on-chain.

| Attack | Result | Transaction |
|---|---|---|
| Bybit-style rewrite of the Safe | Refused | [0x1fbd…08eb](https://sepolia.etherscan.io/tx/0x1fbdbd2c3a8fe1464c5bfbb6f39b46fd046c31a15c11f5d66548c950db7308eb) |
| Pull 50% of the treasury through an existing approval | Refused | [0x6c51…8965](https://sepolia.etherscan.io/tx/0x6c511c0ed514d38e2ae84070f827448a220080f08168178152759447ea758965) |
| Permit2 permit signed off-chain (no Safe transaction) | Refused | [0x3385…e1b0](https://sepolia.etherscan.io/tx/0x33853e3edd186df30f6192b0baab32301520ab1f5e039384886dc5557345e1b0) |
| Execute before the delay | Refused | [0xfe7e…a60b](https://sepolia.etherscan.io/tx/0xfe7ec819b01be57c6a50a0a2f1fba2464e375379745ace282460a439ee73a60b) |
| Queue with one signature on a 2-of-3 Safe | Refused | [0x0c93…4b8b](https://sepolia.etherscan.io/tx/0x0c931ed363673f0893f868f7a6e53caf7288b44d4bbb428ed1a6539de6fd4b8b) |
| Owners cancel a drain after the alert | Cancelled | [0xa804…083f](https://sepolia.etherscan.io/tx/0xa8047d0b7df523c12692931db84c43d82b9a3577830110641c4857243b8f083f) |
| Ordinary 2% payment (control) | **Executed** | [0xc09b…ab5b](https://sepolia.etherscan.io/tx/0xc09bb57cd6380647dcda240e22157b782bf09f495fb48073c8d67b0a9f96ab5b) |

The Safe kept 980,000 of its 1,000,000 test dollars (only the payment left). The attacker received nothing.

Guard on Sepolia: [`0xa850c0653F3623D8b7c7032139fBD1577445B981`](https://sepolia.etherscan.io/address/0xa850c0653F3623D8b7c7032139fBD1577445B981)

Interactive walkthrough of the proof: [`demo/index.html`](demo/index.html).

### The Binance hack, replayed (30 October 2026)

On 29 October 2026 an attacker who controlled Binance's withdrawal service drained about **$6.3B** from its hot
wallets. One wallet lost 1.615B USDT in **3 minutes 24 seconds**. Those funds were not in a Safe, so Web3 Guardian could
not have protected them. The replay asks: what if the same funds had sat in a Safe with Web3 Guardian, against an
attacker with the same power (valid owner signatures on anything)?

| | Real hack | With Web3 Guardian |
|---|---|---|
| Sweep everything in one transaction | 100% gone in 3 min 24 s | Refused for 48 h. At 56 min, when Tether froze the stolen USDT, the vault still held 100% |
| The team reacts within hours | Too late | Sweep cancelled: nothing lost |
| Sweep split into slices, team reacts after the first | — | Only the first small slice (5%) leaves |
| Automated executor pulls 823M USDC | All of it | Full pull refused; a small daily amount at most |

Replayed on Sepolia with the same 1,615,000,000 tokens (testnet delays are minutes):

| Step | Result | Transaction |
|---|---|---|
| Full sweep, at once | Refused | [0xff3c…b724](https://sepolia.etherscan.io/tx/0xff3c3a2542e427d1e3ca2d89aa2e1837ba598b91b4819badc6dfa2c513ddb724) |
| Independent sentinel adds time | Delayed | [0x23ef…b772](https://sepolia.etherscan.io/tx/0x23ef71ec3502f60dfdb1ea88269c4512af26bab398eb7ccccd0cc63d1bddb772) |
| Full sweep, after the owners cancelled | Refused for good | [0x20ff…1730](https://sepolia.etherscan.io/tx/0x20ffa3468a99c1167a029a1fd103dc740eaae61aecc60e4ba825e09d1f311730) |
| First 5% slice, before anyone reacted | Executed | [0xb5c2…ba59](https://sepolia.etherscan.io/tx/0xb5c29e33b784ff0a62ccf91f75fb5bd491b3762176cdef6a9c4bb79206e9ba59) |
| Remaining 25% and 70% slices, cancelled | Refused | [0x7a1e…ea70](https://sepolia.etherscan.io/tx/0x7a1e909cdde02052eb4396469e442eb6cab1df06edda73d69ea4d2baf91dea70), [0x160e…b8f](https://sepolia.etherscan.io/tx/0x160e8f28dd42ca10ee4322655da1acdd59300e9e7cc124022d2437ce66fc6b8f) |

**The Safe kept 95%.** In the real hack, 100% was gone in minutes. Web3 Guardian buys the time; people still have to
cancel. Next on the roadmap because of this hack: approved destinations and fixed caps, so even the first slice cannot
reach an unknown address.

### Real attacks replayed

The real [Bybit](https://etherscan.io/tx/0x46deef0f52e3a983b67abf4714448a41dd7ffd6d32d32da69d62081c68ad7882) and
[WazirX](https://etherscan.io/tx/0x48164d3adbab78c2cb9876f6e17f88e321097fcd14cadd57556866e4ef3e185d) attack
transactions, replayed byte for byte on a mainnet fork: without the guard, each one takes over the Safe; with it, each
one waits and is refused.

### Test suite

92 tests in continuous integration, including 25 theft scenarios on a mainnet fork with real USDC, the real Permit2
and Safe's real contracts, plus fuzzing and invariant tests.

## Who it is for

DAOs, foundations, associations and companies that keep a treasury in a Safe and whose signers could be tricked, by a
compromised interface, a compromised device or a phishing message.

**What it does not do:** an attacker who already *holds* enough keys can be slowed down, not stopped. Web3 Guardian
protects honest signers who are deceived.

## Roadmap

- [x] Guard prototype, real-attack replays, theft scenarios
- [x] Alerting node (watches, simulates, explains each transaction)
- [x] Public attack proof on Sepolia
- [x] Binance hack (October 2026) replayed on a mainnet fork and on Sepolia
- [ ] Approved destinations and fixed caps
- [ ] Sign through the usual Safe app (relay)
- [ ] Pilots on small Safes with associations
- [ ] Independent security audit
- [ ] Mainnet and L2 deployments

Progress is logged in [DEVLOG.md](DEVLOG.md).

## Contact

Built by [Eiden-lab](https://github.com/Eiden-lab). Interested in a pilot, an audit or a grant conversation? Open an
issue on this repository.

---

Copyright © 2026 Eiden-lab. All rights reserved.
