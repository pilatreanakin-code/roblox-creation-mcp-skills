# Blender Export Code Templates

These are the exact bpy code blocks to run via `mcp__blender__execute_blender_code`. Execute them in order.

## Table of Contents
1. Setup & Validation
2. Export Single Object as FBX
3. Export All Scene Objects as FBX
4. Bake Diffuse Texture to PNG
5. Extract Material Color (no texture)
6. Extract Object Transforms
7. Build Export Manifest (JSON)
8. Full Pipeline Script (all-in-one)

---

## 1. Setup & Validation

Run this first to ensure the scene is ready and create the export directory.

```python
import bpy
import os
import json

export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
os.makedirs(export_dir, exist_ok=True)

# Count mesh objects
mesh_objects = [obj for obj in bpy.context.scene.objects if obj.type == 'MESH']
selected_objects = [obj for obj in bpy.context.selected_objects if obj.type == 'MESH']

print(json.dumps({
    "export_dir": export_dir,
    "total_mesh_objects": len(mesh_objects),
    "selected_mesh_objects": len(selected_objects),
    "object_names": [obj.name for obj in mesh_objects],
    "selected_names": [obj.name for obj in selected_objects],
}))
```

---

## 2. Export Single Object as FBX

Replace `OBJECT_NAME` with the target object name.

```python
import bpy
import os

export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
obj_name = "OBJECT_NAME"

# Deselect all, select target
bpy.ops.object.select_all(action='DESELECT')
obj = bpy.data.objects.get(obj_name)
if obj is None:
    print(f"ERROR: Object '{obj_name}' not found")
else:
    obj.select_set(True)
    bpy.context.view_layer.objects.active = obj

    # Apply modifiers
    for mod in obj.modifiers:
        try:
            bpy.ops.object.modifier_apply(modifier=mod.name)
        except:
            pass

    # Triangulate
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.select_all(action='SELECT')
    bpy.ops.mesh.quads_convert_to_tris()
    bpy.ops.object.mode_set(mode='OBJECT')

    # Export FBX
    filepath = os.path.join(export_dir, f"{obj_name}.fbx")
    bpy.ops.export_scene.fbx(
        filepath=filepath,
        use_selection=True,
        mesh_smooth_type='FACE',
        add_leaf_bones=False,
        apply_scale_options='FBX_SCALE_ALL',
        axis_forward='-Z',
        axis_up='Y',
    )
    print(f"OK: Exported {filepath}")
```

---

## 3. Export All Scene Objects as FBX (individual files)

```python
import bpy
import os
import json

export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
results = []

mesh_objects = [obj for obj in bpy.context.scene.objects if obj.type == 'MESH']

for obj in mesh_objects:
    bpy.ops.object.select_all(action='DESELECT')
    obj.select_set(True)
    bpy.context.view_layer.objects.active = obj

    # Apply modifiers on a copy to avoid destructive changes
    # (skip if user wants to preserve the scene)
    for mod in list(obj.modifiers):
        try:
            bpy.ops.object.modifier_apply(modifier=mod.name)
        except:
            pass

    # Triangulate
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.select_all(action='SELECT')
    bpy.ops.mesh.quads_convert_to_tris()
    bpy.ops.object.mode_set(mode='OBJECT')

    filepath = os.path.join(export_dir, f"{obj.name}.fbx")
    bpy.ops.export_scene.fbx(
        filepath=filepath,
        use_selection=True,
        mesh_smooth_type='FACE',
        add_leaf_bones=False,
        apply_scale_options='FBX_SCALE_ALL',
        axis_forward='-Z',
        axis_up='Y',
    )
    results.append({"name": obj.name, "fbx_path": filepath})

print(json.dumps(results))
```

**Warning:** This applies modifiers and triangulates destructively. If the user wants to preserve their scene, suggest saving a copy first or using a duplicate-then-export approach.

---

## 4. Bake ALL Textures for Roblox Export (MANDATORY)

Roblox CANNOT use Blender's procedural materials (Noise, Voronoi, Color Ramp, etc.). ALL materials must be baked to PNG images before export. This script bakes EVERY mesh object in the scene.

**Run this BEFORE exporting any FBX files. Not optional.**

```python
import bpy
import os

export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
os.makedirs(export_dir, exist_ok=True)
resolution = 1024

# Save and switch to Cycles (required for baking)
original_engine = bpy.context.scene.render.engine
bpy.context.scene.render.engine = 'CYCLES'
bpy.context.scene.cycles.samples = 32
bpy.context.scene.cycles.use_denoising = False

mesh_objects = [obj for obj in bpy.context.scene.objects if obj.type == 'MESH']
baked_files = []

for obj in mesh_objects:
    print(f"--- Baking: {obj.name} ---")
    
    # Skip objects with no materials
    if not obj.material_slots or not any(ms.material for ms in obj.material_slots):
        print(f"  SKIP: {obj.name} has no materials")
        continue
    
    # Step 1: Ensure UVs exist
    bpy.ops.object.select_all(action='DESELECT')
    obj.select_set(True)
    bpy.context.view_layer.objects.active = obj
    
    if not obj.data.uv_layers:
        print(f"  Creating UVs for {obj.name}...")
        bpy.ops.object.mode_set(mode='EDIT')
        bpy.ops.mesh.select_all(action='SELECT')
        bpy.ops.uv.smart_project(angle_limit=66, island_margin=0.02)
        bpy.ops.object.mode_set(mode='OBJECT')
    
    # Step 2: Create bake target image
    img_name = f"{obj.name}_baked"
    if img_name in bpy.data.images:
        bpy.data.images.remove(bpy.data.images[img_name])
    img = bpy.data.images.new(img_name, resolution, resolution, alpha=False)
    
    # Step 3: Add BakeTarget image node to EVERY material on this object
    bake_nodes = []
    for mat_slot in obj.material_slots:
        mat = mat_slot.material
        if mat and mat.use_nodes:
            nodes = mat.node_tree.nodes
            # Remove old bake node if exists
            old = nodes.get("BakeTarget")
            if old:
                nodes.remove(old)
            # Create new bake target
            tex_node = nodes.new('ShaderNodeTexImage')
            tex_node.name = "BakeTarget"
            tex_node.image = img
            # CRITICAL: must be the ACTIVE node for baking
            nodes.active = tex_node
            bake_nodes.append((mat, tex_node))
    
    if not bake_nodes:
        print(f"  SKIP: {obj.name} has no node-based materials")
        continue
    
    # Step 4: Select only this object and bake
    bpy.ops.object.select_all(action='DESELECT')
    obj.select_set(True)
    bpy.context.view_layer.objects.active = obj
    
    try:
        bpy.ops.object.bake(
            type='DIFFUSE',
            pass_filter={'COLOR'},
            use_clear=True,
            margin=4
        )
        
        # Step 5: Save the baked image
        filepath = os.path.join(export_dir, f"{obj.name}_diffuse.png")
        img.filepath_raw = filepath
        img.file_format = 'PNG'
        img.save()
        baked_files.append({"name": obj.name, "texture_path": filepath})
        print(f"  OK: Saved {filepath}")
        
    except Exception as e:
        print(f"  ERROR baking {obj.name}: {e}")
        # Fallback: try to at least get the base color as a flat image
        try:
            mat = obj.material_slots[0].material
            if mat and mat.use_nodes:
                principled = None
                for node in mat.node_tree.nodes:
                    if node.type == 'BSDF_PRINCIPLED':
                        principled = node
                        break
                if principled:
                    color = principled.inputs['Base Color'].default_value
                    # Create solid color image
                    pixels = [color[0], color[1], color[2], 1.0] * (resolution * resolution)
                    img.pixels = pixels
                    filepath = os.path.join(export_dir, f"{obj.name}_diffuse.png")
                    img.filepath_raw = filepath
                    img.file_format = 'PNG'
                    img.save()
                    baked_files.append({"name": obj.name, "texture_path": filepath})
                    print(f"  FALLBACK: Saved flat color for {obj.name}")
        except Exception as e2:
            print(f"  FALLBACK FAILED: {e2}")
    
    # Step 6: Cleanup bake nodes
    for mat, tex_node in bake_nodes:
        try:
            mat.node_tree.nodes.remove(tex_node)
        except:
            pass

# Restore render engine
bpy.context.scene.render.engine = original_engine

print(f"\n=== BAKING COMPLETE ===")
print(f"Baked {len(baked_files)} textures:")
for bf in baked_files:
    print(f"  {bf['name']} → {bf['texture_path']}")

import json
manifest_path = os.path.join(export_dir, "bake_manifest.json")
with open(manifest_path, 'w') as f:
    json.dump(baked_files, f, indent=2)
print(f"Manifest: {manifest_path}")
```

**Key features of this script:**
- Bakes EVERY mesh object, not just one
- Auto-generates UVs if missing
- Creates a BakeTarget node on every material of every object
- Falls back to flat color if procedural bake fails (so you at least get the base color)
- Saves a manifest listing all baked textures
- Cleans up after itself

**IMPORTANT:** For multi-part objects (like a sword with blade + guard + handle + pommel), each part gets its OWN baked texture. The upload script then uploads each texture separately and assigns it to the correct MeshPart in Roblox.

### When Baking Fails
Common issues:
- **"No active image found"**: The BakeTarget node isn't set as active. Make sure `nodes.active = tex_node`
- **"Circular reference"**: A node setup loops. Disconnect any feedback loops before baking
- **Black output**: Object has no UV map. Run Smart UV Project first
- **Very slow bake**: Reduce resolution to 512 for small props, or reduce Cycles samples to 16
```

---

## 5. Extract Material Color (when no texture)

Use this when an object has a solid color material but no image texture.

```python
import bpy
import json

obj_name = "OBJECT_NAME"
obj = bpy.data.objects.get(obj_name)

color_info = {"name": obj_name, "has_texture": False, "color": [200, 200, 200]}

if obj and obj.material_slots:
    mat = obj.material_slots[0].material
    if mat and mat.use_nodes:
        principled = None
        for node in mat.node_tree.nodes:
            if node.type == 'BSDF_PRINCIPLED':
                principled = node
                break
        if principled:
            base_input = principled.inputs.get('Base Color')
            if base_input:
                # Check if connected to a texture
                if base_input.links:
                    color_info["has_texture"] = True
                else:
                    c = base_input.default_value
                    color_info["color"] = [int(c[0]*255), int(c[1]*255), int(c[2]*255)]

print(json.dumps(color_info))
```

---

## 6. Extract Object Transforms (with World-Space Bounding Box)

Gets position, rotation, scale, AND the world-space bounding box for all mesh objects. The world-space bounding box is critical for correct sizing in Roblox — it accounts for object scale, modifiers, and parenting.

```python
import bpy
import json
import math
import mathutils

transforms = []
for obj in bpy.context.scene.objects:
    if obj.type != 'MESH':
        continue
    loc = obj.location
    rot = obj.rotation_euler
    scale = obj.scale
    dims = obj.dimensions

    # CRITICAL: Calculate world-space bounding box
    # This gives the TRUE size of the object as it appears in the scene,
    # accounting for scale, modifiers, and parent transforms.
    bbox_corners = [obj.matrix_world @ mathutils.Vector(corner) for corner in obj.bound_box]
    ws_min = mathutils.Vector((
        min(c.x for c in bbox_corners),
        min(c.y for c in bbox_corners),
        min(c.z for c in bbox_corners),
    ))
    ws_max = mathutils.Vector((
        max(c.x for c in bbox_corners),
        max(c.y for c in bbox_corners),
        max(c.z for c in bbox_corners),
    ))
    ws_size = ws_max - ws_min  # world-space width, depth, height
    ws_center = (ws_min + ws_max) / 2  # world-space center point

    transforms.append({
        "name": obj.name,
        "location": {"x": round(loc.x, 4), "y": round(loc.y, 4), "z": round(loc.z, 4)},
        "rotation_euler": {"x": round(rot.x, 4), "y": round(rot.y, 4), "z": round(rot.z, 4)},
        "scale": {"x": round(scale.x, 4), "y": round(scale.y, 4), "z": round(scale.z, 4)},
        "dimensions": {"x": round(dims.x, 4), "y": round(dims.y, 4), "z": round(dims.z, 4)},
        "world_bbox_size": {"x": round(ws_size.x, 4), "y": round(ws_size.y, 4), "z": round(ws_size.z, 4)},
        "world_bbox_center": {"x": round(ws_center.x, 4), "y": round(ws_center.y, 4), "z": round(ws_center.z, 4)},
        "parent": obj.parent.name if obj.parent else None,
        "collection": obj.users_collection[0].name if obj.users_collection else None,
    })

print(json.dumps(transforms))
```

**Why world_bbox_size matters:** In Blender, an eye might have local scale 0.1 and the body scale 1.0. The `dimensions` field reflects local space, but `world_bbox_size` gives you the actual visible size in the scene. This is what Roblox needs to size each part correctly relative to the others.

---

## 7. Build Export Manifest

After running exports, build a manifest that Stage 2 and 3 consume. This combines FBX paths, texture paths, transforms, colors, AND world-space bounding boxes for correct sizing.

```python
import bpy
import os
import json
import mathutils

export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
manifest = []

for obj in bpy.context.scene.objects:
    if obj.type != 'MESH':
        continue

    # Calculate world-space bounding box
    bbox_corners = [obj.matrix_world @ mathutils.Vector(corner) for corner in obj.bound_box]
    ws_min = mathutils.Vector((
        min(c.x for c in bbox_corners),
        min(c.y for c in bbox_corners),
        min(c.z for c in bbox_corners),
    ))
    ws_max = mathutils.Vector((
        max(c.x for c in bbox_corners),
        max(c.y for c in bbox_corners),
        max(c.z for c in bbox_corners),
    ))
    ws_size = ws_max - ws_min
    ws_center = (ws_min + ws_max) / 2

    entry = {
        "name": obj.name,
        "fbx_path": os.path.join(export_dir, f"{obj.name}.fbx"),
        "texture_path": None,
        "color": [200, 200, 200],
        "has_texture": False,
        "location": [round(obj.location.x, 4), round(obj.location.y, 4), round(obj.location.z, 4)],
        "rotation": [round(obj.rotation_euler.x, 4), round(obj.rotation_euler.y, 4), round(obj.rotation_euler.z, 4)],
        "scale": [round(obj.scale.x, 4), round(obj.scale.y, 4), round(obj.scale.z, 4)],
        "dimensions": [round(obj.dimensions.x, 4), round(obj.dimensions.y, 4), round(obj.dimensions.z, 4)],
        "world_bbox_size": [round(ws_size.x, 4), round(ws_size.y, 4), round(ws_size.z, 4)],
        "world_bbox_center": [round(ws_center.x, 4), round(ws_center.y, 4), round(ws_center.z, 4)],
        "parent": obj.parent.name if obj.parent else None,
    }

    # Check for texture
    tex_path = os.path.join(export_dir, f"{obj.name}_diffuse.png")
    if os.path.exists(tex_path):
        entry["texture_path"] = tex_path
        entry["has_texture"] = True
    else:
        # Get base color
        if obj.material_slots:
            mat = obj.material_slots[0].material
            if mat and mat.use_nodes:
                for node in mat.node_tree.nodes:
                    if node.type == 'BSDF_PRINCIPLED':
                        bc = node.inputs.get('Base Color')
                        if bc and not bc.links:
                            c = bc.default_value
                            entry["color"] = [int(c[0]*255), int(c[1]*255), int(c[2]*255)]
                        elif bc and bc.links:
                            entry["has_texture"] = True  # needs baking
                        break

    manifest.append(entry)

manifest_path = os.path.join(export_dir, "manifest.json")
with open(manifest_path, 'w') as f:
    json.dump(manifest, f, indent=2)
print(f"OK: Manifest written to {manifest_path}")
print(json.dumps(manifest))
```

---

## 8. Full Pipeline Script (all-in-one for single object)

This combines steps 2, 4/5, and 6 into one script for quick single-object exports.

```python
import bpy
import os
import json
import math

obj_name = "OBJECT_NAME"
export_dir = os.path.join(os.environ.get('TEMP', '/tmp'), 'blender_roblox_export')
os.makedirs(export_dir, exist_ok=True)

obj = bpy.data.objects.get(obj_name)
if not obj or obj.type != 'MESH':
    print(f"ERROR: '{obj_name}' is not a valid mesh object")
else:
    result = {"name": obj_name, "fbx_path": None, "texture_path": None, "color": None, "transform": None}

    # Select only this object
    bpy.ops.object.select_all(action='DESELECT')
    obj.select_set(True)
    bpy.context.view_layer.objects.active = obj

    # Apply modifiers
    for mod in list(obj.modifiers):
        try:
            bpy.ops.object.modifier_apply(modifier=mod.name)
        except:
            pass

    # Triangulate
    bpy.ops.object.mode_set(mode='EDIT')
    bpy.ops.mesh.select_all(action='SELECT')
    bpy.ops.mesh.quads_convert_to_tris()
    bpy.ops.object.mode_set(mode='OBJECT')

    # Export FBX
    fbx_path = os.path.join(export_dir, f"{obj_name}.fbx")
    bpy.ops.export_scene.fbx(
        filepath=fbx_path,
        use_selection=True,
        mesh_smooth_type='FACE',
        add_leaf_bones=False,
        apply_scale_options='FBX_SCALE_ALL',
        axis_forward='-Z',
        axis_up='Y',
    )
    result["fbx_path"] = fbx_path

    # Transform
    loc = obj.location
    rot = obj.rotation_euler
    scale = obj.scale
    result["transform"] = {
        "blender_pos": [round(loc.x,4), round(loc.y,4), round(loc.z,4)],
        "blender_rot": [round(rot.x,4), round(rot.y,4), round(rot.z,4)],
        "blender_scale": [round(scale.x,4), round(scale.y,4), round(scale.z,4)],
        "roblox_pos": [round(loc.x,4), round(loc.z,4), round(-loc.y,4)],
    }

    # Color / Texture
    has_image_texture = False
    if obj.material_slots:
        mat = obj.material_slots[0].material
        if mat and mat.use_nodes:
            for node in mat.node_tree.nodes:
                if node.type == 'BSDF_PRINCIPLED':
                    bc = node.inputs.get('Base Color')
                    if bc and bc.links:
                        has_image_texture = True
                    elif bc:
                        c = bc.default_value
                        result["color"] = [int(c[0]*255), int(c[1]*255), int(c[2]*255)]
                    break

    # Bake texture if needed
    if has_image_texture and obj.data.uv_layers:
        img_name = f"{obj_name}_bake"
        if img_name in bpy.data.images:
            bpy.data.images.remove(bpy.data.images[img_name])
        img = bpy.data.images.new(img_name, 1024, 1024)

        for ms in obj.material_slots:
            m = ms.material
            if m and m.use_nodes:
                tn = m.node_tree.nodes.new('ShaderNodeTexImage')
                tn.image = img
                tn.name = "BakeTarget"
                m.node_tree.nodes.active = tn

        orig_engine = bpy.context.scene.render.engine
        bpy.context.scene.render.engine = 'CYCLES'
        bpy.context.scene.cycles.samples = 16
        bpy.ops.object.bake(type='DIFFUSE', pass_filter={'COLOR'}, use_clear=True)

        tex_path = os.path.join(export_dir, f"{obj_name}_diffuse.png")
        img.filepath_raw = tex_path
        img.file_format = 'PNG'
        img.save()
        result["texture_path"] = tex_path

        bpy.context.scene.render.engine = orig_engine
        for ms in obj.material_slots:
            m = ms.material
            if m and m.use_nodes:
                bn = m.node_tree.nodes.get("BakeTarget")
                if bn:
                    m.node_tree.nodes.remove(bn)

    print(json.dumps(result))
```

**Note:** Run this code in chunks if Blender MCP has execution time limits. Split into: export FBX → bake texture → extract transform.
