# Dispute Mechanism

## Overview

The dispute mechanism protects users from dishonest market resolution by creating economic incentives for honest behavior and allowing community intervention when needed.

## How It Works

### 1. Resolution Bond Requirement

When creating a market with Creator or Oracle resolution type, a **resolution bond** must be specified:

- **Minimum**: 1 SOL
- **Purpose**: Stake that resolver must lock when resolving
- **Incentive**: Creates financial risk for dishonest resolution

```rust
pub fn create_market(
    ctx: Context<CreateMarket>,
    question: String,
    end_time: i64,
    resolution_type: ResolutionType,
    oracle: Option<Pubkey>,
    resolution_bond: u64, // Must be >= 1 SOL
) -> Result<()>
```

### 2. Resolution Process

When a creator/oracle resolves a market:

1. **Resolver stakes bond**: Must transfer the resolution bond to market account
2. **Outcome is set**: Market outcome is recorded but NOT finalized
3. **24-hour dispute window opens**: Anyone can challenge the resolution
4. **Resolution time recorded**: Timestamp for dispute deadline calculation

```rust
pub fn resolve_market_creator(
    ctx: Context<ResolveMarketCreator>,
    outcome: bool,
) -> Result<()>
```

### 3. Dispute Window (24 Hours)

After resolution, there's a 24-hour period where:

- **Anyone can dispute** by matching the resolution bond
- **Disputer stakes equal amount** to the original bond
- **Market goes to community vote** if disputed
- **Original outcome is reset** and voting begins

```rust
pub fn dispute_resolution(
    ctx: Context<DisputeResolution>,
) -> Result<()>
```

**Requirements to Dispute:**
- Must be within 24 hours of resolution
- Must stake amount equal to resolution bond
- Market must not already be disputed
- Market must not be finalized

### 4. Community Vote (If Disputed)

When a dispute occurs:

1. **Voting opens**: Any user can vote YES or NO
2. **Minimum votes required**: 10 votes to finalize
3. **Majority wins**: Outcome determined by vote count
4. **Bond distribution**: Winner gets both bonds (2x stake)

```rust
pub fn vote_resolution(
    ctx: Context<VoteResolution>,
    vote_yes: bool,
) -> Result<()>

pub fn finalize_community_resolution(
    ctx: Context<FinalizeResolution>,
) -> Result<()>
```

### 5. Finalization

**If NOT Disputed:**
- After 24 hours, anyone can call `finalize_undisputed`
- Resolver gets bond back
- Market is finalized with original outcome

**If Disputed:**
- After minimum votes reached, call `finalize_community_resolution`
- Community vote determines final outcome
- Winner receives both bonds (2x their stake)
- Loser loses their entire bond

## Economic Incentives

### For Honest Resolvers

✅ **Low Risk**: No one will dispute correct resolution (they'd lose their stake)
✅ **Bond Returned**: Get full bond back after 24 hours
✅ **Reputation**: Build trust for future markets

### For Dishonest Resolvers

❌ **High Risk**: Likely to be disputed by community
❌ **Lose Bond**: Forfeit entire stake if community votes against you
❌ **Reputation Damage**: Lose credibility

### For Disputers

✅ **Profitable if Right**: Win 2x your stake (both bonds)
❌ **Costly if Wrong**: Lose entire stake
⚖️ **Risk/Reward**: Only dispute if confident in correctness

## Example Scenarios

### Scenario 1: Honest Resolution (No Dispute)

```
Market: "Will Bitcoin hit $100k by Dec 2024?"
Actual Outcome: YES
Resolution Bond: 5 SOL

1. Creator resolves as YES (stakes 5 SOL)
2. 24-hour window passes
3. No one disputes (outcome is correct)
4. Creator calls finalize_undisputed
5. Creator gets 5 SOL back
6. Market finalizes as YES
```

### Scenario 2: Dishonest Resolution (Disputed)

```
Market: "Will Bitcoin hit $100k by Dec 2024?"
Actual Outcome: YES
Resolution Bond: 5 SOL

1. Creator resolves as NO (stakes 5 SOL) ❌ DISHONEST
2. User notices error, disputes (stakes 5 SOL)
3. Community votes: 85% vote YES
4. Finalize with community vote
5. Disputer wins: Gets 10 SOL (5 + 5) ✅
6. Creator loses: Forfeits 5 SOL ❌
7. Market correctly finalizes as YES
```

### Scenario 3: False Dispute

```
Market: "Will Bitcoin hit $100k by Dec 2024?"
Actual Outcome: NO
Resolution Bond: 5 SOL

1. Creator resolves as NO (stakes 5 SOL) ✅ CORRECT
2. User incorrectly disputes (stakes 5 SOL)
3. Community votes: 90% vote NO
4. Finalize with community vote
5. Creator wins: Gets 10 SOL (5 + 5) ✅
6. Disputer loses: Forfeits 5 SOL ❌
7. Market correctly finalizes as NO
```

## Game Theory

The mechanism creates a **Nash Equilibrium** where honesty is optimal:

1. **Honest resolution** → No dispute → Bond returned → Best outcome
2. **Dishonest resolution** → Likely dispute → Lose bond → Worst outcome
3. **False dispute** → Community votes correctly → Lose bond → Bad outcome
4. **Valid dispute** → Community votes correctly → Win 2x → Good outcome

## Security Features

### Prevents Dishonest Resolution
- Financial penalty for incorrect resolution
- Community oversight and correction
- Profitable to catch dishonesty

### Prevents Spam Disputes
- Must stake equal bond amount
- Lose stake if wrong
- Economic cost to frivolous disputes

### Prevents Collusion
- Community vote requires minimum 10 votes
- Majority decision (hard to manipulate)
- Public and transparent

## Smart Contract Functions

### Create Market with Bond
```rust
create_market(
    question: String,
    end_time: i64,
    resolution_type: ResolutionType,
    oracle: Option<Pubkey>,
    resolution_bond: u64, // >= 1 SOL
)
```

### Resolve Market (Stakes Bond)
```rust
resolve_market_creator(outcome: bool)
resolve_market_oracle(outcome: bool)
```

### Dispute Resolution (Stakes Bond)
```rust
dispute_resolution()
```

### Vote on Disputed Market
```rust
vote_resolution(vote_yes: bool)
```

### Finalize Market
```rust
finalize_undisputed() // After 24h, no dispute
finalize_community_resolution() // After voting complete
```

## Configuration Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Minimum Bond | 1 SOL | High enough to deter dishonesty, low enough to be accessible |
| Dispute Window | 24 hours | Enough time for community to notice and react |
| Minimum Votes | 10 votes | Prevents manipulation, ensures community consensus |
| Bond Multiplier | 2x | Profitable to dispute dishonesty, costly to be wrong |

## Future Enhancements

### Reputation System
- Track resolver accuracy over time
- Display resolution history
- Weight votes by reputation

### Dynamic Bonds
- Scale bond with market size
- Higher stakes for larger markets
- Percentage-based bonds

### Multi-Signature Disputes
- Require multiple disputers
- Reduce false dispute risk
- Stronger signal of incorrect resolution

### Graduated Penalties
- First offense: Warning
- Repeat offenders: Higher penalties
- Permanent bans for serial dishonesty

### Automated Oracle Integration
- Chainlink price feeds
- Pyth Network data
- Switchboard oracles
- Reduce need for manual resolution

## Best Practices

### For Market Creators
1. Set appropriate bond amount (higher for controversial markets)
2. Only resolve when outcome is clear and verifiable
3. Wait for official confirmation before resolving
4. Document evidence for resolution

### For Disputers
1. Only dispute if confident outcome is wrong
2. Gather evidence before disputing
3. Consider bond amount vs. potential gain
4. Act quickly (within 24-hour window)

### For Voters
1. Research the market question thoroughly
2. Look for objective evidence
3. Vote honestly (your reputation matters)
4. Don't vote if uncertain

## Conclusion

The dispute mechanism creates a trustless system where:
- **Honesty is profitable** (keep your bond)
- **Dishonesty is costly** (lose your bond)
- **Community has final say** (decentralized arbitration)
- **Economic incentives align** with correct outcomes

This ensures market integrity without requiring trusted third parties.
