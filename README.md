# Roblox & Blender Skills for Claude Code

A collection of Claude Code skills for 3D modeling and game development with Blender and Roblox Studio.

## Skills

### [Blender to Roblox](./blender-to-roblox)
Fully automated export pipeline. Blender MCP exports FBX + bakes textures to PNG → Open Cloud API uploads to Roblox → Roblox Studio MCP places everything with correct scale, position, and textures.

### [Blender Pro](./blender-pro)
Transforms the Blender MCP into a professional 3D artist. Deep interviews, blueprint design, proper modeling techniques, visual self-critique, multiple options mode, and auto-save project files.

## Compatibility

Built for **Claude Code**, but works with any AI tool that supports MCPs:
- **Cursor** — load SKILL.md as context
- **Windsurf** — add reference files to workspace
- **Cline / Continue** (VS Code) — paste SKILL.md as instructions

No AI at all? Follow the reference docs manually — all code is standard Python (bpy) and Luau.

## Installation

Clone and copy to your skills directory:

```
git clone https://github.com/pilatreanakin-code/roblox-blender-skills.git
cp -r roblox-blender-skills/blender-to-roblox ~/.claude/skills/blender-to-roblox
cp -r roblox-blender-skills/blender-pro ~/.claude/skills/blender-pro
```

## License

MIT
