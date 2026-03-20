# Roblox Studio Placement Code — Reference

Luau code to run via `mcp__Roblox_Studio__run_code` to place uploaded meshes in Studio.

## Table of Contents
1. The Scaling Problem (READ THIS FIRST)
2. Place a Single MeshPart with Correct Scale
3. Place with Texture
4. Place Multiple Objects — Batch Script with Scaling
5. Create Hierarchy (Models from Collections)
6. Coordinate System Conversion
7. Common Adjustments

---

## 1. The Scaling Problem (READ THIS FIRST)

When Blender exports individual parts of a multi-part model, each part may have different local scales. For example, a character's body might be at scale 1.0 but the eyes at scale 0.05. When each is exported as a separate FBX, Roblox imports each one and assigns it a default Size based on the mesh's own vertices — it has no idea about the original scale relative to other parts.

**The fix: use the world-space bounding box from Blender to resize each MeshPart after import.**

The pipeline:
1. Blender export captures `world_bbox_size` for each object (the actual visible size in the scene)
2. MeshPart is placed in Roblox with its default import size
3. We read the MeshPart's initial Size (what Roblox auto-assigned)
4. We compute a scale ratio: `desired_size / initial_size`
5. We apply the scaled Size so all parts are proportionally correct relative to each other

**CRITICAL:** You must read the MeshPart's Size AFTER setting the MeshId, because Roblox recalculates the bounding box when the mesh loads. Add a short `task.wait()` after setting MeshId to let it load.

---

## 2. Place a Single MeshPart with Correct Scale

```lua
-- Place with correct scaling from Blender world-space bounding box
local meshPart = Instance.new("MeshPart")
meshPart.Name = "OBJECT_NAME"
meshPart.Anchored = true
meshPart.Parent = workspace

-- Set mesh (must be parented first so it loads)
meshPart.MeshId = "rbxassetid://MESH_ASSET_ID"
task.wait(0.5) -- wait for mesh to load so Size updates

-- Read the default import size (Roblox auto-assigned)
local importedSize = meshPart.Size

-- Desired size from Blender world-space bounding box (converted to Roblox axes)
-- Blender world_bbox_size = [bx, by, bz]
-- Roblox: X = bx, Y = bz, Z = by
local SCALE_MULTIPLIER = 1 -- adjust if everything is too small/large overall
local desiredX = BLENDER_BBOX_X * SCALE_MULTIPLIER
local desiredY = BLENDER_BBOX_Z * SCALE_MULTIPLIER  -- Blender Z = Roblox Y
local desiredZ = BLENDER_BBOX_Y * SCALE_MULTIPLIER  -- Blender Y = Roblox Z

-- Calculate scale ratio per axis
local ratioX = (importedSize.X > 0) and (desiredX / importedSize.X) or 1
local ratioY = (importedSize.Y > 0) and (desiredY / importedSize.Y) or 1
local ratioZ = (importedSize.Z > 0) and (desiredZ / importedSize.Z) or 1

-- Use UNIFORM scale (largest ratio) to avoid distortion
local uniformRatio = math.max(ratioX, ratioY, ratioZ)
meshPart.Size = importedSize * uniformRatio

-- OR use per-axis scaling if mesh proportions should match exactly
-- meshPart.Size = Vector3.new(desiredX, desiredY, desiredZ)

-- Position (from Blender world_bbox_center, converted to Roblox coords)
-- Blender center = [cx, cy, cz] → Roblox = (cx, cz, -cy)
meshPart.CFrame = CFrame.new(RBX_X, RBX_Y, RBX_Z)

-- Color (if no texture)
meshPart.Color = Color3.fromRGB(R, G, B)

print("Placed: " .. meshPart.Name .. " | Size: " .. tostring(meshPart.Size))
```

---

## 3. Place with Texture

```lua
local meshPart = Instance.new("MeshPart")
meshPart.Name = "OBJECT_NAME"
meshPart.Anchored = true
meshPart.Parent = workspace

meshPart.MeshId = "rbxassetid://MESH_ASSET_ID"
meshPart.TextureID = "rbxassetid://TEXTURE_ASSET_ID"
task.wait(0.5)

-- Scale using world bounding box (same logic as above)
local importedSize = meshPart.Size
local desiredX = BLENDER_BBOX_X * SCALE_MULTIPLIER
local desiredY = BLENDER_BBOX_Z * SCALE_MULTIPLIER
local desiredZ = BLENDER_BBOX_Y * SCALE_MULTIPLIER
local uniformRatio = math.max(
    (importedSize.X > 0) and (desiredX / importedSize.X) or 1,
    (importedSize.Y > 0) and (desiredY / importedSize.Y) or 1,
    (importedSize.Z > 0) and (desiredZ / importedSize.Z) or 1
)
meshPart.Size = importedSize * uniformRatio

meshPart.CFrame = CFrame.new(RBX_X, RBX_Y, RBX_Z)
print("Placed with texture: " .. meshPart.Name)
```

---

## 4. Place Multiple Objects — Batch Script with Scaling

This is the main script the skill should generate. It reads the manifest and upload results, applies correct scaling using world bounding boxes, and positions everything with proper coordinate conversion.

```lua
-- Auto-generated placement script with world-space scaling
local SCALE_MULTIPLIER = 1 -- set globally, adjust if everything is wrong size

local function place(name, meshId, textureId, bboxX, bboxY, bboxZ, centerX, centerY, centerZ, r, g, b)
    local mp = Instance.new("MeshPart")
    mp.Name = name
    mp.Anchored = true
    mp.Parent = workspace

    -- Load mesh
    mp.MeshId = "rbxassetid://" .. tostring(meshId)
    if textureId and textureId ~= "nil" then
        mp.TextureID = "rbxassetid://" .. tostring(textureId)
    else
        mp.Color = Color3.fromRGB(r or 200, g or 200, b or 200)
    end
    task.wait(0.5) -- let mesh load

    -- Scale to match Blender world-space size
    -- bboxX/Y/Z are BLENDER world_bbox_size values (Blender coords)
    -- Convert: Roblox X = Blender X, Roblox Y = Blender Z, Roblox Z = Blender Y
    local desiredSizeX = bboxX * SCALE_MULTIPLIER
    local desiredSizeY = bboxZ * SCALE_MULTIPLIER
    local desiredSizeZ = bboxY * SCALE_MULTIPLIER

    local imported = mp.Size
    if imported.X > 0 and imported.Y > 0 and imported.Z > 0 then
        local rX = desiredSizeX / imported.X
        local rY = desiredSizeY / imported.Y
        local rZ = desiredSizeZ / imported.Z
        local uniformRatio = math.max(rX, rY, rZ)
        mp.Size = imported * uniformRatio
    end

    -- Position using Blender world_bbox_center (converted to Roblox coords)
    -- Blender (cx, cy, cz) → Roblox (cx, cz, -cy)
    local rbxX = centerX * SCALE_MULTIPLIER
    local rbxY = centerZ * SCALE_MULTIPLIER
    local rbxZ = -centerY * SCALE_MULTIPLIER
    mp.CFrame = CFrame.new(rbxX, rbxY, rbxZ)

    print("Placed: " .. name .. " size=" .. tostring(mp.Size))
end

-- Objects (generated from manifest + upload results)
-- place(name, meshId, textureId, bboxX, bboxY, bboxZ, centerX, centerY, centerZ, r, g, b)
place("Body", 111111, nil, 1.2, 0.8, 2.0, 0, 0, 1.0, 200, 150, 120)
place("Eye_L", 222222, nil, 0.1, 0.1, 0.1, -0.15, -0.4, 1.8, 255, 255, 255)
place("Eye_R", 333333, nil, 0.1, 0.1, 0.1, 0.15, -0.4, 1.8, 255, 255, 255)
-- ... one line per object, values from manifest world_bbox_size and world_bbox_center
```

**When generating this script, the skill should:**
1. Read upload_results.json to get mesh_id and texture_id per object
2. Read manifest.json to get `world_bbox_size`, `world_bbox_center`, and colors
3. Generate one `place()` call per object using the Blender-coordinate values directly (the function handles conversion internally)
4. Set SCALE_MULTIPLIER based on user preference or default 1
5. Run the whole script via `mcp__Roblox_Studio__run_code`

**Note on task.wait():** Each object needs 0.5s for the mesh to load before we can read its Size. For 10 objects, that's 5 seconds total. This is expected. For large scenes (50+ objects), batch them into groups of 10 per run_code call.

---

## 5. Create Hierarchy (Models from Collections)

If Blender objects are organized in collections, create corresponding Models in Roblox.

```lua
-- Create model for a Blender collection
local model = Instance.new("Model")
model.Name = "COLLECTION_NAME"
model.Parent = workspace

-- Place objects inside the model (same scaling logic)
local mp = Instance.new("MeshPart")
mp.Name = "OBJECT_NAME"
mp.Anchored = true
mp.Parent = model  -- parent to model, not workspace

mp.MeshId = "rbxassetid://MESH_ID"
task.wait(0.5)

-- Apply world-space scaling (same as above)
local imported = mp.Size
local desired = Vector3.new(BBOX_RBX_X, BBOX_RBX_Y, BBOX_RBX_Z)
local ratio = math.max(desired.X/imported.X, desired.Y/imported.Y, desired.Z/imported.Z)
mp.Size = imported * ratio

mp.CFrame = CFrame.new(X, Y, Z)
print("Created model: COLLECTION_NAME with " .. mp.Name)
```

**Hierarchy logic:**
- Each unique Blender collection becomes a Roblox Model
- Objects with the same `collection` field in the manifest go into the same Model
- Objects with no collection go directly into workspace
- Nested collections: create nested Models

---

## 6. Coordinate System Conversion

Blender and Roblox use different coordinate systems. Apply this conversion:

```
Blender (right-handed, Z-up)    Roblox (left-handed, Y-up)
  X  →  X (same)
  Y  →  -Z (Blender Y becomes negative Roblox Z)
  Z  →  Y (Blender Z becomes Roblox Y)
```

**Position conversion (from world_bbox_center):**
```
roblox_x = blender_center_x
roblox_y = blender_center_z
roblox_z = -blender_center_y
```

**Size conversion (from world_bbox_size):**
```
roblox_size_x = blender_bbox_x
roblox_size_y = blender_bbox_z
roblox_size_z = blender_bbox_y
```

**Rotation conversion (Euler):**
```
roblox_rx = blender_rx
roblox_ry = blender_rz
roblox_rz = -blender_ry
```

**Scale multiplier:**
By default, 1 Blender unit = 1 Roblox stud. If objects appear too small or large, ask the user for a scale multiplier. Common values:
- Characters/props: 1-3x
- Architectural scenes: 3-5x
- Vehicles: 2-4x

---

## 7. Common Adjustments

**All parts are correct proportions but everything is too small/large:**
Adjust SCALE_MULTIPLIER globally. Try 2, 5, or 10 until it looks right. This is normal — Blender meters and Roblox studs are similar but not identical depending on the project.

**Parts are correct size but positioned wrong:**
Check that you're using `world_bbox_center` (not `location`) for positioning. The center of the bounding box gives more accurate placement than the object origin, which could be at the feet, center, or anywhere.

**Mesh appears underground:**
Add half the Y-size to lift it:
```lua
local yOffset = meshPart.Size.Y / 2
meshPart.CFrame = CFrame.new(x, y + yOffset, z)
```

**Mesh appears with wrong rotation:**
The FBX export with `axis_forward='-Z', axis_up='Y'` should handle most cases. If still wrong, try applying rotation in Blender before export:
```python
bpy.ops.object.transform_apply(location=False, rotation=True, scale=True)
```

**Mesh is too detailed for Roblox:**
Roblox has a 10,000 triangle limit per MeshPart. Decimate in Blender:
```python
bpy.ops.object.modifier_add(type='DECIMATE')
bpy.context.object.modifiers["Decimate"].ratio = 0.5
bpy.ops.object.modifier_apply(modifier="Decimate")
```

**Texture appears stretched:**
UV issue. Ensure UVs are properly unwrapped before baking. The skill auto-generates UVs via Smart UV Project if none exist.

**Multiple materials on one object:**
Roblox MeshParts support only ONE texture. If a Blender object has multiple materials:
1. Bake all materials into a single atlas texture
2. Or inform the user and suggest splitting the object in Blender

**Placing inside a specific folder:**
```lua
local targetFolder = workspace:FindFirstChild("MyFolder")
if not targetFolder then
    targetFolder = Instance.new("Folder")
    targetFolder.Name = "MyFolder"
    targetFolder.Parent = workspace
end
meshPart.Parent = targetFolder
```
