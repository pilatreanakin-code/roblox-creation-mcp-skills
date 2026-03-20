---
name: blender-pro
description: Makes the Blender MCP dramatically better at 3D modeling. Use this skill whenever the user asks to create, model, sculpt, or build ANYTHING in Blender — characters, creatures, pets, vehicles, buildings, props, weapons, environments, terrain, maps, decorations, or any 3D asset. This skill enforces proper Blender workflows with modifiers, node-based materials, and correct topology instead of naive vertex manipulation. Triggers on any mention of "make in Blender", "model a", "create a 3D", "build me a", "Blender model", or any 3D modeling request. ALWAYS use this skill for ANY Blender modeling task.
---

# Blender Pro — Professional 3D Modeling Skill

The AI generates a unique, hyper-detailed internal blueprint for every model, then executes it stage by stage. No fixed recipes — every model is designed fresh.

---

## THE WORKFLOW (4 steps, in order, no skipping)

### STEP 1 — Interview the User

Before creating anything, have a conversation to understand exactly what they want. The goal is to gather enough detail that no two users asking for "a sword" get the same result.

**Round 1 — What and Why:**
- What exactly are they making? Push past vague answers. "A sword" → "What kind of sword?"
- What's it for? Roblox game, animation, render, 3D print, just for fun?
- Any reference images or inspirations? Ask them to describe a specific look they have in mind.

**Round 2 — Art Style (offer choices):**
Ask: "What style should this be?" and offer:
- Low-poly flat (angular, minimal geometry, bold colors — like Roblox or Crossy Road)
- Stylized smooth (cartoon-like, vibrant, rounded — like Fortnite or Fall Guys)
- Semi-realistic (detailed but stylized — like Zelda BOTW or Genshin Impact)
- Realistic (photorealistic, PBR materials)
- Or describe your own

**Round 3 — Specific Details (adapt to what they're making):**
Ask detailed questions based on the model type. Read `references/objects/model-patterns.md` for the full question list per category, but here are examples:

For a weapon: "Straight or curved blade? Single or double-edged? How long is the blade relative to the handle? What shape is the guard — a cross, disc, S-curve, or something else? Does the pommel have a gem, a skull, or something simple? Any engravings, runes, or glow? What material — steel, wood, bone, crystal?"

For a character: "Body shape — round, slim, muscular? Big head or realistic proportions? What kind of eyes — big cute ones or realistic? Ears, tail, horns, wings? What are they wearing? What colors?"

For a building: "How many floors? Roof type — peaked, flat, dome? Window style — arched, rectangular, circular? Front door — big wooden door, glass, gate? Any special features — chimney, balcony, tower?"

**The more details you extract, the more unique the model will be. Never settle for vague answers.**

**Round 4 — Colors and Materials:**
- What's the main color or color palette?
- What material should the surface look like? (rough stone, polished metal, weathered wood, etc.)
- Should different parts have different materials or all the same?

**Round 5 — Confirm:**
Summarize everything in one paragraph: "I'll make [detailed description] in [style] with [materials]. The [specific details]. Sound right, or want to change anything?"

Only proceed after user confirms.

**MULTIPLE OPTIONS MODE:**
If the user asks for multiple options (e.g., "show me 3 versions", "give me some options", "I'm not sure, show me variations"), do this:

1. Generate a SEPARATE blueprint for each option. Each must be genuinely different — different shapes, different proportions, different material choices. Not just color swaps.
2. Build all options in the SAME Blender scene, placed SIDE BY SIDE with enough spacing that they don't overlap (typically 3-5 Blender units apart on the X axis).
3. The number of options matches what the user asked for. If they say "give me options" without a number, default to 3.
4. Take a screenshot showing ALL options visible at once (zoom out the viewport if needed).
5. Label each option clearly: name the objects "Option1_Blade", "Option2_Blade", etc. or create text labels.
6. Ask the user which one they prefer, or if they want to mix elements from different options.
7. Once the user picks, delete the rejected options and refine the chosen one.

---

### STEP 2 — Generate the Blueprint (THE KEY STEP)

Before writing ANY bpy code, write a complete internal blueprint. This is your design document. It must be extremely specific — not "a blade shape" but exact dimensions, exact operations, exact node connections.

**The blueprint has 3 mandatory sections:**

#### SECTION A: Parts List
For EVERY part, specify:
- Name (e.g., "Blade")
- Starting primitive (cube, sphere, cylinder, cone, torus, ico_sphere — see `references/objects/primitives.md` for all options and parameters)
- Exact initial scale (x, y, z)
- Shape modifications: which edit-mode operations to use (extrude, bevel, loop cut, subdivide, merge, etc. — see `references/objects/edit-operations.md`)
- Modifiers: which ones, in what order, with exact parameter values (see `references/objects/modifiers.md`)
- Shading: smooth or flat, auto_smooth angle if smooth
- Vertex count target

**Example of sufficient detail:**
"Blade: primitive_cube_add(size=1). Scale to (0.07, 0.018, 0.6). Apply scale. Edit mode: subdivide(number_cuts=4). Select vertices where z > 0.55, scale X to 0.1 for taper to point. Select center vertical edge loop, scale Y * 1.6 for diamond cross-section. Add modifier Subdivision Surface levels=1/render=2. Shade smooth, auto_smooth_angle=30 degrees."

#### SECTION B: Material Plan
For EVERY material, specify:
- Name and which parts use it
- Principled BSDF values: Base Color (exact RGB 0-1), Roughness, Metallic, Specular, Subsurface, Emission
- Procedural texture nodes: type, scale, detail, connections (see `references/textures/shader-nodes.md`)
- Node chain: exactly what connects to what
- Visual description: "weathered oak with visible grain, darker in crevices"

See `references/textures/material-recipes.md` for common material patterns.

**Example of sufficient detail:**
"Material 'Steel': Principled BSDF metallic=1.0, roughness=0.25. Noise Texture (scale=50, detail=4) → Color Ramp (stop0: RGB 0.6,0.6,0.65 at 0.0, stop1: RGB 0.75,0.75,0.8 at 1.0) → Base Color. Same Noise → Math(MULTIPLY, 0.15) → Math(ADD, 0.2) → Roughness. Noise → Bump(strength=0.1) → Normal."

#### SECTION C: Assembly Plan
For EVERY part, specify:
- How it connects to adjacent parts (which part it attaches to, from which side)
- Overlap amount — decide this per connection based on:
  - How big is the connecting part vs the receiving part?
  - Would it look more natural slightly embedded or just resting on top?
  - Small parts into large parts = more overlap. Large flat parts on each other = less overlap.
  - Think like a real object: a gem is SET INTO a pommel, not balanced on top. A handle INSERTS into a guard, not just touches it.
- Parent hierarchy: what parents to what

**CRITICAL ASSEMBLY RULES:**
1. Never hardcode positions. Always calculate from bounding boxes.
2. Parts should ALWAYS slightly overlap/penetrate each other — real objects are never perfectly flush. The AI decides HOW MUCH based on what looks natural for that specific connection.
3. After assembly, ALWAYS take a screenshot and evaluate if the connections look natural. If any part looks like it's floating or barely touching, increase the overlap and try again.

```python
# Get bounding box in world space
bbox = [obj.matrix_world @ Vector(corner) for corner in obj.bound_box]
bottom_z = min(v.z for v in bbox)
top_z = max(v.z for v in bbox)
height = top_z - bottom_z
```
Use `references/objects/model-patterns.md` for the assembly helper functions.

---

### STEP 3 — Build in Stages

Execute the blueprint using `mcp__blender__execute_blender_code`. Break into stages:

**Stage A — Create all parts** (one execute call per part, or batch small parts)
- Clear scene
- Create each part with its primitive, scale, and modifiers
- DO NOT position them yet

**Stage B — Apply materials** (one execute call)
- Create all materials with full node trees
- Assign to parts
- Switch viewport to MATERIAL preview

**Stage C — Assemble** (one execute call)
- Calculate bounding boxes of all parts
- Position parts using bounding box math (no gaps)
- Set parent hierarchy

**Stage C.5 — Self-Verify Assembly (MANDATORY)**
Run verification to catch misaligned or disconnected parts.
```python
from mathutils import Vector

print("=== ASSEMBLY VERIFICATION ===")
issues = []
parts = [obj for obj in bpy.context.scene.objects if obj.type == 'MESH']

for obj in parts:
    bbox = [obj.matrix_world @ Vector(c) for c in obj.bound_box]
    min_x = min(v.x for v in bbox)
    max_x = max(v.x for v in bbox)
    min_z = min(v.z for v in bbox)
    max_z = max(v.z for v in bbox)
    center_x = (min_x + max_x) / 2
    print(f"{obj.name}: center_x={center_x:.4f}, z_range=[{min_z:.4f} to {max_z:.4f}]")

# Check 1: Are all parts roughly centered on same X?
for obj in parts:
    bbox = [obj.matrix_world @ Vector(c) for c in obj.bound_box]
    cx = (min(v.x for v in bbox) + max(v.x for v in bbox)) / 2
    if abs(cx) > 0.5:
        issues.append(f"MISALIGNED: {obj.name} center_x={cx:.4f}")

# Check 2: Are there parts completely disconnected (big gaps)?
z_ranges = []
for obj in parts:
    bbox = [obj.matrix_world @ Vector(c) for c in obj.bound_box]
    z_ranges.append((obj.name, min(v.z for v in bbox), max(v.z for v in bbox)))
z_ranges.sort(key=lambda x: x[1])
for i in range(len(z_ranges) - 1):
    gap = z_ranges[i+1][1] - z_ranges[i][2]
    if gap > 0.02:  # positive gap = parts NOT overlapping
        issues.append(f"DISCONNECTED: {z_ranges[i][0]} and {z_ranges[i+1][0]} have a gap of {gap:.4f} — they should overlap")
    # Negative gap = overlap, which is GOOD

if issues:
    print("ISSUES FOUND — FIX BEFORE PROCEEDING:")
    for issue in issues:
        print(f"  - {issue}")
else:
    print("ASSEMBLY OK — parts connected")
```
Fix any issues, then proceed.

**Stage D — Visual Self-Critique (MANDATORY before showing user)**

Take a viewport screenshot with `mcp__blender__get_viewport_screenshot`. Before showing it to the user, critically evaluate EVERY aspect:

**Check shapes:**
- Do proportions look right? (head too big? limbs too thin? blade too wide?)
- Are shapes interesting or just basic primitives? (a "sword blade" should have a taper and cross-section, not just be a flat rectangle)
- Do organic shapes look smooth enough? (subdivision level high enough?)
- Do hard-surface shapes have proper edge definition? (bevel modifier applied?)

**Check materials/textures:**
- Do materials look like the intended surface? (does "wood" actually look like wood, or just brown plastic?)
- Is there enough surface variation? (real materials have noise, grain, roughness variation)
- Are colors too saturated or too dull for the chosen style?
- Do all parts that should match actually have the SAME material?

**Check connections/assembly:**
- Are parts properly overlapping? (no floating, no just-touching)
- Do structural joints look solid? (handle inserted into guard, not balanced on it)
- Are decorative elements embedded into their parent? (gems, eyes, buttons sunk in)

**Check overall:**
- Does it read as the thing the user asked for? (would someone looking at this immediately know what it is?)
- Is anything obviously wrong from this angle? Try another angle if unsure.

**If ANYTHING looks off:** Fix it NOW. Adjust overlap, tweak shapes, modify materials, change proportions. Take another screenshot. Repeat until you're satisfied.

**Only after you're satisfied:** Show the screenshot to the user and ask: "How does this look? Want me to change anything?"

---

### STEP 4 — Iterate

If the user wants changes:
- Identify which aspect needs modification (shape? materials? positions? proportions?)
- Fix only that part
- Run self-critique again (Stage D)
- Screenshot again
- Repeat until user is satisfied

---

### STEP 5 — Save to Project (automatic after model is approved)

Once the user approves the model, AUTOMATICALLY save a complete description file. The user does NOT need to ask for this — do it every time. The user can also trigger this manually by saying "save it", "log it", "save to project", or "archive this".

**Folder structure on disk:**
```
~/BlenderProjects/
  [GameName]/
    [ModelName].md
    [ModelName].md
    ...
```

- `GameName` = the project/game name. Ask the user what game this is for on the FIRST save. Remember it for the rest of the session. If they haven't mentioned a game, ask: "What project or game is this for? I'll save the model specs under that name."
- `ModelName` = descriptive name based on what was built (e.g., "golden_sword", "cute_fire_fox_pet", "medieval_house")

**The .md file must contain ALL of the following:**

```markdown
# [Model Name]
Created: [date]
Style: [art style chosen]
Purpose: [what it's for — Roblox, render, etc.]

## Description
[1-2 sentence plain English description of what was built and what it looks like]

## Parts
For each part:
- **[Part Name]**: [primitive used] → [transformations applied] → [modifiers with exact values]
  - Primitive: `bpy.ops.mesh.primitive_X_add(exact params used)`
  - Scale: (x, y, z)
  - Edit operations: [list every edit mode operation performed]
  - Modifiers: [name, type, every parameter value]
  - Shading: smooth/flat, auto_smooth angle

## Materials
For each material:
- **[Material Name]** (applied to: [list of parts])
  - Principled BSDF: Base Color RGB(x,x,x), Roughness=x, Metallic=x, [any other non-default values]
  - Node chain: [exact node connections, e.g., "Noise(scale=50, detail=4) → ColorRamp(stops...) → Base Color"]
  - Visual description: "[what it looks like in plain English]"

## Assembly
- [Part1] → [Part2]: overlap=X units, connection type (structural/decorative)
- Parent hierarchy: [what parents to what]
- Positioning method: [bounding box snap with overlap values used]

## Blender Settings
- Render engine used: [Cycles/Eevee]
- Subdivision levels: [viewport/render per part]
- Total approximate triangle count: [number]

## Export Notes (if exported to Roblox)
- Textures baked: yes/no
- Bake resolution: [1024/512]
- FBX export settings used
- Any issues encountered during export
```

**Generate this by reading the Blender scene programmatically:**

```python
import bpy
import os
import json
from datetime import datetime

game_name = "GAME_NAME"  # ask user or use from session
model_name = "MODEL_NAME"  # derive from what was built

# Create directory
save_dir = os.path.expanduser(f"~/BlenderProjects/{game_name}")
os.makedirs(save_dir, exist_ok=True)

# Gather data from scene
parts_info = []
materials_info = []

for obj in bpy.context.scene.objects:
    if obj.type != 'MESH':
        continue
    
    # Part info
    part = {
        "name": obj.name,
        "location": [round(x, 4) for x in obj.location],
        "scale": [round(x, 4) for x in obj.scale],
        "dimensions": [round(x, 4) for x in obj.dimensions],
        "modifiers": [],
        "materials": [],
        "vertex_count": len(obj.data.vertices),
        "face_count": len(obj.data.polygons),
    }
    
    # Modifiers
    for mod in obj.modifiers:
        mod_info = {"name": mod.name, "type": mod.type}
        if mod.type == 'SUBSURF':
            mod_info["levels"] = mod.levels
            mod_info["render_levels"] = mod.render_levels
        elif mod.type == 'BEVEL':
            mod_info["width"] = mod.width
            mod_info["segments"] = mod.segments
        elif mod.type == 'MIRROR':
            mod_info["axis"] = [mod.use_axis[0], mod.use_axis[1], mod.use_axis[2]]
        elif mod.type == 'SOLIDIFY':
            mod_info["thickness"] = mod.thickness
            mod_info["offset"] = mod.offset
        elif mod.type == 'BOOLEAN':
            mod_info["operation"] = mod.operation
        elif mod.type == 'ARRAY':
            mod_info["count"] = mod.count
        part["modifiers"].append(mod_info)
    
    # Materials on this part
    for ms in obj.material_slots:
        if ms.material:
            mat = ms.material
            mat_info = {"name": mat.name, "nodes": []}
            if mat.use_nodes:
                for node in mat.node_tree.nodes:
                    node_data = {"type": node.type, "name": node.name}
                    if node.type == 'BSDF_PRINCIPLED':
                        bc = node.inputs.get('Base Color')
                        if bc:
                            node_data["base_color"] = [round(x, 3) for x in bc.default_value[:3]]
                        node_data["roughness"] = round(node.inputs['Roughness'].default_value, 3)
                        node_data["metallic"] = round(node.inputs['Metallic'].default_value, 3)
                    elif node.type == 'TEX_NOISE':
                        node_data["scale"] = round(node.inputs['Scale'].default_value, 1)
                        node_data["detail"] = round(node.inputs['Detail'].default_value, 1)
                    elif node.type == 'TEX_VORONOI':
                        node_data["scale"] = round(node.inputs['Scale'].default_value, 1)
                    mat_info["nodes"].append(node_data)
            part["materials"].append(mat_info["name"])
            if mat_info not in materials_info:
                materials_info.append(mat_info)
    
    parts_info.append(part)

# Save as JSON (machine readable)
json_path = os.path.join(save_dir, f"{model_name}.json")
with open(json_path, 'w') as f:
    json.dump({"parts": parts_info, "materials": materials_info, "date": str(datetime.now())}, f, indent=2)

print(f"Saved project file: {json_path}")
```

Then ALSO generate the human-readable .md file using the gathered data. The .md is what the user reads. The .json is for the AI to reload later if the user says "rebuild the golden sword from my project."

**RELOADING:** If a user says "load the golden sword" or "rebuild [model name]", check `~/BlenderProjects/` for matching files and use the saved specs to recreate it exactly.

---

## HARD RULES

1. **NEVER create a single primitive and call it done.** Minimum 3 separate objects per model.
2. **NEVER skip materials.** Every model must have proper Principled BSDF materials with at least base color + roughness.
3. **NEVER hardcode Z positions for assembly.** Always use bounding box calculations.
4. **ALWAYS use modifiers** (subdivision, bevel, mirror) instead of manual vertex editing where possible.
5. **ALWAYS take a screenshot** after building and ask for feedback.
6. **ALWAYS apply the SAME material** to parts that should look the same. Create ONE material, assign to multiple objects.
7. **Stages can be as long as needed.** Don't artificially split code that belongs together. Each stage should be one logical operation, regardless of line count.

---

## REFERENCE FILES

Read these BEFORE generating your blueprint:

**For shapes and modeling:**
- `references/objects/primitives.md` — Every primitive with all parameters
- `references/objects/edit-operations.md` — Every edit mode operation (extrude, bevel, loop cut, merge, etc.)
- `references/objects/modifiers.md` — Every modifier with parameters and usage

**For materials and textures:**
- `references/textures/shader-nodes.md` — Every shader node, texture node, converter node with connections
- `references/textures/material-recipes.md` — Ready-to-use material patterns (metal, wood, glass, skin, stone, etc.)
- `references/textures/uv-mapping.md` — All UV mapping methods

**For specific model types:**
- `references/objects/model-patterns.md` — Patterns for characters, vehicles, buildings, props, environments

**For Roblox export:**
- `references/objects/roblox-optimization.md` — Tri budgets, export settings, constraints

---

## ERROR RECOVERY

If bpy code errors:
1. Read the error — most common: wrong mode (need EDIT vs OBJECT), no active object, wrong context
2. Fix: `bpy.ops.object.mode_set(mode='OBJECT')` or `mode='EDIT'`
3. Fix: `bpy.context.view_layer.objects.active = obj` before operations
4. Fix: `bpy.ops.object.select_all(action='DESELECT')` then select target
5. Never rewrite everything — fix the specific issue
