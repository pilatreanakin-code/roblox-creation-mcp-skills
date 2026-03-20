# Blender Pro — Professional 3D Modeling Skill

A Claude Code skill that transforms the Blender MCP from a primitive-placer into a professional 3D artist. Instead of creating basic shapes and calling it done, the AI interviews you about what you want, designs a detailed blueprint, builds in stages with proper modifiers and node-based materials, self-critiques the result visually, and iterates until it looks right.

## Features

- **Deep interview** — asks detailed questions about style, shape, materials, proportions so every model is unique
- **Blueprint system** — AI writes itself a hyper-specific design document before touching Blender
- **Proper techniques** — uses modifiers (subdivision, mirror, bevel, boolean), node-based materials, correct topology
- **Self-critique** — evaluates its own work for shape quality, material accuracy, and assembly before showing you
- **Smart assembly** — parts overlap naturally like real objects, no floating gaps
- **Multiple options mode** — ask for N variations and the AI builds them side by side for you to pick
- **Complete API knowledge** — every primitive, edit operation, modifier, and shader node documented

## Requirements

- Claude Code with Blender MCP connected
- Blender open with a scene

## Installation

Place the blender-pro folder at ~/.claude/skills/blender-pro/

## Usage

Just ask Claude Code to make something in Blender:

- "Make me a wooden sword"
- "Create a cute cat pet for my Roblox game"
- "Build a medieval house"
- "Design a fantasy axe, show me 3 options"

The AI will interview you for details before building.

## File Structure

```
blender-pro/
├── SKILL.md — Main workflow (interview → blueprint → build → critique)
└── references/
    ├── objects/
    │   ├── primitives.md — Every mesh primitive with parameters
    │   ├── edit-operations.md — Every edit mode operation
    │   ├── modifiers.md — Every modifier with usage
    │   ├── model-patterns.md — Design questions per model type
    │   └── roblox-optimization.md — Export constraints
    └── textures/
        ├── shader-nodes.md — Every shader and texture node
        ├── material-recipes.md — Metal, wood, glass, skin, stone recipes
        └── uv-mapping.md — All UV mapping methods
```
