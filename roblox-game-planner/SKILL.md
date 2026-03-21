---
name: roblox-game-planner
description: Generates a complete game design folder from a single game idea. Use this skill when the user wants to plan, design, or document a Roblox game — from concept to full production-ready specs. Triggers on "plan a game", "game idea", "game design document", "GDD", "plan my game", "design a game", "game concept", or any request to create game documentation. Creates multiple interconnected documents covering every aspect of the game.
---

# Roblox Game Planner — Full Game Design from One Prompt

Takes a game idea and generates a complete folder of production-ready documents. Every document is detailed enough that a developer can build from it without asking questions.

---

## THE WORKFLOW

### STEP 1 — Interview

Get the core idea. Ask:
- What type of game? (simulator, tycoon, RPG, obby, fighting, etc.)
- One-sentence pitch? ("Pet simulator where you chase storms and collect debris")
- What makes it unique vs existing games?
- Target audience? (age range, casual vs hardcore)
- Art style? (cartoon, realistic, low-poly, pixel)
- How does the player make money/progress? (collect, fight, build, race?)

If the user already gave a detailed description, skip the questions and start generating.

### STEP 2 — Generate the Folder

Create ALL documents on the user's Desktop in a folder named after the game:

```
~/Desktop/[GameName]/
├── 01_GDD.md                          ← Master game design document
├── 02_GUI_Design.md                   ← Every screen described in detail
├── 03_Models_Assets.md                ← Every 3D model and asset needed
├── 04_Progression_Economy.md          ← All math, formulas, currencies, rates
├── 05_Monetization.md                 ← GamePasses, DevProducts, pricing
├── 06_Map_World.md                    ← Zones, layout, unlock order
├── 07_Update_Roadmap.md              ← Post-launch content plan
└── scripts/                           ← One file per game system
    ├── 00_Architecture_Overview.md    ← Script list, remotes, data structure
    ├── 01_Currency.md
    ├── 02_Inventory.md
    ├── 03_Shop.md
    ├── 04_Pets.md
    ├── 05_Egg_Hatching.md
    ├── 06_Rebirth.md
    ├── 07_Zones.md
    ├── 08_Daily_Rewards.md
    ├── 09_Codes.md
    ├── 10_Combat.md                   ← (if applicable)
    ├── 11_NPC_AI.md                   ← (if applicable)
    ├── 12_Leaderboard.md
    ├── 13_Trading.md                  ← (if applicable)
    ├── 14_Achievements.md             ← (if applicable)
    ├── 15_Tutorial.md
    └── 16_Data_Saving.md
```

Only create scripts/ files for systems that apply to the game. A tycoon doesn't need Combat. An obby doesn't need Pets. Pick what fits.

### STEP 3 — Document Quality Rules

Every document must:
- Be detailed enough to build from without questions
- Use specific numbers, not vague ("costs 500 coins" not "costs some coins")
- Cross-reference other documents ("see 04_Progression_Economy.md for pricing formula")
- Include tables where data is structured
- Include examples for every concept
- Use markdown with clear headers

---

## DOCUMENT TEMPLATES

Read `references/` files for the exact structure of each document. These tell the AI what sections and level of detail each doc needs.

### REFERENCE FILES
- `references/gdd-template.md` — Structure for the master GDD
- `references/gui-template.md` — Structure for GUI design doc
- `references/models-template.md` — Structure for assets doc
- `references/economy-template.md` — Structure for progression/economy
- `references/monetization-template.md` — Structure for monetization
- `references/map-template.md` — Structure for world layout
- `references/roadmap-template.md` — Structure for update plan
- `references/scripts-template.md` — Structure for each script system doc

---

## HARD RULES

1. **Generate ALL documents in one go.** Don't ask "which one should I start with?" — make them all.
2. **Use real numbers.** Prices, XP curves, drop rates, multipliers — all concrete. Never "TBD" or "to be decided."
3. **Cross-reference between docs.** Economy doc feeds into Shop script doc. GDD references Map doc for zones. Everything is connected.
4. **Only include relevant systems.** Don't add Combat to a farming simulator. Don't add Pets to a racing game. Match the game concept.
5. **Scripts folder = one file per system.** Each file describes: what scripts are needed, what remotes, what data, client vs server flow, and pseudocode or actual Luau code snippets.
6. **The GDD is the longest document (30+ sections).** It covers EVERYTHING about the game at a high level. Other docs go deep on specifics.
7. **Economy doc must have actual formulas.** Not "prices increase exponentially" — write `price = 100 * 1.5^(tier-1)` with a table showing levels 1-20.
8. **GUI doc must describe EVERY screen.** Not just "there's a shop screen" — list every element, position, color, font, animation.
9. **Map doc must describe EVERY zone.** Theme, biome, enemies, collectibles, NPCs, unlock requirement, music mood.
10. **After generating, tell the user:** "Your game design folder is ready at ~/Desktop/[GameName]/. Start with 01_GDD.md for the full overview."
