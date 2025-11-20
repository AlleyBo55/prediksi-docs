# Predixi Documentation

Welcome to the Predixi documentation. This guide will help you understand how the platform works.

## Table of Contents

1. [Architecture Overview](./ARCHITECTURE.md)
2. [Resolution Types](./RESOLUTION_TYPES.md)
3. [Dispute Mechanism](./DISPUTE_MECHANISM.md)
4. [Token Bonding Curve](./TOKEN_BONDING_CURVE.md)
5. [API Reference](./API_REFERENCE.md)

## Quick Start

### For Users

**Creating a Prediction Market:**
1. Connect your Solana wallet
2. Click "Create Market"
3. Enter your question
4. Choose resolution type (Creator/Oracle/Community)
5. Set resolution bond (if applicable)
6. Set end date
7. Add initial liquidity
8. Confirm transaction

**Trading on Markets:**
1. Browse available markets
2. Click on a market
3. Choose YES or NO
4. Enter bet amount
5. Confirm transaction
6. Track your position

**Launching a Token:**
1. Click "Launch Token"
2. Enter token details (name, symbol, description)
3. Upload token image
4. Confirm transaction (0.1 SOL fee)
5. Token launches with bonding curve

### For Developers

**Setup:**
```bash
# Install dependencies
npm install

# Build smart contract
anchor build

# Deploy to devnet
anchor deploy

# Start frontend
npm run dev
```

**Testing:**
```bash
# Run tests
anchor test

# Run specific test
anchor test -- --grep "market creation"
```

## Core Concepts

### Prediction Markets

Binary outcome markets where users bet on YES/NO outcomes. Uses automated market maker (AMM) for pricing.

**Key Features:**
- Binary outcomes (YES/NO)
- Pool-based pricing
- Time-limited markets
- Multiple resolution types
- Dispute mechanism

### Token Creation

Fair launch tokens with bonding curve mechanics, similar to Pump.fun.

**Key Features:**
- Bonding curve pricing
- No presale
- Automatic Raydium graduation
- Community-driven
- Meme-friendly

### Resolution System

Three ways to resolve markets:
1. **Creator Resolution**: Creator decides outcome (with dispute period)
2. **Oracle Resolution**: Automated via price feeds or data sources
3. **Community Resolution**: Community votes on outcome

### Dispute Mechanism

Protects against dishonest resolution:
- Resolver stakes bond
- 24-hour dispute window
- Anyone can challenge
- Community votes if disputed
- Winner gets 2x stake

## Security

### Smart Contract Security
- Access control on all functions
- Time-based restrictions
- Economic incentives for honesty
- Overflow protection
- Reentrancy guards

### Economic Security
- Resolution bonds
- Dispute stakes
- Minimum vote thresholds
- Slashing for dishonesty

## Documentation Files

### [ARCHITECTURE.md](./ARCHITECTURE.md)
Complete system architecture including:
- Tech stack
- Component structure
- Data flow
- Account structures
- Security architecture

### [RESOLUTION_TYPES.md](./RESOLUTION_TYPES.md)
Detailed explanation of resolution methods:
- Creator resolution
- Oracle resolution
- Community resolution
- Comparison and use cases

### [DISPUTE_MECHANISM.md](./DISPUTE_MECHANISM.md)
How disputes work:
- Resolution bonds
- Dispute process
- Economic incentives
- Game theory
- Example scenarios

### [TOKEN_BONDING_CURVE.md](./TOKEN_BONDING_CURVE.md)
Token creation mechanics:
- Bonding curve formula
- Pricing mechanism
- Graduation process
- Raydium integration

### [API_REFERENCE.md](./API_REFERENCE.md)
Smart contract API:
- Function signatures
- Parameters
- Return values
- Error codes
- Usage examples

## FAQ

### General

**Q: What is Predixi?**
A: A decentralized platform combining prediction markets (like Polymarket) with token creation (like Pump.fun) on Solana.

**Q: What blockchain does it use?**
A: Solana (Devnet for testing, Mainnet for production).

**Q: Do I need a wallet?**
A: Yes, you need a Solana wallet like Phantom or Solflare.

### Markets

**Q: How do prediction markets work?**
A: Users bet SOL on YES/NO outcomes. Winners split the total pool proportionally.

**Q: What happens if the creator lies?**
A: Anyone can dispute within 24 hours by staking the same bond. Community votes decide the outcome.

**Q: Can I create any type of market?**
A: Yes, but markets should have clear, verifiable outcomes for best results.

**Q: How long does resolution take?**
A: Creator/Oracle: Immediate + 24h dispute period. Community: Depends on voting participation.

### Tokens

**Q: How does the bonding curve work?**
A: Price starts at 0.0001 SOL and increases quadratically as more tokens are bought.

**Q: What happens at graduation?**
A: At 69k SOL market cap, liquidity is automatically deposited to Raydium DEX.

**Q: Can I sell tokens before graduation?**
A: Yes, you can sell back to the bonding curve at current price.

### Disputes

**Q: How much does it cost to dispute?**
A: You must stake the same amount as the resolution bond.

**Q: What if I dispute incorrectly?**
A: You lose your entire stake if community votes against you.

**Q: How long do I have to dispute?**
A: 24 hours from resolution time.

**Q: Who can vote on disputes?**
A: Anyone can vote. Minimum 10 votes required to finalize.

## Support

### Community
- Discord: [Coming Soon]
- Twitter: [Coming Soon]
- Telegram: [Coming Soon]

### Development
- GitHub: [Repository Link]
- Issues: [GitHub Issues]
- Discussions: [GitHub Discussions]

### Contact
- Email: support@predixi.io
- Documentation: docs.predixi.io

## Contributing

We welcome contributions! See [CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines.

### Areas to Contribute
- Smart contract improvements
- Frontend features
- Documentation
- Testing
- Bug fixes
- Oracle integrations

## License

MIT License - see [LICENSE](../LICENSE) for details.

## Roadmap

### Phase 1: MVP (Current)
- ✅ Basic prediction markets
- ✅ Token creation with bonding curve
- ✅ Three resolution types
- ✅ Dispute mechanism
- ✅ Frontend interface

### Phase 2: Enhancement
- [ ] Oracle integrations (Chainlink, Pyth)
- [ ] Advanced trading features
- [ ] Portfolio tracking
- [ ] Market analytics
- [ ] Mobile responsive design

### Phase 3: Scale
- [ ] Mainnet deployment
- [ ] Indexer for fast queries
- [ ] Advanced charting
- [ ] Social features
- [ ] Reputation system

### Phase 4: Ecosystem
- [ ] Mobile apps
- [ ] API for third parties
- [ ] Governance token
- [ ] DAO structure
- [ ] Cross-chain bridges

## Version History

### v0.1.0 (Current)
- Initial release
- Basic market creation
- Token bonding curve
- Resolution system
- Dispute mechanism

## Acknowledgments

Inspired by:
- Polymarket (prediction markets)
- Pump.fun (token creation)
- Augur (decentralized prediction markets)
- Uniswap (bonding curves)
