# Dynamic Outcomes - Why It's Disabled

## Summary
Adding outcomes to a market after creation is **permanently disabled** because it breaks the Automated Market Maker (AMM) mathematics.

## The Problem

### 1. AMM Math Breaks
The AMM pricing formula assumes a **fixed number of outcomes** at creation:
```
price_yes = yes_pool / (yes_pool + no_pool)
price_no = no_pool / (yes_pool + no_pool)
```

When you add a third outcome after bets are placed:
- The formula becomes invalid (designed for 2 outcomes only)
- Pool ratios lose meaning
- Pricing becomes unpredictable

### 2. Existing Bets Become Invalid
Example scenario:
```
Initial state:
- YES pool: 100 SOL
- NO pool: 100 SOL
- User A bet 50 SOL on YES (expects 50% of total if wins)

Creator adds "MAYBE" outcome:
- YES pool: 100 SOL
- NO pool: 100 SOL  
- MAYBE pool: 0 SOL
- User A's expected payout is now wrong
```

### 3. Liquidity Dilution
- New outcomes dilute existing liquidity unpredictably
- Winners' payouts become incorrect
- No fair way to redistribute existing bets

### 4. Manipulation Risk
Malicious creators could:
- Wait for bets to accumulate
- Add outcomes to dilute positions
- Manipulate final resolution

## The Solution

**All outcomes MUST be defined at market creation.**

This is how professional prediction markets work:
- ✅ **Polymarket**: All outcomes at creation
- ✅ **Kalshi**: All outcomes at creation
- ✅ **PredictIt**: All outcomes at creation

## Implementation

### Backend (Solana Smart Contract)
```rust
// DISABLED in backend/programs/predixi/src/lib.rs
// Lines ~350-370
/*
pub fn add_outcome(
    ctx: Context<AddOutcome>,
    outcome: String,
) -> Result<()> {
    // This would break AMM - DO NOT IMPLEMENT
    Ok(())
}
*/
```

### Backend API (Go)
```go
// DISABLED in api-backend/internal/handler/market_handler.go
// Lines ~200-220
/*
func (h *MarketHandler) AddOutcome(c *gin.Context) {
    c.JSON(http.StatusForbidden, gin.H{
        "error": "Adding outcomes after market creation is disabled - breaks AMM math",
    })
}
*/
```

### Frontend (Next.js)
```tsx
// DISABLED in frontend/app/market/[id]/page.tsx
// Comment added at line ~130
/* DISABLED: Adding outcomes after market creation
   Why: Breaks AMM math - pool ratios, pricing, and payouts become invalid
   Solution: All outcomes must be defined at market creation */
```

## Alternative Considered (Rejected)

### Option: Allow adding outcomes before first bet
**Rejected because:**
- Still creates confusion (when is it allowed?)
- Race condition (bet placed while creator adding outcome)
- Adds complexity for minimal benefit
- Better UX: Force creator to think through all outcomes upfront

## Best Practices

### For Market Creators
1. ✅ Define ALL possible outcomes at creation
2. ✅ Use "Other" as catch-all if needed
3. ✅ Keep outcome count reasonable (2-10)
4. ❌ Don't try to add outcomes later

### For Developers
1. ✅ Validate outcome count at creation (min 2, max 10)
2. ✅ Show clear error if creator tries to add outcomes
3. ✅ Document this limitation in UI
4. ❌ Never implement dynamic outcome addition

## References
- [AMM System Documentation](./AMM_SYSTEM.md)
- [Polymarket Documentation](https://docs.polymarket.com)
- [Kalshi Market Rules](https://kalshi.com/rules)
