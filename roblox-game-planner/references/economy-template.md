# Progression & Economy Template — Required Sections

## 1. Currencies
For each currency:
| Currency | How Earned | How Spent | Starting Amount |
|----------|-----------|-----------|-----------------|
| Coins | Collecting debris, selling, quests | Shop items, zone unlocks, rebirth | 0 |
| Gems | Daily rewards, achievements, Robux | Premium items, speed boosts | 0 |

## 2. XP / Leveling
- Formula: `XP_required(level) = [exact formula]`
- Table showing levels 1-50 with XP needed, cumulative XP, what unlocks
- How XP is earned (per action, amounts)

| Level | XP Required | Cumulative | Unlocks |
|-------|------------|------------|---------|
| 1 | 0 | 0 | Tutorial |
| 2 | 100 | 100 | Zone 1 |
| 5 | 500 | 1500 | Zone 2 |

## 3. Earning Rates
- Base earn rate at level 1
- How multipliers stack (additive or multiplicative)
- Earn rate formula: `coins_per_sec = base * level_mult * pet_mult * rebirth_mult * gamepass_mult`
- Table showing effective earn rate at key milestones

## 4. Item Pricing
- Formula: `price(tier) = base_price * growth^(tier-1)`
- Table for each shop category with all items and prices
- Zone unlock costs

## 5. Rebirth / Prestige
- Cost formula: `rebirth_cost(n) = [formula]`
- What resets on rebirth
- What you gain: multiplier formula
- Table: rebirth 1-20 with cost, multiplier, estimated time to reach

## 6. Pet Stats
- Stats by rarity: power, speed, luck
- Level scaling: `stat = base * (1 + 0.1 * level)`
- Evolution multipliers

## 7. Drop Rates
Per egg type:
| Egg | Common % | Uncommon % | Rare % | Epic % | Legendary % | Cost |
|-----|----------|------------|--------|--------|-------------|------|

## 8. Time-to-Milestone Estimates
| Milestone | Estimated Time | Assumes |
|-----------|---------------|---------|
| Reach Zone 2 | 30 min | No GamePass |
| First Rebirth | 3 hours | Active play |
| Legendary Pet | 10 hours | Average luck |

## 9. Sinks vs Faucets
- Where currency ENTERS the game (faucets)
- Where currency LEAVES the game (sinks)
- Balance analysis: is currency accumulating too fast? Too slow?

## 10. Pay vs Free Balance
- How much faster is a paying player? (2x? 5x? 10x?)
- What can ONLY be obtained by paying? (cosmetics only? or power too?)
