# Prediction Market Mechanics

## Overview

Predixi is a decentralized prediction market platform that allows users to create and trade on binary outcome markets. The system combines automated market making (AMM) with a robust dispute resolution mechanism.

## Market Creation

### Requirements
- **Minimum Resolution Bond**: 100,000 IDR
- **Minimum Duration**: 1 hour from creation
- **Creation Fee**: Dynamic fee calculated by AMM service

### Market Types
1. **Binary Markets**: YES/NO outcomes
2. **Multi-outcome Markets**: Multiple possible outcomes

### Resolution Types
1. **Creator Resolution**: Only market creator can resolve
2. **Oracle Resolution**: External oracle provides resolution
3. **Community Resolution**: Community voting determines outcome

## Trading Mechanism

### Automated Market Maker (AMM)
- **Algorithm**: Constant Product Market Maker (x × y = k)
- **Initial Pools**: 1,000,000 IDR each for YES and NO
- **Dynamic Pricing**: Prices adjust automatically based on pool ratios
- **Always Liquid**: Never stops accepting bets, self-balancing
- **Fee Structure**: 2% trading fee (1.5% platform + 0.5% creator)

### Trading Requirements
- **Minimum Bet**: 10,000 IDR per transaction
- **Maximum Bet**: 10,000,000 IDR per transaction
- **Maximum Per User Per Market**: 10,000,000 IDR total
- **No Cancellation**: Bets are final once placed

### AMM-Based Pricing (Constant Product Market Maker)
- **Formula**: yes_pool × no_pool = k (constant)
- **Initial Pools**: 1,000,000 IDR each (YES and NO)
- **Dynamic Pricing**: Prices adjust automatically based on betting volume
- **Shares System**: You receive shares based on AMM calculation, not fixed amounts

### How Betting Works

#### Binary Markets (YES/NO)
- **Bet Amount**: You choose how much IDR to bet (10k-10M)
- **Shares Received**: Calculated by AMM based on current pools
- **No Short Selling**: You can only buy YES or NO positions
- **Example**: Bet 100,000 IDR on YES → Get ~89,254 shares (varies by pool state)

#### Multi-Outcome Markets
- **Simple Pool System**: Each outcome has its own pool
- **Equal Shares**: 1 IDR bet = 1 share (simpler than binary AMM)
- **Example**: Bet 100,000 IDR on outcome A → Get 100,000 shares of outcome A

### Fee Structure

#### Trading Fees (Charged When Betting)
- **Total Fee**: 2% of bet amount
- **Platform Fee**: 1.5% goes to platform
- **Creator Fee**: 0.5% goes to market creator
- **Example**: 100,000 IDR bet → 2,000 IDR fee → 98,000 IDR goes to pool

#### Claim Fees (Charged When Winning)
- **Claim Fee**: 1% of profit only (not principal)
- **Example**: Win 150,000 IDR from 100,000 IDR bet → Profit: 50,000 IDR → Fee: 500 IDR

### Payout Calculation

#### Binary Markets (AMM-Based)
- **Formula**: (your_shares ÷ winning_pool) × total_pool
- **Example**: You have 89,254 shares, YES wins, final pools: 2M YES + 1M NO
- **Calculation**: (89,254 ÷ 2,000,000) × 3,000,000 = 133,881 IDR
- **Minus Claim Fee**: 1% of profit = ~339 IDR fee

#### Multi-Outcome Markets (Simple)
- **Formula**: (your_shares ÷ winning_outcome_pool) × total_all_pools
- **Simpler**: If outcome A wins and you have shares in A, you get proportional payout

### Price Discovery

#### How Prices Change
- **AMM Effect**: Large bets cause bigger price movements (slippage)
- **Example**: 100k bet moves price from 50% to 54.7% in balanced market
- **Example**: Same 100k bet moves price from 70% to 72% in imbalanced market
- **Anti-Manipulation**: Exponentially expensive to move prices significantly

## Market Lifecycle

### 1. Active Phase
- Users can trade YES/NO positions
- Prices fluctuate based on trading activity
- Market remains open until end time

### 2. Ended Phase
- Trading stops at specified end time
- Market awaits resolution
- Only authorized parties can resolve based on resolution type

### 3. Resolved Phase
- Outcome is determined (YES or NO)
- 24-hour dispute period begins
- Winning positions can be claimed
- Losing positions become worthless

### 4. Disputed Phase (Optional)
- Any user (except creator) can dispute resolution
- Requires dispute bond equal to resolution bond
- Community voting period: 72 hours
- Minimum 10 votes required for finalization

### 5. Finalized Phase
- Final outcome confirmed
- All positions settled
- Bonds distributed to appropriate parties

## Dispute Resolution System

### Dispute Requirements
- Market must be in "resolved" status
- Within 24-hour dispute window
- Disputer cannot be market creator
- Must pay dispute bond (equal to resolution bond)

### Community Voting
- **Voting Period**: 72 hours
- **Minimum Votes**: 10 votes required
- **Decision Method**: Simple majority (>50%)
- **Voting Power**: Equal weight per user

### Bond Distribution
- **Undisputed Resolution**: Creator receives resolution bond back
- **Successful Dispute**: Disputer receives both bonds (creator + disputer)
- **Failed Dispute**: Creator receives both bonds (creator + disputer)

## Economic Incentives

### For Market Creators
- **Revenue**: Trading fees from market activity
- **Bond Recovery**: Resolution bond returned if undisputed
- **Risk**: Lose bond if disputed and community disagrees

### For Traders
- **Profit**: Buy low, sell high or hold to expiration
- **Risk**: Lose entire position if wrong outcome

### For Disputers
- **Reward**: Win both bonds if community agrees
- **Risk**: Lose dispute bond if community disagrees

### For Voters
- **Incentive**: Maintain market integrity
- **Responsibility**: Accurate outcome determination

## Security Features

### Time-based Controls
- Minimum market duration (1 hour)
- Fixed dispute period (24 hours)
- Fixed voting period (72 hours)

### Economic Security
- Resolution bonds prevent frivolous market creation
- Dispute bonds prevent spam disputes
- Community voting prevents manipulation

### Access Controls
- Creator-only resolution for creator markets
- Oracle-only resolution for oracle markets
- Community-only resolution for disputed markets

### Technical Safeguards
- Atomic balance operations
- Overflow protection
- State validation checks

## Market Categories

### Supported Categories
- **Sports**: Sports events and competitions
- **Politics**: Elections and political events
- **Crypto**: Cryptocurrency price predictions
- **Weather**: Weather-related outcomes
- **Entertainment**: Entertainment industry events
- **Economics**: Economic indicators and events

## Integration Points

### Oracle Integration
- **Chainlink**: For price feeds and external data
- **Pyth Network**: For real-time market data
- **Custom Oracles**: For specialized data sources

### Wallet Integration
- **Solana Wallet Adapter**: Multi-wallet support
- **Web3 Integration**: Seamless blockchain interaction
- **Mobile Wallets**: Mobile app compatibility

## Risk Management

### Market Risks
- **Liquidity Risk**: Low trading volume markets
- **Resolution Risk**: Disputed or unclear outcomes
- **Oracle Risk**: External data source failures

### User Risks
- **Trading Risk**: Market price volatility
- **Timing Risk**: Market expiration timing
- **Dispute Risk**: Community disagreement

### Platform Risks
- **Smart Contract Risk**: Code vulnerabilities
- **Governance Risk**: Dispute resolution accuracy
- **Regulatory Risk**: Compliance requirements

## Performance Metrics

### Market Health Indicators
- **Total Volume**: Cumulative trading volume
- **Active Traders**: Number of unique participants
- **Resolution Accuracy**: Percentage of undisputed resolutions
- **Dispute Rate**: Percentage of markets disputed

### Platform Metrics
- **Total Markets**: Number of created markets
- **Success Rate**: Percentage of successfully resolved markets
- **User Retention**: Active user growth
- **Revenue**: Platform fees collected

## Future Enhancements

### Planned Features
- **Advanced Order Types**: Limit orders, stop losses
- **Portfolio Tracking**: User position management
- **Social Features**: Market discussions and analysis
- **Mobile Application**: Native mobile experience

### Scaling Solutions
- **Layer 2 Integration**: Reduced transaction costs
- **Cross-chain Support**: Multi-blockchain compatibility
- **Advanced AMM**: Improved price discovery mechanisms