# Resolution Types

## Overview

Predixi supports three different resolution methods for prediction markets, each suited for different use cases.

## 1. Creator Resolution

### Description
The market creator is responsible for resolving the outcome after the market ends.

### How It Works
1. Market creator calls `resolve_market_creator` after end time
2. Stakes resolution bond (minimum 1 SOL)
3. 24-hour dispute window opens
4. If not disputed, market finalizes and bond is returned
5. If disputed, goes to community vote

### Use Cases
- Personal prediction markets
- Markets with trusted creators
- Small community markets
- Quick resolution needed

### Pros
✅ Fast resolution
✅ Simple process
✅ No external dependencies
✅ Creator has context

### Cons
❌ Requires trust in creator
❌ Potential conflict of interest
❌ Creator might have bet on outcome

### Security
- Resolution bond requirement
- 24-hour dispute period
- Community can override if dishonest

### Example
```
Market: "Will I finish my project by Friday?"
Creator: Project owner
Resolution: Creator knows if they finished
```

## 2. Oracle Resolution

### Description
A designated oracle address (could be a trusted third party, data feed, or multi-sig) resolves the market.

### How It Works
1. Oracle address specified at market creation
2. Only oracle can call `resolve_market_oracle`
3. Oracle resolves based on external data
4. Immediate finalization (no dispute period for oracles)

### Use Cases
- Objective outcomes (prices, scores, events)
- Integration with Chainlink, Pyth, Switchboard
- Sports betting markets
- Financial markets
- Weather predictions

### Pros
✅ Objective and verifiable
✅ Automated resolution possible
✅ No conflict of interest
✅ Trusted data sources

### Cons
❌ Requires oracle setup
❌ Oracle might fail or be unavailable
❌ Centralization risk
❌ Oracle fees

### Security
- Only designated oracle can resolve
- Oracle address set at creation (immutable)
- Can use multi-sig for oracle

### Example
```
Market: "Will BTC price be above $100k on Dec 31, 2024?"
Oracle: Pyth Network BTC/USD price feed
Resolution: Automated based on price at timestamp
```

### Oracle Integration Examples

#### Chainlink
```rust
// Pseudo-code for Chainlink integration
let price = chainlink_feed.get_latest_price();
let outcome = price >= target_price;
resolve_market_oracle(outcome);
```

#### Pyth Network
```rust
// Pseudo-code for Pyth integration
let price_account = pyth_price_account.get_price();
let outcome = price_account.price >= target_price;
resolve_market_oracle(outcome);
```

## 3. Community Resolution

### Description
The community votes on the outcome after the market ends. No single entity controls resolution.

### How It Works
1. Market ends
2. Anyone can call `vote_resolution` to cast vote
3. Minimum 10 votes required
4. Majority vote determines outcome
5. Anyone can call `finalize_community_resolution` after minimum votes

### Use Cases
- Subjective outcomes
- Controversial topics
- No clear oracle available
- Maximum decentralization desired
- Community-driven markets

### Pros
✅ Fully decentralized
✅ No single point of failure
✅ Community consensus
✅ No trust required

### Cons
❌ Slower resolution
❌ Requires voter participation
❌ Potential for manipulation (if low votes)
❌ Subjective outcomes

### Security
- Minimum vote threshold (10 votes)
- Majority decision
- Public and transparent
- Future: Stake-weighted voting

### Example
```
Market: "Was the new movie good?"
Resolution: Community votes based on reviews
Outcome: Majority opinion determines result
```

## Comparison Table

| Feature | Creator | Oracle | Community |
|---------|---------|--------|-----------|
| **Speed** | Fast | Instant | Slow |
| **Trust Required** | High | Medium | None |
| **Decentralization** | Low | Medium | High |
| **Automation** | No | Yes | No |
| **Dispute Period** | 24 hours | None | N/A |
| **Resolution Bond** | Required | Optional | Not Required |
| **Best For** | Personal markets | Objective data | Subjective outcomes |
| **Cost** | Bond only | Oracle fees | Free |

## Choosing the Right Type

### Choose Creator Resolution When:
- You trust the creator
- Quick resolution needed
- Small market with known participants
- Creator has clear information

### Choose Oracle Resolution When:
- Outcome is objective and verifiable
- External data source available
- Automation desired
- High-value market needs certainty

### Choose Community Resolution When:
- Outcome is subjective
- Maximum decentralization required
- No trusted oracle available
- Community engagement desired

## Resolution Flow Diagrams

### Creator Resolution Flow
```
Market Created (with bond amount)
    ↓
Market Ends
    ↓
Creator Resolves (stakes bond)
    ↓
24-Hour Dispute Window
    ↓
    ├─→ No Dispute → Finalize → Bond Returned
    └─→ Disputed → Community Vote → Winner Gets 2x Bond
```

### Oracle Resolution Flow
```
Market Created (with oracle address)
    ↓
Market Ends
    ↓
Oracle Resolves (automated or manual)
    ↓
Immediately Finalized
    ↓
Payouts Available
```

### Community Resolution Flow
```
Market Created
    ↓
Market Ends
    ↓
Community Voting Opens
    ↓
Users Cast Votes
    ↓
Minimum 10 Votes Reached
    ↓
Finalize with Majority Outcome
    ↓
Payouts Available
```

## Smart Contract Functions by Type

### Creator Resolution
```rust
// Create market
create_market(
    question: "Will X happen?",
    end_time: timestamp,
    resolution_type: ResolutionType::Creator,
    oracle: None,
    resolution_bond: 5_000_000_000, // 5 SOL
)

// Resolve (creator only)
resolve_market_creator(outcome: true)

// Dispute (anyone, within 24h)
dispute_resolution()

// Finalize (after 24h, no dispute)
finalize_undisputed()
```

### Oracle Resolution
```rust
// Create market
create_market(
    question: "Will BTC > $100k?",
    end_time: timestamp,
    resolution_type: ResolutionType::Oracle,
    oracle: Some(oracle_pubkey),
    resolution_bond: 0, // Optional for oracles
)

// Resolve (oracle only)
resolve_market_oracle(outcome: true)
```

### Community Resolution
```rust
// Create market
create_market(
    question: "Was the event successful?",
    end_time: timestamp,
    resolution_type: ResolutionType::Community,
    oracle: None,
    resolution_bond: 0, // Not required
)

// Vote (anyone)
vote_resolution(vote_yes: true)

// Finalize (after 10+ votes)
finalize_community_resolution()
```

## Best Practices

### For All Types
1. Write clear, unambiguous questions
2. Set appropriate end times
3. Specify resolution criteria upfront
4. Document evidence sources

### For Creator Resolution
1. Set high enough bond to ensure honesty
2. Resolve promptly after outcome is clear
3. Provide evidence for resolution
4. Be prepared for disputes

### For Oracle Resolution
1. Choose reliable oracle sources
2. Test oracle integration thoroughly
3. Have fallback resolution method
4. Monitor oracle uptime

### For Community Resolution
1. Encourage voter participation
2. Provide clear voting guidelines
3. Share relevant information
4. Consider incentivizing voters

## Future Enhancements

### Hybrid Resolution
- Start with oracle, fallback to community
- Multi-oracle consensus
- Weighted voting by stake

### Automated Resolution
- Smart contract conditions
- On-chain data verification
- Time-based automatic resolution

### Reputation-Weighted Voting
- Track voter accuracy
- Weight votes by reputation
- Reward good voters

### Escalation Mechanism
- Start with creator
- Escalate to oracle if disputed
- Final escalation to community
