# Blender Pro — Professional 3D Modeling Skill

Transforms the Blender MCP into a professional 3D artist. Instead of placing basic primitives, the AI interviews you, designs a detailed blueprint, builds with proper modifiers and node materials, self-critiques visually, and iterates.

## Features

- Deep interview — detailed questions so every model is unique
- Blueprint system — AI writes a design doc before building
- Proper techniques — modifiers, node materials, correct topology
- Visual self-critique — evaluates shapes, textures, connections before showing you
- Smart assembly — parts overlap naturally, no floating gaps
- Multiple options mode — "show me 3 versions" builds them side by side
- Auto-save to project — saves full specs (parts, materials, modifiers, palettes) to a project folder automatically
- Reload models — "rebuild the golden sword" recreates from saved specs

## Requirements

- Claude Code with Blender MCP
- Blender open

## Installation

Copy the blender-pro folder to ~/.claude/skills/blender-pro/

## Usage

Ask Claude Code to make anything:

- "Make me a wooden sword"
- "Create a cute cat pet, stylized smooth"
- "Build a medieval house, show me 3 options"
- "Design a fantasy axe for my Roblox game"

The AI asks for details before building. After approval it auto-saves the model specs.

## File Structure

```
blender-pro/
├── SKILL.md
└── references/
    ├── objects/
    │   ├── primitives.md
    │   ├── edit-operations.md
    │   ├── modifiers.md
    │   ├── model-patterns.md
    │   └── roblox-optimization.md
    └── textures/
        ├── shader-nodes.md
        ├── material-recipes.md
        └── uv-mapping.md
```

## License

MIT
