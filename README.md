# Roblox & Blender Skills for Claude Code

A collection of Claude Code skills for 3D modeling, UI design, and game development.

These skills turn Claude Code into a professional Roblox game developer and 3D artist. Each skill is a set of markdown files that Claude reads before building — giving it domain expertise, best practices, and complete code templates so it produces polished, production-ready output instead of generic boilerplate.

## Skills

### [Blender to Roblox](./blender-to-roblox)
Fully automated export pipeline from Blender to Roblox Studio. Exports FBX meshes, bakes ALL procedural textures to PNG, uploads via Open Cloud API, and places in Studio with correct scale and coordinate conversion (Z-up to Y-up). Handles multi-material objects, falls back to flat color on bake failure, and preserves world-space bounding boxes.

### [Blender Pro](./blender-pro)
Transforms the Blender MCP into a professional 3D artist. Conducts deep interviews before modeling so every asset is unique. Uses a blueprint system to plan geometry before building. Enforces proper modifier workflows (Subdivision Surface, Mirror, Solidify), node-based materials, and correct topology. Includes visual self-critique, smart assembly with natural overlap, multiple options mode for side-by-side comparison, and auto-save specs so models can be reloaded later.

### [Roblox GUI](./roblox-gui)
Creates polished, game-ready Roblox GUIs through the Roblox Studio MCP. Supports 4 style presets (cartoon/Pet Sim X, sleek/Blox Fruits, fantasy/RPG, sci-fi) or fully custom styling via style-config.md. Every GUI automatically gets: button-to-frame wiring, open/close scale animations, hover + click effects on all buttons, dim background, Escape key close, and a mandatory screenshot checklist before showing the user. Tabs actually swap content (not just color). Uses responsive Scale sizing everywhere. Includes creative design techniques (layered frames, rotated backgrounds) so every screen looks unique. Ships with complete LocalScript templates for shops, settings, notifications, sliders, and toggles.

### [Roblox Luau](./roblox-luau)
Professional Luau game scripting for Roblox via the Studio MCP. Enforces 15 hard rules that prevent every common AI scripting mistake — client/server separation, server-side validation on every action, proper DataStore with pcall + retry + auto-save + BindToClose, and anti-exploit patterns (rate limiting, type checking, distance validation). Includes complete, production-ready code templates for 11 game systems: currency, shop, inventory, pets, rebirth, zones, daily rewards, codes, combat, NPC AI, and leaderboards. Covers OOP module organization, performance best practices (debounce, cleanup, pooling), and MCP-specific patterns for creating scripts via execute_luau.

### [Roblox Game Planner](./roblox-game-planner)
Generates a complete game design folder from a single game idea — one prompt in, full production specs out. Creates 8 interconnected documents: Master GDD (game vision, core loop, target audience), GUI Design (every screen with colors, animations, layout), Models & Assets (every 3D model, icon, and effect needed), Progression & Economy (XP curves, pricing formulas, drop rates with real math), Monetization (GamePasses, DevProducts, pricing strategy), Map & World (zones with themes, enemies, unlock requirements), Update Roadmap (6-month content plan), and Scripts (architecture docs with code for every game system). Every document uses real numbers, formulas, and cross-references — not placeholder text.

## Compatibility

Built for **Claude Code**, but works with any AI tool supporting MCPs:
- **Cursor** — load SKILL.md as context
- **Windsurf** — add reference files to workspace
- **Cline / Continue** (VS Code) — paste as instructions

No AI? Follow the reference docs manually — standard Python (bpy), Luau, and Roblox API.

## Installation

```
git clone https://github.com/pilatreanakin-code/roblox-blender-skills.git
cp -r roblox-blender-skills/blender-to-roblox ~/.claude/skills/blender-to-roblox
cp -r roblox-blender-skills/blender-pro ~/.claude/skills/blender-pro
cp -r roblox-blender-skills/roblox-gui ~/.claude/skills/roblox-gui
cp -r roblox-blender-skills/roblox-luau ~/.claude/skills/roblox-luau
cp -r roblox-blender-skills/roblox-game-planner ~/.claude/skills/roblox-game-planner
```

## License
MIT
