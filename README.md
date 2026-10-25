# Billium

**A peer-to-peer mining game chain and non-custodial Solana wallet.**

Mine blocks on an ordinary desktop computer and earn **BILLIUM**, an SPL token on Solana.

> **Status:** Draft v1.0, under development. Parameters, fees, and timelines may change before mainnet launch.

---

## What is Billium?

Billium is a desktop gaming ecosystem made of four parts:

- **Billium Core**: a desktop app with a non-custodial Solana wallet, a lightweight full node, and a SHA-256d miner.
- **Billium Game Chain**: a Bitcoin-style proof-of-work chain that records blocks and rewards only. It has no smart contracts and no spendable fees.
- **Payout Service**: an operator-run service that independently verifies claimed blocks, waits for confirmations, and pays BILLIUM on Solana.
- **BILLIUM token**: a classic SPL token. Rewards are settled on Solana, not on the game chain.

The design targets low-spec machines. At the initial target, a typical 1-2 MH/s CPU finds a solo block in roughly 9-18 minutes. A difficulty ceiling keeps the network from becoming too hard for such machines.

```text
+-----------------------------+   P2P    +-----------------------------+
| Billium Core (desktop)      |<-------->| Other Billium Core nodes    |
| wallet / full node / miner  |          +-----------------------------+
+--------------+--------------+
               | HTTPS
               v
+-----------------------------+   RPC    +-----------------------------+
| Payout Service (operator)   |<-------->| Solana                      |
| observer node / verifier    |          | SPL Token, BILLIUM mint     |
+-----------------------------+          +-----------------------------+
```

## Key parameters

| Parameter | Value |
|---|---|
| Proof-of-work | SHA-256d |
| Block time target | 600 seconds |
| Initial / hardest target | `2^226` / `2^220` |
| Retarget | every 20 blocks, clamped to 4x |
| Initial block reward | 1,125 BILLIUM |
| Halving interval | 200,000 blocks |
| Total supply | 1,000,000,000 BILLIUM |
| Allocation | 45% mining, 45% liquidity, 10% developer reserve |
| Mining cap | 450,000,000 BILLIUM |
| Token decimals | 9 (read on-chain) |
| Payout confirmations | 6 (default) |
| Default / max mining threads | 2 / 4 |
| Peer limit | 16 |

## How it works

1. Import an existing Solana wallet.
2. Activate it once with a single atomic transaction that creates your BILLIUM associated token account (ATA) and pays the activation fee.
3. Register the wallet with the payout service.
4. Connect to peers and sync the game chain.
5. Mine. Broadcast any valid block to peers.
6. Submit a claim. After the confirmation policy is met, BILLIUM is sent to your ATA.

### Activation cost

| Item | SOL |
|---|---:|
| ATA rent (queried dynamically) | 0.00203928 |
| Admin fee (one-time, non-refundable) | 0.00408000 |
| Network transaction fee (estimate) | 0.00000500 |
| **Estimated total** | **0.00612428** |

If the ATA already exists, the cost is about 0.004085 SOL.

## Requirements

- Windows 10 or 11
- Java 17
- 4-core CPU, 4 GB RAM (reference low-spec machine); heap target below 512 MB
- A Solana wallet with roughly 0.0065 SOL for activation

## Tech stack

- Java 17, Swing UI
- Maven (shaded runnable JAR)
- SQLite for chain and local state
- SolanaJ for RPC and transaction building
- ZXing for QR codes, Jackson for JSON

## Build

```bash
mvn clean package
```

Run the resulting shaded JAR from `target/` with Java 17.

## Project layout

```text
hector.billium
  core/       configuration, crypto, JSON, paths, logging
  wallet/     keystore, Solana RPC, ATA, send/receive
  game/       genesis, block, proof-of-work, subsidy, difficulty
  p2p/        node, peer, protocol, sync, miner
  payout/     server, API, worker, database
  ui/         frame, navigation, panels, dialogs
```

## Security and trust model

- **Non-custodial.** Your private key is encrypted locally (PBKDF2-HMAC-SHA256, 210,000 iterations, AES-256-GCM) and never sent to a server. There is no custodial recovery, so back up your key.
- **Centralized settlement.** The game chain is peer-to-peer, but payouts depend on a known operator. Players trust the operator to honor valid claims. The payout service re-verifies every block itself and never pays a block twice.
- **Fairness limits.** Proof-of-work still favors higher hashpower. The two-thread default is a client-side cap that a modified client can bypass. Billium is accessible to low-spec hardware, but it is not hashpower-equal.
- **Not for high-value assets.** The game chain has a modest security budget.

See the white paper for the full threat model.

## Roadmap

A 14-week plan starting 26 October 2026: wallet and Solana integration, game core, P2P networking, mining and chain UI, payout service, onboarding, hardening, release candidate, then a buffer. Mainnet readiness is targeted for February 2027, or March 2027 with a security audit.

Planned later: pool or lottery fairness mode, multisig treasury, hardware wallet support, mobile companion.

## Documentation

- [White paper (PDF)](billium-whitepaper.pdf)

## Contributing

Issues and pull requests are welcome. For larger changes, please open an issue first to discuss the approach.

## License

To be decided.

## Disclaimer

This repository is a technical and product design project, not an offer of securities, a solicitation, or financial advice. BILLIUM is a game utility and reward token. Its value, liquidity, and availability are not guaranteed. Rewards depend on network conditions and operator policy. Comply with your local law before participating.
