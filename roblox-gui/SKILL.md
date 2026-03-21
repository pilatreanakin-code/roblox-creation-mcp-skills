---
name: roblox-gui
description: Creates professional, polished Roblox GUIs via the Roblox Studio MCP. Use this skill whenever the user asks to create, design, or build any UI/GUI in Roblox Studio — health bars, shops, menus, HUDs, inventory screens, settings panels, notification systems, buttons, popups, or any user interface element. Triggers on any mention of "GUI", "UI", "menu", "HUD", "health bar", "shop GUI", "inventory", "settings menu", "popup", "notification", "button", "screen", "interface", or any Roblox GUI request. ALWAYS use this skill for ANY Roblox GUI task.
---

# Roblox GUI — Professional UI Design Skill

---

## ABSOLUTE RULES (apply to EVERY GUI, no exceptions)

### 1. Responsive Scaling
EVERY element uses Scale sizing. Never pixel Offset for Size or Position.
```lua
-- CORRECT:
frame.Size = UDim2.new(0.5, 0, 0.6, 0)
-- WRONG:
frame.Size = UDim2.new(0, 500, 0, 400)
```
**Offset exceptions ONLY for:** UICorner radius, UIPadding, UIStroke thickness, tiny icons (24x24).
**Square elements:** Always add UIAspectRatioConstraint (AspectRatio = 1).
**Centering:** Always AnchorPoint = Vector2.new(0.5, 0.5) + Position = UDim2.new(0.5, 0, 0.5, 0).

### 2. IgnoreGuiInset
The ScreenGui AND any full-screen overlay (dim background) must have `IgnoreGuiInset = true` so they cover the ENTIRE screen including under the top bar.
```lua
screenGui.IgnoreGuiInset = true
```

### 3. Alignment
ScrollingFrames, content areas, and grids must be aligned flush with the parent frame edges (respecting UIPadding). The bottom of a ScrollingFrame should align with the bottom of its parent — never float above it.

### 4. The User's Prompt Overrides Everything
If the user specifies exact buttons, categories, items, colors, or layout in their prompt — use EXACTLY what they said. Do NOT add, remove, or change based on `references/common-guis.md`. The common-guis reference is ONLY for when the user gives vague requests like "make me a shop" without specifics. If they say "4 tabs: Vehicles, Gear, Backpack, Special" — make exactly those 4, not 5, not 3.

---

## STEP 0 — Read Style Config

Read `style-config.md` first. It has the user's visual preferences and fonts.

**Default fonts from config:** FredokaOne for titles, GothamBold for body/buttons. Use these unless the user overrides.

**If a preset is set:**

**`cartoon`** (Pet Simulator X style):
- Backgrounds: WHITE (RGB 255,255,255)
- UIStroke: BLACK, thickness 3-4px, on EVERYTHING (frames, buttons, cards)
- UICorner: varies by size — large frames 16-20px, medium cards 10-12px, small buttons 6-8px
- Title: VERY BIG FredokaOne (size 36-48), positioned ABOVE the frame, WHITE text with BLACK TextStrokeColor3 (TextStrokeTransparency = 0)
- Close button: bright red/pink (RGB 220,50,70) ROUNDED SQUARE (UICorner 8px, NOT circle), thick black border, white bold X, overlapping top-right
- Action buttons: NEON GREEN (RGB 80,220,50), dark green border (RGB 40,140,20), white bold text, UICorner 8px
- Cards: white background, thick black border (2-3px), UICorner 10-12px
- Nav buttons: SQUARE (never circle), UICorner 10-12px, thick black border, icon inside, label below
- Currency: gold/yellow numbers (RGB 255,200,50)
- Overall: bright, white, bubbly, cartoon, thick outlines everywhere
- Use creative techniques: layered frames, rotated background frames, varied tab styles — make every screen look unique while staying on-style

**`sleek`**: dark navy bg, thin grey stroke 1.5px, corners 8-10px, Gotham font, blue accents
**`fantasy`**: tan/parchment bg, gold stroke 2px, corners 6-8px, warm palette
**`scifi`**: very dark bg, cyan glow stroke 2px, sharp corners 4px, neon accents

**If empty:** Ask the user. Once chosen, apply to EVERYTHING consistently.

---

## STEP 1 — Interview

What are they building? Get specifics. Confirm before building.

---

## STEP 2 — Build with Auto-Linking and Auto-Animation

### AUTO-LINKING (MANDATORY)
When the user asks for a button AND a frame — ALWAYS wire them together automatically:
- Button click → opens frame (with animation)
- Close button (X) → closes frame (with animation)
- Dim background click → closes frame
- Escape key → closes frame
- Only one major screen open at a time

The user should NEVER have to ask for this. It's implied.

### AUTO-ANIMATION (MANDATORY)
Every GUI gets animations automatically. The user doesn't ask — you just add them:

**Every screen open:** Scale pop animation
```lua
local TweenService = game:GetService("TweenService")

local function openScreen(frame, dim, targetSize)
    frame.Visible = true
    frame.AnchorPoint = Vector2.new(0.5, 0.5)
    frame.Position = UDim2.new(0.5, 0, 0.5, 0)
    frame.Size = UDim2.new(0, 0, 0, 0)
    if dim then
        dim.Visible = true
        TweenService:Create(dim, TweenInfo.new(0.25), {BackgroundTransparency = 0.5}):Play()
    end
    TweenService:Create(frame, TweenInfo.new(0.35, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
        Size = targetSize
    }):Play()
end
```

**Every screen close:** Scale down + dim fade
```lua
local function closeScreen(frame, dim, callback)
    local tween = TweenService:Create(frame, TweenInfo.new(0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
        Size = UDim2.new(0, 0, 0, 0)
    })
    if dim then
        TweenService:Create(dim, TweenInfo.new(0.2), {BackgroundTransparency = 1}):Play()
    end
    tween:Play()
    tween.Completed:Connect(function()
        frame.Visible = false
        if dim then dim.Visible = false end
        if callback then callback() end
    end)
end
```

**Every button hover:** Scale up 5% + optional color lighten
```lua
local function setupHover(btn)
    local origSize = btn.Size
    local origColor = btn.BackgroundColor3
    local hoverColor = Color3.new(
        math.min(origColor.R * 1.15, 1),
        math.min(origColor.G * 1.15, 1),
        math.min(origColor.B * 1.15, 1)
    )
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {
            Size = UDim2.new(origSize.X.Scale * 1.05, 0, origSize.Y.Scale * 1.05, 0),
            BackgroundColor3 = hoverColor
        }):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {
            Size = origSize,
            BackgroundColor3 = origColor
        }):Play()
    end)
end
```

**Every button click press:** Shrink to 93% then bounce back
```lua
local function setupClick(btn)
    local origSize = btn.Size
    btn.MouseButton1Down:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.08), {
            Size = UDim2.new(origSize.X.Scale * 0.93, 0, origSize.Y.Scale * 0.93, 0)
        }):Play()
    end)
    btn.MouseButton1Up:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.12, Enum.EasingStyle.Back), {
            Size = origSize
        }):Play()
    end)
end
```

### DIM BACKGROUND
Always create behind popups/screens:
```lua
local dim = Instance.new("TextButton")
dim.Name = "ScreenDim"
dim.Size = UDim2.new(1, 0, 1, 0)
dim.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
dim.BackgroundTransparency = 1
dim.Text = ""
dim.AutoButtonColor = false
dim.ZIndex = 5
dim.Visible = false
dim.Parent = screenGui
-- ScreenGui must have IgnoreGuiInset = true
```

### ESCAPE KEY CLOSE
```lua
game:GetService("UserInputService").InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.Escape then
        -- close whichever screen is open
    end
end)
```

### MULTIPLE SCREENS MANAGER
```lua
local currentOpen = nil
local function openOnly(frameName)
    if currentOpen then closeScreen(currentOpen, dim) end
    currentOpen = frames[frameName]
    openScreen(currentOpen, dim, targetSizes[frameName])
end
```

---

## STEP 3 — Screenshot Check (MANDATORY)

After building, play-test and take a screenshot. Open `references/checklist.md` and run every single check. If ANY fails, fix it, screenshot again, repeat. Only show the user when everything passes.

---

## HARD RULES SUMMARY

1. ALL sizing in Scale. UIAspectRatio on squares.
2. IgnoreGuiInset = true on ScreenGui and dim overlay.
3. **ScrollingFrame flush with parent bottom edge — NO gap.** If inside a colored inner frame, the inner frame also reaches the bottom of the outer frame with NO gap.
4. UICorner on EVERY frame and button. **Smaller elements = less rounded** (see styling.md). UIPadding inside EVERY container.
5. UIStroke instead of BorderSizePixel (always set BorderSizePixel = 0).
6. Hover + click animation on EVERY button. Automatically. User doesn't ask.
7. Open/close animation on EVERY screen and ALL companion elements (title, decor frame, close button). Save original properties at startup to prevent transparency bugs on reopen.
8. Button + frame = auto-wired. Automatically. User doesn't ask.
9. Only one major screen open at a time.
10. Escape key and dim-click close any open GUI.
11. Fonts: FredokaOne for titles, GothamBold for body (unless config overrides).
12. **Titles min TextSize 56**, white + black stroke. Biggest text on screen.
13. **Nav/sidebar buttons must be clearly visible** — minimum 0.06 Scale width.
14. **Buttons are ROUNDED SQUARES, never circles.** Close button = UICorner 8px, NOT UDim.new(1,0).
15. **Label below nav button: 2-4px gap max.** Must nearly touch the button.
16. User's prompt overrides common-guis.md. If they specify exact buttons/tabs/items, use exactly those.
17. Consistent style across all GUIs in the game.
18. **Decorative technique: pick ONE per screen** — either colored inner frame OR rotated background frame, never both (unless user asks). Choose randomly.
19. **Tabs MUST swap content.** Tabs that only change color are BROKEN.
20. **Card spacing: CellPadding 0.02 minimum.** Cards never touch. UIPadding 10-12px on ScrollingFrame.

---

## REFERENCE FILES

- `style-config.md` — User's visual style (READ FIRST ALWAYS)
- `references/checklist.md` — Screenshot checklist. Run after EVERY build. (READ AFTER BUILDING)
- `references/code-patterns.md` — COMPLETE working LocalScript templates (READ SECOND — NEVER skip this)
- `references/gui-elements.md` — Every GUI instance with properties
- `references/styling.md` — UICorner, UIStroke, UIPadding, fonts, colors
- `references/layouts.md` — UIListLayout, UIGridLayout, responsive patterns
- `references/common-guis.md` — Design patterns from top games (use ONLY for vague requests)
- `references/animations.md` — TweenService, hover effects, transitions
- `references/client-server.md` — RemoteEvents, validation, data binding

---

## THE MOST IMPORTANT RULE

**Every GUI MUST include a working LocalScript.** If you create frames, buttons, and labels but no script — the GUI is BROKEN and USELESS. The LocalScript is not optional. It is the FIRST thing you plan and the LAST thing you verify works.

Read `references/code-patterns.md` for exact templates. Pattern 1 (Button + Frame + LocalScript) covers 90% of requests. Follow the template structure exactly — adapt names and values, keep the structure.
