# Blender to Roblox Studio Skill

A Claude Code skill that fully automates exporting 3D meshes and textures from Blender into Roblox Studio. No manual file dragging — everything is handled through MCPs and the Roblox Open Cloud API.

## How It Works

Blender (MCP) → Export FBX + Bake Textures → Upload to Roblox (Open Cloud API) → Place in Studio (MCP)

1. **Blender MCP** exports each object as `.fbx`, bakes textures to `.png`, and captures world-space bounding boxes
2. **Roblox Open Cloud API** uploads meshes and textures to Roblox's servers
3. **Roblox Studio MCP** places every MeshPart with correct scale, position, rotation, and textures

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Blender MCP](https://github.com/ahujasid/blender-mcp) connected to Claude Code
- [Roblox Studio MCP](https://github.com/longbui99/roblox-studio-mcp) connected to Claude Code
- Roblox Open Cloud API key with Assets read+write permissions

## Compatibility

This skill is built for **Claude Code**, but the pipeline works with any AI coding assistant that supports MCPs:

- **Cursor** — load SKILL.md as context in your project
- **Windsurf** — add the reference files to your workspace
- **Cline / Continue** (VS Code) — paste SKILL.md as system instructions

For non-Claude tools: ignore the skill format and just use the files in `references/` as instructions. The Python scripts (Blender export), API calls (upload), and Luau code (Studio placement) are universal — no Claude-specific code anywhere.

**No AI at all?** You can follow the reference docs manually:
1. Run the Python scripts from `blender-export-code.md` in Blender's scripting tab
2. Run the upload commands from `open-cloud-api.md` in your terminal
3. Paste the Luau code from `roblox-placement-code.md` into Studio's command bar

## Installation

1. Download or clone this repo
2. Place the folder at `~/.claude/skills/blender-to-roblox/`
3. Set up your Roblox Open Cloud API key (see below)

## API Key Setup

1. Go to [create.roblox.com/dashboard/credentials](https://create.roblox.com/dashboard/credentials)
2. Click **Create API Key**
3. Add **Assets** API system with **Read** and **Write** permissions
4. Set accepted IP addresses to `0.0.0.0/0`
5. Save and copy the generated key

## Usage

Open Claude Code with both MCPs connected, then:

```
My Roblox Open Cloud API key is: YOUR_KEY_HERE
My Roblox Creator ID is: YOUR_ID_HERE

Export all objects from my Blender scene to Roblox Studio with textures.
```

### Other commands

```
Export only the selected objects from Blender to Roblox Studio.
```

```
Export the "Buildings" collection from Blender to Roblox Studio.
```

## Features

- **Full mesh export** — real FBX meshes, not Part reconstructions
- **Texture baking** — automatically bakes diffuse textures to PNG and uploads them
- **Correct scaling** — uses world-space bounding boxes so multi-part models maintain correct proportions
- **Coordinate conversion** — handles Blender Z-up to Roblox Y-up axis conversion
- **Hierarchy support** — Blender collections become Roblox Models
- **Color fallback** — objects without textures get their Blender material color applied

## File Structure

```
blender-to-roblox/
├── SKILL.md                              # Main skill instructions
└── references/
    ├── setup-guide.md                    # API key setup walkthrough
    ├── blender-export-code.md            # Python/bpy export templates
    ├── open-cloud-api.md                 # Upload API reference
    └── roblox-placement-code.md          # Luau placement with scaling
```

## Troubleshooting

| Problem | Fix |
|---|---|
| Parts are wrong size relative to each other | Make sure skill uses world_bbox_size not dimensions |
| Everything is too small/large | Adjust SCALE_MULTIPLIER (try 2, 5, or 10) |
| 401 Unauthorized | Regenerate your API key |
| 403 Forbidden | Recreate key with Assets read+write |
| Asset moderated | Rename to something generic, simplify mesh |
| Texture stretched | Check UVs in Blender before export |

## License

MIT
