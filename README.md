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
