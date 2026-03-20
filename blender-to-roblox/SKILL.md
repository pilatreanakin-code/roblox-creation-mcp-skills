---
name: blender-to-roblox
description: Export 3D meshes and textures from Blender to Roblox Studio — fully automated. Use this skill whenever the user wants to move models, objects, scenes, or assets from Blender into Roblox Studio. Triggers include any mention of "export from Blender to Roblox", "move Blender model to Studio", "import mesh into Roblox", "Blender to Roblox pipeline", "upload mesh to Roblox", or any request to transfer 3D assets between Blender and Roblox Studio. Also triggers when the user wants to export a full Blender scene to Roblox, bake textures for Roblox, or set up a Blender-to-Roblox workflow. Requires: Blender MCP, Roblox Studio MCP, and a Roblox Open Cloud API key.
---

# Blender to Roblox Studio — Mesh & Texture Export Skill

This skill automates the full pipeline of exporting 3D meshes with textures from Blender into Roblox Studio. No manual file dragging — everything is handled through MCPs and the Roblox Open Cloud API.

## Prerequisites

Before first use, the user needs a Roblox Open Cloud API key. If they don't have one, read `references/setup-guide.md` and walk them through it.

The API key and Creator ID should be stored as environment variables:
- `ROBLOX_API_KEY` — the Open Cloud API key
- `ROBLOX_CREATOR_ID` — the user's Roblox User ID (found at roblox.com → profile URL)

If these aren't set, ask the user for them and export them in the current shell session:
```bash
set ROBLOX_API_KEY=their_key_here
set ROBLOX_CREATOR_ID=their_id_here
```

## The Pipeline (3 Stages)

Every export follows this exact sequence. Do NOT skip or reorder stages.

### Stage 1: Blender Export

Use `mcp__blender__execute_blender_code` to run Python (bpy) code. **Follow this exact order:**

**Step 1.1 — BAKE ALL TEXTURES FIRST (MANDATORY — NOT OPTIONAL)**
Roblox CANNOT use Blender's procedural materials. Node-based materials (Noise Texture, Voronoi, Color Ramp, wood grain, etc.) will appear as flat grey or black in Roblox. Every material MUST be baked to a PNG image.

Run the "Bake ALL Textures" script from `references/blender-export-code.md` section 4 BEFORE doing anything else. This:
- Auto-generates UVs on any object missing them
- Bakes every mesh object's material to a PNG
- Falls back to flat base color if procedural bake fails
- Saves a manifest of all baked textures

**Step 1.2 — Export each mesh as FBX**
For EACH object:
   a. Apply all modifiers
   b. Apply transforms
   c. Triangulate the mesh
   d. Export as `.fbx` to the temp directory

**Step 1.3 — Extract transforms and metadata**
For EACH object:
   a. **CRITICAL: Calculate world-space bounding box size and center** using `obj.matrix_world @ Vector(corner)` for all 8 bounding box corners
   b. Extract material base color as fallback (in case texture upload fails)
   c. Build the export manifest (JSON) linking each object to its FBX path, texture PNG path, bounding box, and color

Read `references/blender-export-code.md` for the exact code templates.

**Critical Blender export settings for Roblox:**
- FBX: `use_selection=True, mesh_smooth_type='FACE', add_leaf_bones=False, apply_scale_options='FBX_SCALE_ALL'`
- Bake resolution: 1024x1024 (default), 512x512 for small props
- Apply all modifiers before export
- Triangulate meshes before export

**IF BAKING FAILS:** At minimum, extract the Principled BSDF Base Color value and create a solid-color PNG. The object should NEVER arrive in Roblox without any texture — it must have at least a flat color image.

### Stage 2: Upload to Roblox via Open Cloud API

For each exported file, upload it to Roblox's asset servers using the Open Cloud Assets API.

**Upload a mesh (.fbx):**
```bash
curl -s -X POST "https://apis.roblox.com/assets/v1/assets" \
  -H "x-api-key: %ROBLOX_API_KEY%" \
  -F "request={\"assetType\":\"Model\",\"displayName\":\"OBJECT_NAME\",\"description\":\"Exported from Blender\",\"creationContext\":{\"creator\":{\"userId\":\"CREATOR_ID\"}}};type=application/json" \
  -F "fileContent=@PATH_TO_FBX;type=model/fbx"
```

**Upload a texture (.png):**
```bash
curl -s -X POST "https://apis.roblox.com/assets/v1/assets" \
  -H "x-api-key: %ROBLOX_API_KEY%" \
  -F "request={\"assetType\":\"Decal\",\"displayName\":\"OBJECT_NAME_texture\",\"description\":\"Texture from Blender\",\"creationContext\":{\"creator\":{\"userId\":\"CREATOR_ID\"}}};type=application/json" \
  -F "fileContent=@PATH_TO_PNG;type=image/png"
```

**Handling the response:**
The API returns a JSON operation. Poll for completion:
```bash
curl -s -X GET "https://apis.roblox.com/assets/v1/operations/OPERATION_ID" \
  -H "x-api-key: %ROBLOX_API_KEY%"
```

When `done: true`, extract the `assetId` from the response. The Roblox asset URI format is:
- Mesh: `rbxassetid://ASSET_ID`
- Texture: `rbxassetid://ASSET_ID`

**Important:** Poll every 2 seconds, up to 30 attempts. Asset moderation can take a few seconds. If it fails moderation, inform the user and skip that asset.

Read `references/open-cloud-api.md` for full API details, error handling, and edge cases.

### Stage 3: Place in Roblox Studio (with Correct Scaling)

Use `mcp__Roblox_Studio__run_code` to execute Luau that creates the objects in Studio.

**CRITICAL — SCALING FIX FOR MULTI-PART MODELS:**
When a model has multiple parts (body, eyes, feet, accessories), each part may have different local scales in Blender. Exporting them individually means Roblox has no idea about their relative sizes. The fix:

1. Place the MeshPart and set its MeshId
2. Wait 0.5 seconds for the mesh to load (`task.wait(0.5)`)
3. Read the auto-assigned `meshPart.Size` (Roblox's default import size)
4. Calculate a uniform scale ratio: `desired_bbox / imported_size`
5. Apply: `meshPart.Size = importedSize * ratio`
6. Position using `world_bbox_center` (NOT object origin) for accurate placement

The `world_bbox_size` and `world_bbox_center` come from the Blender manifest (Stage 1). They represent the actual visible size and position in the Blender scene.

For each uploaded mesh:
```lua
local meshPart = Instance.new("MeshPart")
meshPart.Name = "OBJECT_NAME"
meshPart.Anchored = true
meshPart.Parent = workspace

meshPart.MeshId = "rbxassetid://MESH_ASSET_ID"
-- meshPart.TextureID = "rbxassetid://TEXTURE_ASSET_ID" (if has texture)
task.wait(0.5) -- REQUIRED: let mesh load before reading Size

-- Scale to match Blender world-space bounding box
local imported = meshPart.Size
-- world_bbox_size from Blender: [bx, by, bz]
-- Convert: Roblox X = bx, Roblox Y = bz (Blender Z-up), Roblox Z = by
local SCALE = 1 -- global multiplier, adjust if needed
local desX, desY, desZ = BBOX_X * SCALE, BBOX_Z * SCALE, BBOX_Y * SCALE
local ratio = math.max(desX/imported.X, desY/imported.Y, desZ/imported.Z)
meshPart.Size = imported * ratio

-- Position from world_bbox_center: Blender (cx,cy,cz) → Roblox (cx, cz, -cy)
meshPart.CFrame = CFrame.new(CENTER_X * SCALE, CENTER_Z * SCALE, -CENTER_Y * SCALE)
```

Read `references/roblox-placement-code.md` for the full batch placement script, hierarchy creation, and troubleshooting.

**Coordinate system conversion (Blender to Roblox):**
- Blender X → Roblox X
- Blender Y → Roblox -Z
- Blender Z → Roblox Y
- Scale: 1 Blender unit = 1 Roblox stud (adjustable via SCALE_MULTIPLIER)

## Export Modes

### Single Object
User says something like "export this object to Roblox". Export only the active/selected object.

### Full Scene
User says "export everything" or "export the whole scene". Export all mesh objects, maintaining hierarchy:
- Create a Model in Roblox for each Blender collection/parent
- Parent MeshParts to their respective Models
- Maintain relative transforms

### Batch by Collection
User says "export the Buildings collection". Export only objects in the named Blender collection.

## Error Handling

| Error | Cause | Fix |
|---|---|---|
| `401 Unauthorized` | Bad API key | Ask user to regenerate key |
| `403 Forbidden` | Missing Assets permission | Re-create key with Assets read+write |
| `400 Bad Request` on mesh | FBX too large or invalid | Re-export with lower poly count, check triangulation |
| `Moderated` status | Asset flagged by Roblox | Inform user, suggest renaming or simplifying |
| No texture on object | Object uses procedural material | Bake to texture first, or export color only |
| Parts are wrong size relative to each other | Local scale not baked, or using local dims instead of world bbox | Use `world_bbox_size` from manifest for sizing, NOT `dimensions` or `scale`. Read roblox-placement-code.md Section 1 |
| Mesh appears tiny in Studio | Scale mismatch | Increase SCALE_MULTIPLIER (try 2, 5, 10) |
| Mesh appears rotated | Coordinate system | Double-check the axis conversion |
| Parts in wrong positions | Using object origin instead of bbox center | Use `world_bbox_center` for positioning |

## Temp Directory

Use `%TEMP%\blender_roblox_export\` on Windows as the working directory for all exported files. Create it if it doesn't exist. Clean up after successful placement in Studio (ask user first).

## Step-by-Step Execution Checklist

When executing the pipeline, follow this exact order and confirm each step:

1. Check that ROBLOX_API_KEY and ROBLOX_CREATOR_ID are available
2. Ask user: which objects to export (selected, all, or specific collection)?
3. Run Blender export code → confirm files written to disk
4. For each file, upload to Roblox → collect all asset IDs
5. Run Roblox Studio placement code → confirm objects appear
6. Report: "Exported X meshes and Y textures. All placed in workspace."

Always tell the user what you're doing at each step. If any step fails, stop and troubleshoot before continuing.
