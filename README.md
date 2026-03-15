# 🛡️ TicketShield — Anti-Scalping Ticket Protocol on Solana

> **Built at INNOVAT3 Hackathon — IIEST Shibpur, 2026**

> "BookMyShow is a shop assistant who *could* sell you a Coke for ₹500.  
> TicketShield is a vending machine that **literally cannot.**"

---

## The Problem

In January 2025, Coldplay tickets worth ₹6,500 sold on secondary markets for ₹32,000.  
BookMyShow's terms of service said scalping was prohibited. Scalpers didn't care.

The rule existed. It was written in a document. Nobody technically enforced it.

**TicketShield makes the rule part of the ticket itself.**  
The ticket cannot change hands above a certain price — not because we say so, but because the math says so.

---

## How It Works
```
Organiser deploys event
  → Sets face price + resale cap permanently on-chain. Nobody can edit them after.

Fan purchases ticket NFT  
  → Pays face price directly to organiser's wallet. Contract mints a unique token to fan's Phantom wallet.

Fan lists ticket into on-chain escrow
  → Ticket leaves fan's wallet into a contract-controlled lockbox. Frozen until sold or cancelled.

Contract validates resale price
  → Single on-chain check inside Solana runtime: price within cap? Cannot be bypassed.

Atomic completion or full revert
  → Valid price → instant swap of SOL + ticket. Too high → nothing moves, rejection logged on-chain forever.
```

---

## The One Line That Changes Everything
```rust
require!(
    listing.asking_price <= event.face_price
        .checked_mul(event.max_resale_bps as u64)
        .unwrap()
        .checked_div(10000)
        .unwrap(),
    ErrorCode::ResalePriceTooHigh
);
```

This runs on the Solana blockchain — not on our server, not in our frontend.  
It cannot be intercepted, bribed, or appealed. The code has no complaints department.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Blockchain | Solana (devnet) |
| Smart Contract | Anchor framework (Rust) |
| Frontend | React + TypeScript + Vite |
| Wallet | Phantom via @solana/wallet-adapter |
| Token Standard | SPL Token (Solana NFT) |

---

## Contract Instructions

| Instruction | What it does |
|---|---|
| `create_event` | Deploys event with face price + resale cap on-chain |
| `purchase_ticket` | Mints ticket NFT to buyer, transfers SOL to organiser |
| `list_ticket` | Moves ticket into escrow PDA, creates Listing account |
| `buy_listed_ticket` | Validates price, executes atomic swap or reverts |
| `cancel_listing` | Returns ticket from escrow back to seller's wallet |

---

## Project Structure
```
TicketShield/
├── anchor/                    # Solana smart contract (Rust + Anchor)
│   ├── programs/ticketshield/
│   │   └── src/
│   │       ├── instructions/  # create_event, purchase_ticket, list_ticket, etc.
│   │       ├── state/         # Event, Listing account structs
│   │       └── errors.rs      # Custom error codes
│   └── tests/                 # Anchor integration tests
│
└── frontend/                  # React frontend
    └── src/
        ├── pages/             # Home, EventPage, MyTickets, ResaleMarket, CreateEvent, Transparency
        ├── components/        # WalletConnect, TicketCard, TransactionToast
        ├── hooks/             # useProgram, useEventData
        ├── styles/            # Dark theme CSS design system
        └── utils/             # constants, mockData
```

---

## Pages

| Page | What it does |
|---|---|
| **Home** | Hero with problem statement + how it works |
| **Buy Tickets** | Fan purchases ticket at face price |
| **My Tickets** | View owned tickets, list for resale |
| **Resale Market** | Browse listings, buy resale tickets |
| **Organiser** | Deploy new event on-chain |
| **Transparency** | Public audit trail — every transaction, no login needed |

---

## Running Locally

### Prerequisites
- Rust — https://rustup.rs
- Solana CLI — https://docs.solana.com/cli/install-solana-cli-tools
- Anchor — `cargo install --git https://github.com/coral-xyz/anchor avm --locked && avm install 0.29.0`
- Node + Yarn — https://nodejs.org

### Smart Contract
```bash
cd anchor
anchor build
solana-keygen pubkey target/deploy/ticketshield-keypair.json
# Replace placeholder program ID in:
#   programs/ticketshield/src/lib.rs
#   Anchor.toml
#   ../frontend/src/utils/constants.ts
anchor build
cp target/idl/ticketshield.json ../frontend/src/idl/ticketshield.json
solana airdrop 2 --url devnet
anchor deploy --provider.cluster devnet
```

### Frontend
```bash
cd frontend
yarn install
yarn dev
# Open http://localhost:5173
```

---

## Demo Walkthrough

1. Connect Phantom wallet (devnet)
2. Go to **Organiser** → Create event (Coldplay India — Mumbai, 0.065 SOL, 110% resale cap)
3. Go to **Buy Tickets** → Purchase a ticket
4. Go to **My Tickets** → List for resale at 0.06 SOL ✅ (within cap — succeeds)
5. Go to **My Tickets** → List for resale at 0.32 SOL ❌ (exceeds cap — contract rejects)
6. Go to **Transparency** → See every transaction on-chain, no login needed

---

## Team

| Person | Contribution |
|---|---|
| **Person A** | Smart contract core — `create_event`, `purchase_ticket`, state structs, deployment |
| **Person B** | Resale enforcement — `list_ticket`, `buy_listed_ticket`, `cancel_listing`, tests |
| **Person C** | Frontend hooks + components — `useProgram`, `useEventData`, `EventPage`, `MyTickets`, dark UI |
| **Person D** | Home page, `ResaleMarket`, `Transparency`, pitch deck, demo script |

---

## Market Context

- India's live events market: **₹8,000 crore** and growing
- Coldplay India 2025: tickets resold at **5x face price** within hours of release
- Current solutions: terms of service documents that nobody enforces
- TicketShield: cryptographic enforcement that nobody can bypass

---

*Built with Anchor, React, and the belief that some problems are better solved with math than policy.*
