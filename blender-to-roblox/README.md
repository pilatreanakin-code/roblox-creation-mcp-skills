# Blender to Roblox Studio — Export Skill

A Claude Code skill that fully automates exporting 3D meshes and textures from Blender into Roblox Studio. No manual file dragging.

## Pipeline

Blender MCP exports FBX + bakes ALL textures to PNG → Open Cloud API uploads to Roblox → Roblox Studio MCP places everything with correct scale, position, and textures.

## Features

- Full mesh export (real FBX, not Part reconstruction)
- Mandatory texture baking (procedural materials converted to PNG automatically)
- Correct scaling using world-space bounding boxes
- Coordinate conversion (Blender Z-up to Roblox Y-up)
- Fallback to flat color if bake fails (never arrives textureless)
- Hierarchy support (Blender collections become Roblox Models)

## Requirements

- Claude Code with Blender MCP and Roblox Studio MCP
- Roblox Open Cloud API key with Assets read+write

## Installation

Copy the blender-to-roblox folder to ~/.claude/skills/blender-to-roblox/

## Usage

Open Claude Code with both MCPs, then:

"My Roblox API key is: KEY. My Creator ID is: ID. Export everything from Blender to Roblox Studio."

## License

MIT
