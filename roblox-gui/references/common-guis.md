# Common GUI Patterns — Based on Top Roblox Games

Research from Pet Simulator X, Blox Fruits, Adopt Me, Bee Swarm, Tower Defense Simulator, Murder Mystery 2, King Legacy, Anime Fighters, All Star Tower Defense. These are DESIGN PRINCIPLES — adapt to the user's game, don't copy literally.

---

## SHOP / STORE

### Standard Elements (90%+ of games have these)
- Title "Shop" in header
- Currency balance display in header (icon + number, always visible)
- Category tabs (horizontal top or vertical left sidebar)
- Item grid in ScrollingFrame with UIGridLayout (4-6 columns)
- Each item card: image, name, price with currency icon, rarity border color
- Buy button per card (changes to "Owned" after purchase)
- Close button (X) top-right

### Premium Features
- "Featured" or "Limited Time" section at top with special border/glow
- Search bar for large shops
- Filter by rarity/price
- Item detail panel on click (larger image, description, stats)
- Purchase animation: green flash, particle burst, sound

### Item Card Structure
```
ItemCard (Frame, ~80x100)
  RarityBorder (UIStroke, color by rarity)
  UICorner (8px)
  ItemImage (ImageLabel, centered, 60% of card)
  ItemName (TextLabel, bottom, small text)
  PriceRow (Frame, bottom)
    CurrencyIcon (ImageLabel, tiny)
    PriceText (TextLabel)
  BuyButton (TextButton, overlays card or below price)
    Changes to "Owned" (greyed out) after purchase
```

### Frame Hierarchy
```
ScreenGui > BackgroundDim (Frame, semi-transparent)
  > ShopFrame (main panel, centered, ~60% screen)
    > Header (Frame, top)
        Title (TextLabel "SHOP")
        CurrencyDisplay (Frame: icon + number)
        CloseButton (TextButton "X")
    > TabBar (Frame, below header)
        UIListLayout (Horizontal)
        Tab1, Tab2, Tab3... (TextButtons)
    > ContentArea (Frame, fills remaining space)
        ScrollingFrame (AutomaticCanvasSize Y)
          UIGridLayout (4-6 columns)
          UIPadding (12px all sides)
          ItemCard, ItemCard, ItemCard...
```

### Buy Flow
1. Player clicks Buy -> client fires RemoteEvent with item ID only (NOT price)
2. Server validates: item exists, player can afford, doesn't already own
3. Server deducts currency, grants item, fires client update
4. Client: button tweens to "Owned", green flash on card, coin sound plays
5. Currency counter animates down to new balance

### What Makes It Professional
- Consistent card sizes with proper padding
- Images use ScaleType.Fit (never distorted)
- Smooth tab switching (content fades/slides)
- Hover effect on cards (slight scale up or border glow)
- Sound on every interaction (click, purchase, error)
- Owned items clearly different (greyed, checkmark, no buy button)

---

## PET / COMPANION SCREEN

### Standard Elements
- Grid of pet cards in ScrollingFrame (3-4 columns)
- Each card: pet image, name, rarity border, equipped badge, power/level
- Equip/Unequip button (per pet or in detail panel)
- "Equip Best" button in header (auto-equips top N pets by total stats)
- Sort/Filter buttons (by rarity, level, power, equipped first)
- Equipped count indicator "3/3 Equipped"

### Pet Card Structure
```
PetCard (Frame, square)
  UICorner (8px)
  UIStroke (color = rarity color, thickness 2)
  PetImage (ImageLabel or ViewportFrame, centered, 70% of card)
  PetName (TextLabel, bottom)
  LevelBadge (TextLabel, top-left corner, "Lv.5")
  EquippedBadge (ImageLabel, top-right, star or checkmark, visible only if equipped)
  PowerLabel (TextLabel, bottom-right, stat number)
```

### Key Buttons
- **Equip/Unequip**: on each card or in detail panel. Green when equippable, blue when equipped
- **Equip Best**: prominent button in header, sorts by total power and equips top N
- **Delete/Release**: red button, requires confirmation popup. Multi-select mode: checkboxes on cards + "Delete Selected"
- **Sort dropdown**: By Rarity, By Power, By Level, By Name, Equipped First

### Detail Panel (opens on card click)
```
DetailPanel (Frame, slides in from right or popup)
  LargerPetImage (ViewportFrame or ImageLabel)
  PetName + Rarity label
  Stats list (Power, Speed, Luck with label + value each)
  Description text
  EquipButton (green/blue)
  DeleteButton (red, small)
  CloseButton
```

### Visual States
- Equipped: bright glowing UIStroke, star badge in corner, slightly brighter
- Unequipped: normal border, no badge
- Locked slots: dark background, padlock icon, "Unlock with GamePass"

### Collection/Index Tab
- Shows ALL possible pets including unowned
- Unowned: silhouette (dark, desaturated) with "???" name
- Owned: full color with stats
- Completion counter "47/120 Discovered"

---

## EGG HATCHING

### Standard Elements
- Row or grid of egg types
- Each egg: 3D rotating model (ViewportFrame) or polished 2D image
- Name and cost below each egg
- "Hatch x1" and "Hatch x3" buttons with cost
- Drop rates/odds panel (expandable, shows pet chances as %)
- Auto-hatch toggle

### Egg Card Structure
```
EggCard (Frame)
  EggDisplay (ViewportFrame with rotating model, or ImageLabel)
  EggName (TextLabel)
  CostLabel (TextLabel with currency icon)
  HatchButton ("Hatch x1 - 100 coins")
  HatchMultiButton ("Hatch x3 - 270 coins")
  OddsButton ("View Odds")
```

### Hatch Animation Sequence
1. Egg appears centered on screen (large)
2. Egg shakes (rotation tween left/right, 3-4 times, increasing intensity)
3. Egg cracks (overlay crack sprite or swap image)
4. Burst of particles (stars, sparkles, colored by rarity)
5. Pet revealed with glow effect (scale from 0 to full, Back easing)
6. Rarity text appears above pet ("LEGENDARY!" gold color, shake effect)
7. Sound: crack, pop, fanfare for rare+

### Triple Hatch
- 3 eggs side by side, each hatches sequentially or simultaneously
- Results shown in a row: 3 pet cards with rarity borders
- "Keep All" button

---

## INVENTORY / BACKPACK

### Standard Elements
- Grid of item slots (UIGridLayout, 4-6 columns)
- Each slot: item icon, stack count (bottom-right "x5"), rarity border
- Equipped items: green overlay or checkmark badge
- Empty slots: dark frame with faint border
- Category tabs (Weapons, Armor, Consumables, Materials)
- Sort buttons (By Rarity, By Name, By Level)
- Detail panel on item click

### Item Slot Structure
```
ItemSlot (Frame, square, ~70x70)
  UICorner (6px)
  UIStroke (rarity color or grey for empty)
  ItemIcon (ImageLabel, centered, ScaleType.Fit)
  StackCount (TextLabel, bottom-right, "x5", visible if count > 1)
  EquippedBadge (top-right, green checkmark, visible if equipped)
```

### Item Actions (in detail panel)
- Equip (green button)
- Use (blue button)
- Drop/Delete (red button, with confirmation)
- Double-click shortcut: equip instantly

### Equipped Indication
- Glowing yellow/green UIStroke (thicker)
- Small "E" badge or checkmark in corner
- Sorted to front when "Equipped First" sort active

---

## MAIN MENU / LOBBY

### Standard Buttons
Play (biggest, primary color), Shop, Inventory, Settings, Codes, Daily Rewards, Credits

### Layout Options
- Vertical sidebar left: icon buttons stacked
- Center column: large buttons stacked, logo above
- Bottom bar: horizontal icons (mobile-friendly)

### Background
- Best: blurred 3D game scene or animated ViewportFrame
- Okay: themed static image with color overlay
- Bad: plain solid color

### Frame Hierarchy
```
ScreenGui > BackgroundImage (full screen)
  > LogoFrame (top center, animated scale on load)
  > ButtonContainer (centered or sidebar)
      UIListLayout (Vertical, padding 12)
      PlayButton (biggest, primary color, pulse glow)
      ShopButton
      InventoryButton
      SettingsButton
      CodesButton
  > VersionLabel (bottom-left, tiny)
  > SocialLinks (bottom-right, small icons)
```

---

## HUD (Heads-Up Display)

### Standard Layout
- **Top-left**: Health bar + secondary stat (stamina/energy)
- **Top-right**: Currency display(s) (coins, gems)
- **Top-center**: Boss health bar (conditional)
- **Bottom-center**: Hotbar/toolbar (ability slots, 6-8 slots)
- **Bottom-right** (mobile): Jump + ability buttons

### Health Bar
```
HealthBarBG (Frame, dark, full width)
  UICorner
  HealthBarFill (Frame, colored, width = Health/MaxHealth as Scale)
    UICorner (same radius)
  HealthText (TextLabel, centered, "75/100")

Behavior:
  On damage: tween Fill size, flash red, optional shake
  On heal: tween Fill, flash green
  Low health (<25%): pulse red glow loop
```

### Currency Display
- Icon (24x24) + Number (GothamBold, 18)
- Animate number counting on change
- Green flash on increase, red on decrease

### Damage Numbers
- BillboardGui on humanoid, TextLabel with number
- Tween: rise up Y, fade out, slight random X drift
- Color by type: white normal, yellow crit, green heal

---

## SETTINGS

### Standard Options
- Music Volume (slider)
- SFX Volume (slider)
- Graphics Quality (Low/Med/High buttons)
- Notifications (toggle)
- Reset Progress (button + confirmation)

### Slider
```
SliderTrack (Frame, thin horizontal, grey)
  SliderFill (Frame, colored, width = %)
  SliderKnob (circle on fill edge, draggable)
  ValueLabel ("75%")
```

### Toggle
```
ToggleBG (pill shape, grey OFF / green ON)
  UICorner (full round)
  ToggleKnob (circle, left OFF / right ON)
Tween position + color on click
```

---

## CODES

### Structure (simple)
```
CodesFrame (small centered popup)
  Title ("Enter Code")
  InputField (TextBox, placeholder "Code here...")
  RedeemButton ("Redeem")
  ResultLabel (hidden until attempt: green "Redeemed!" or red "Invalid")
  SocialLinks ("Follow us for codes!")
```

---

## DAILY REWARDS

### Structure
```
DailyFrame (centered popup, auto-shows on join)
  Title "Daily Rewards"
  DayGrid (7 columns)
    DayCard (each):
      DayNumber ("Day 1")
      RewardIcon + Amount
      Status: checkmark (claimed) / glow (available) / lock (future)
  StreakCounter ("Streak: 5 days")
  ClaimButton (active on available day only)
  TimerLabel ("Next in: 14h 23m")
```

### Visual States
- Claimed: green checkmark, faded
- Available (today): glowing border, pulse animation, CLAIM active
- Locked: grey, padlock, no interaction
- Day 7: gold border, bigger, premium reward

---

## TRADING

### Structure
```
TradeFrame (split in half)
  YourSide (left): your name, your item grid, add button, accept button
  TheirSide (right): their name, their items, their status
  ConfirmButton (only when both accepted)
  CancelButton
```

---

## LEADERBOARD

### Structure
```
LeaderboardFrame
  Tabs (Daily / Weekly / All-Time)
  ScrollingFrame with rows
    Each row: Rank, Avatar, Name, StatValue
    Top 3: gold/silver/bronze background
    Your row: highlighted, pinned at bottom
```

---

## NOTIFICATIONS / TOASTS

### Types
- Success (green): purchase, code redeemed
- Error (red): not enough coins, invalid
- Info (blue): update, zone unlocked
- Achievement (gold): milestone reached
- Warning (yellow): inventory full

### Behavior
- Slide in from top-right, auto-dismiss 3-5 seconds
- Queue: max 3-4 visible, new pushes old up
- Each: colored accent stripe left, icon, message text

---

## CONFIRMATION POPUPS

```
DimBackground (full screen, black 50%)
  PopupFrame (centered, ~350x200)
    UICorner (12px)
    Title ("Are you sure?")
    Description ("Buy X for 5,000 coins?")
    CancelButton (grey, left)
    ConfirmButton (colored, right: green=buy, red=delete)
    Animation: scale pop 0 to full (Back easing)
```

---

## LOADING SCREEN

```
LoadingGui (full screen, in ReplicatedFirst)
  BackgroundImage (game art)
  Logo (centered, large)
  ProgressBar (bottom center)
  TipLabel (bottom, cycles every 3-5s)
```
```lua
game:GetService("ReplicatedFirst"):RemoveDefaultLoadingScreen()
-- Fade out when loaded
```

---

## UNIVERSAL PATTERNS

### Icons: flat 2D, solid colors, consistent stroke weight. 24x24 inline, 48x48 cards, 64x64 featured.

### Screen Transitions: open = fade + scale pop (Back, 0.3s). Close = scale down + fade (Quint In, 0.2s). Tabs = crossfade content.

### Multiple GUIs: only one major screen open at a time. Close current before opening new. HUD stays behind dim.

### Sounds: click = pop. Purchase = coin. Error = buzz. Achievement = fanfare. Open = whoosh. Close = reverse whoosh.

### Responsive: ALL Scale sizing. AnchorPoint for centering. Buttons min 48x48 for mobile. Test phone + PC + Xbox.

### Professional vs Amateur
| Professional | Amateur |
|---|---|
| Consistent padding everywhere | Uneven gaps, touching edges |
| UICorner on everything | Sharp corners |
| Hover + click animations | Static buttons |
| Open/close tweens | Instant appear/disappear |
| Custom fonts (GothamBold) | Default font |
| Consistent rarity colors | Random colors |
| Sound on every action | Silent |
| Server validates all | Client trusted |
| Works on mobile + PC | Broken on mobile |
| ScaleType.Fit on images | Stretched images |
