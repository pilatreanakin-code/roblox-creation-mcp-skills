# Script System Document Template

Each file in the scripts/ folder follows this structure:

## 1. System Name
What this system does in one sentence.

## 2. Scripts Needed
| Script | Type | Location | Purpose |
|--------|------|----------|---------|
| CurrencyHandler | Script | ServerScriptService | Manages all currency operations |
| CurrencyClient | LocalScript | StarterPlayerScripts | Updates GUI displays |
| CurrencyConfig | ModuleScript | ReplicatedStorage/Modules | Shared constants |

## 3. Remotes
| Remote | Type | Direction | Purpose |
|--------|------|-----------|---------|
| PurchaseEvent | RemoteEvent | Client→Server | Request purchase |
| CurrencyUpdate | RemoteEvent | Server→Client | Notify balance change |

## 4. Data Structure
```lua
playerData.currency = {
    coins = 0,
    gems = 0,
}
```

## 5. Server Logic
What the server does, step by step. Include Luau code snippets.

## 6. Client Logic
What the client does. Include Luau code snippets.

## 7. Flow Diagram
```
Player clicks Buy → Client fires PurchaseEvent(itemId) → Server validates
→ Server deducts coins → Server grants item → Server fires CurrencyUpdate
→ Client updates GUI
```

## 8. Validation Rules
What the server checks before processing:
- Type check all arguments
- Item exists in catalog
- Player can afford
- Player doesn't already own
- Rate limit (cooldown)

## 9. Edge Cases
- What happens if player disconnects mid-purchase?
- What if DataStore fails?
- What if player has exactly enough coins?
