# Predixi Architecture

## System Overview

Predixi is a decentralized platform combining prediction markets (Polymarket-style) with token creation (Pump.fun-style) on Solana blockchain.

## Tech Stack

### Frontend
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Wallet**: Solana Wallet Adapter
- **State**: React Hooks

### Blockchain
- **Network**: Solana (Devnet/Mainnet)
- **Framework**: Anchor 0.29.0
- **Language**: Rust
- **Token Standard**: SPL Token

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                     Frontend (Next.js)                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Markets    │  │    Tokens    │  │  Portfolio   │  │
│  │   Interface  │  │   Interface  │  │   Tracking   │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│         │                  │                  │          │
│         └──────────────────┴──────────────────┘          │
│                            │                             │
│                   ┌────────▼────────┐                    │
│                   │ Wallet Adapter  │                    │
│                   └────────┬────────┘                    │
└────────────────────────────┼──────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Solana RPC     │
                    └────────┬────────┘
                             │
┌────────────────────────────▼──────────────────────────────┐
│                  Solana Blockchain                        │
│  ┌──────────────────────────────────────────────────┐    │
│  │         Predixi Smart Contract (Anchor)          │    │
│  │                                                   │    │
│  │  ┌─────────────────┐  ┌─────────────────┐       │    │
│  │  │ Market Module   │  │  Token Module   │       │    │
│  │  │                 │  │                 │       │    │
│  │  │ • Create        │  │ • Create        │       │    │
│  │  │ • Place Bet     │  │ • Buy/Sell      │       │    │
│  │  │ • Resolve       │  │ • Bonding Curve │       │    │
│  │  │ • Dispute       │  │ • Graduate      │       │    │
│  │  │ • Claim         │  │                 │       │    │
│  │  └─────────────────┘  └─────────────────┘       │    │
│  │                                                   │    │
│  │  ┌─────────────────────────────────────────┐    │    │
│  │  │         Account Structures              │    │    │
│  │  │  • Market  • Bet  • Vote  • Dispute    │    │    │
│  │  │  • TokenInfo  • Mint                    │    │    │
│  │  └─────────────────────────────────────────┘    │    │
│  └──────────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────────┘
```

## Smart Contract Architecture

### Program Structure

```
programs/predixi/src/
├── lib.rs              # Main program logic
├── state/              # Account structures (future)
├── instructions/       # Instruction handlers (future)
└── errors.rs           # Error definitions (future)
```

### Account Structures

#### Market Account
```rust
pub struct Market {
    pub creator: Pubkey,           // 32 bytes
    pub question: String,          // 256 bytes
    pub end_time: i64,             // 8 bytes
    pub resolution_type: ResolutionType, // 1 byte
    pub oracle: Option<Pubkey>,    // 33 bytes
    pub resolution_bond: u64,      // 8 bytes
    pub yes_pool: u64,             // 8 bytes
    pub no_pool: u64,              // 8 bytes
    pub resolved: bool,            // 1 byte
    pub outcome: Option<bool>,     // 1 byte
    pub resolution_time: i64,      // 8 bytes
    pub disputed: bool,            // 1 byte
    pub yes_votes: u32,            // 4 bytes
    pub no_votes: u32,             // 4 bytes
}
```

#### Bet Account
```rust
pub struct Bet {
    pub user: Pubkey,      // 32 bytes
    pub market: Pubkey,    // 32 bytes
    pub amount: u64,       // 8 bytes
    pub bet_yes: bool,     // 1 byte
}
```

#### TokenInfo Account
```rust
pub struct TokenInfo {
    pub creator: Pubkey,       // 32 bytes
    pub mint: Pubkey,          // 32 bytes
    pub name: String,          // 64 bytes
    pub symbol: String,        // 16 bytes
    pub uri: String,           // 256 bytes
    pub total_supply: u64,     // 8 bytes
    pub current_price: u64,    // 8 bytes
    pub market_cap: u64,       // 8 bytes
    pub graduated: bool,       // 1 byte
}
```

## Data Flow

### Market Creation Flow
```
User → Frontend → Wallet Sign → RPC → Smart Contract
                                          ↓
                                    Create Market Account
                                          ↓
                                    Initialize State
                                          ↓
                                    Return Market PDA
                                          ↓
Frontend ← RPC ← Event Emission ← Smart Contract
```

### Betting Flow
```
User → Select Outcome (YES/NO) → Enter Amount
                                      ↓
                              Wallet Sign Transaction
                                      ↓
                              Transfer SOL to Market
                                      ↓
                              Create Bet Account
                                      ↓
                              Update Pool (yes_pool/no_pool)
                                      ↓
                              Emit Event
                                      ↓
                              Update UI
```

### Resolution Flow (Creator)
```
Market Ends → Creator Resolves → Stakes Bond
                                      ↓
                              Set Outcome (not finalized)
                                      ↓
                              Start 24h Dispute Timer
                                      ↓
                    ┌─────────────────┴─────────────────┐
                    ↓                                   ↓
            No Dispute (24h pass)              Dispute Filed
                    ↓                                   ↓
            Finalize Market                    Community Vote
                    ↓                                   ↓
            Return Bond                         Majority Wins
                    ↓                                   ↓
            Enable Claims                       Winner Gets 2x Bond
```

### Token Creation Flow
```
User → Create Token → Upload Metadata
                            ↓
                    Wallet Sign Transaction
                            ↓
                    Create Mint Account
                            ↓
                    Create TokenInfo Account
                            ↓
                    Initialize Bonding Curve
                            ↓
                    Emit Creation Event
                            ↓
                    Display on Platform
```

## Frontend Architecture

### Component Structure

```
app/
├── layout.tsx              # Root layout with wallet provider
├── page.tsx                # Home page (markets/tokens tabs)
├── markets/
│   └── [id]/
│       └── page.tsx        # Individual market page
├── tokens/
│   └── [id]/
│       └── page.tsx        # Individual token page
└── globals.css             # Global styles

components/
├── WalletProvider.tsx      # Solana wallet integration
├── Header.tsx              # Navigation and wallet button
├── MarketCard.tsx          # Market display card
├── TokenCard.tsx           # Token display card
├── CreateMarketModal.tsx   # Market creation form
├── CreateTokenModal.tsx    # Token creation form
├── ResolveMarketModal.tsx  # Resolution interface
└── DisputeMarketModal.tsx  # Dispute interface
```

### State Management

Currently using React hooks for local state. Future considerations:
- Zustand for global state
- React Query for blockchain data caching
- WebSocket for real-time updates

## Security Architecture

### Smart Contract Security

1. **Access Control**
   - Only creator can resolve creator-type markets
   - Only oracle can resolve oracle-type markets
   - Only market participants can vote

2. **Economic Security**
   - Resolution bonds prevent dishonest resolution
   - Dispute mechanism allows community correction
   - Stake requirements prevent spam

3. **Time-Based Security**
   - Markets can't be resolved before end time
   - Dispute window is time-limited (24h)
   - Prevents premature finalization

4. **Overflow Protection**
   - Use u128 for calculations
   - Check for arithmetic overflow
   - Safe math operations

### Frontend Security

1. **Wallet Security**
   - User controls private keys
   - Transaction signing required
   - No private key exposure

2. **Input Validation**
   - Sanitize user inputs
   - Validate amounts and dates
   - Prevent injection attacks

3. **RPC Security**
   - Use trusted RPC endpoints
   - Rate limiting
   - Error handling

## Scalability Considerations

### Current Limitations
- On-chain storage for all data
- Linear search for markets
- No pagination

### Future Improvements

1. **Indexing**
   - Off-chain indexer (The Graph, Helius)
   - Fast queries and filtering
   - Historical data

2. **Caching**
   - Redis for hot data
   - CDN for static assets
   - Client-side caching

3. **Batching**
   - Batch multiple bets
   - Aggregate votes
   - Reduce transaction count

## Deployment Architecture

### Development
```
Local Machine → Solana Localnet → Next.js Dev Server
```

### Staging
```
GitHub → Vercel (Frontend) → Solana Devnet
```

### Production
```
GitHub → Vercel (Frontend) → Solana Mainnet
         ↓
    Custom Domain + CDN
```

## Integration Points

### Current
- Solana Wallet Adapter (Phantom, Solflare)
- Solana Web3.js
- Anchor Client

### Future
- **Oracles**: Chainlink, Pyth, Switchboard
- **DEX**: Raydium (for token graduation)
- **Storage**: Arweave, IPFS (for metadata)
- **Analytics**: Dune, Flipside
- **Notifications**: Push Protocol

## Monitoring & Observability

### Metrics to Track
- Transaction success rate
- Market creation volume
- Trading volume
- Resolution accuracy
- Dispute rate
- User retention

### Tools
- Solana Explorer
- Anchor events
- Frontend analytics (Vercel Analytics)
- Error tracking (Sentry)

## Cost Structure

### User Costs
- Market creation: ~0.01 SOL (rent) + resolution bond
- Place bet: ~0.000005 SOL (transaction fee)
- Token creation: ~0.02 SOL (rent + mint)
- Dispute: Resolution bond amount

### Platform Revenue (Future)
- Market creation fee: 0.1 SOL
- Trading fee: 1% of bet amount
- Token creation fee: 0.1 SOL
- Graduation fee: 1% of liquidity
