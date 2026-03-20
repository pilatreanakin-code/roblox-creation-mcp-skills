# Modifiers — Complete Reference

Add with `obj.modifiers.new("Name", 'TYPE')`. Apply with `bpy.ops.object.modifier_apply(modifier="Name")`.
Recommended stack order: Mirror → Array → Boolean → Subdivision → Solidify → Bevel → Weighted Normal.

## GENERATE MODIFIERS

### Subdivision Surface (MOST USED)
**Use:** Smooth any shape. The #1 modifier for quality models.
```python
mod = obj.modifiers.new("Subdiv", 'SUBSURF')
mod.levels = 1          # viewport (keep low while modeling)
mod.render_levels = 2   # render (higher for final)
mod.subdivision_type = 'CATMULL_CLARK'  # or 'SIMPLE' for flat subdivision
```

### Mirror (ESSENTIAL for symmetry)
**Use:** Model half, mirror the other. Always add FIRST.
```python
mod = obj.modifiers.new("Mirror", 'MIRROR')
mod.use_axis[0] = True      # X axis
mod.use_bisect_axis[0] = True  # cut at center
mod.use_clip = True          # prevent crossing center
# mod.mirror_object = other_obj  # optional: mirror around another object
```

### Bevel (realistic edges)
**Use:** No real object has perfectly sharp edges. Bevel everything.
```python
mod = obj.modifiers.new("Bevel", 'BEVEL')
mod.width = 0.02
mod.segments = 2
mod.limit_method = 'ANGLE'      # only bevel edges sharper than angle
mod.angle_limit = 0.523599      # 30 degrees
# mod.limit_method = 'WEIGHT'   # only bevel edges with bevel weight
```

### Boolean (cut holes, combine shapes)
**Use:** Windows, doors, cutouts, mechanical details.
```python
mod = obj.modifiers.new("Bool", 'BOOLEAN')
mod.operation = 'DIFFERENCE'    # cut (also: 'UNION', 'INTERSECT')
mod.object = cutter_object
# After applying, delete the cutter:
# bpy.ops.object.modifier_apply(modifier="Bool")
# bpy.data.objects.remove(cutter_object)
```

### Array (repeat patterns)
**Use:** Fences, stairs, teeth, scales, windows, bricks.
```python
mod = obj.modifiers.new("Array", 'ARRAY')
mod.count = 5
mod.relative_offset_displace = (1.2, 0, 0)   # relative spacing
# mod.use_constant_offset = True
# mod.constant_offset_displace = (2.0, 0, 0)  # absolute spacing
# mod.fit_type = 'FIT_CURVE'                  # fit along a curve
# mod.curve = curve_object
```

### Solidify (give thickness)
**Use:** Walls, shells, armor plates, leaves, any flat thing that needs depth.
```python
mod = obj.modifiers.new("Solidify", 'SOLIDIFY')
mod.thickness = 0.05
mod.offset = -1        # -1 = inward, 0 = center, 1 = outward
mod.use_even_offset = True
```

### Decimate (reduce polygons)
**Use:** Optimize for games, reduce sculpted meshes.
```python
mod = obj.modifiers.new("Decimate", 'DECIMATE')
mod.ratio = 0.5        # keep 50% of faces
# mod.decimate_type = 'UNSUBDIV'  # reverse subdivision
# mod.iterations = 1
```

### Screw (revolution shapes)
**Use:** Bottles, vases, chess pieces, lathe-turned objects.
```python
mod = obj.modifiers.new("Screw", 'SCREW')
mod.steps = 32
mod.render_steps = 64
mod.angle = 6.28318    # full revolution (2*pi)
mod.axis = 'Z'
# mod.screw_offset = 0.5  # for helix/spring
```

### Skin (tubes from edges)
**Use:** Quick organic shapes, branches, pipes, tentacles.
```python
mod = obj.modifiers.new("Skin", 'SKIN')
# Then adjust per-vertex radius in edit mode
```

### Wireframe
**Use:** Wire/cage effects, lattice-like structures.
```python
mod = obj.modifiers.new("Wire", 'WIREFRAME')
mod.thickness = 0.02
```

### Remesh (retopology)
**Use:** Clean up messy sculpts, create uniform topology.
```python
mod = obj.modifiers.new("Remesh", 'REMESH')
mod.mode = 'VOXEL'     # or 'SMOOTH', 'SHARP', 'BLOCKS'
mod.voxel_size = 0.05
```

### Edge Split
**Use:** Hard edges for flat-shaded low-poly models.
```python
mod = obj.modifiers.new("EdgeSplit", 'EDGE_SPLIT')
mod.split_angle = 0.523599  # 30 degrees
```

### Weld (merge close vertices)
**Use:** Cleanup after boolean operations.
```python
mod = obj.modifiers.new("Weld", 'WELD')
mod.merge_threshold = 0.001
```

## DEFORM MODIFIERS

### Curve (bend along path)
**Use:** Roads, pipes, snakes, text on path, anything following a curve.
```python
mod = obj.modifiers.new("Curve", 'CURVE')
mod.object = curve_object
mod.deform_axis = 'POS_X'
```

### Displace (terrain, surface detail)
**Use:** Terrain from flat plane, surface bumps, procedural detail.
```python
mod = obj.modifiers.new("Displace", 'DISPLACE')
tex = bpy.data.textures.new("DispTex", type='CLOUDS')
tex.noise_scale = 5.0
mod.texture = tex
mod.strength = 2.0
mod.mid_level = 0.5
```

### Simple Deform (twist, bend, taper, stretch)
**Use:** Quick deformations without manual vertex work.
```python
mod = obj.modifiers.new("Deform", 'SIMPLE_DEFORM')
mod.deform_method = 'TWIST'   # TWIST, BEND, TAPER, STRETCH
mod.angle = 1.5708            # 90 degrees
mod.deform_axis = 'Z'
```

### Lattice (cage deformation)
**Use:** Reshape entire object smoothly without touching vertices.
```python
# First create a lattice object
bpy.ops.object.add(type='LATTICE', location=obj.location)
lattice = bpy.context.active_object
lattice.data.points_u = 3
lattice.data.points_v = 3
lattice.data.points_w = 3

# Then add modifier
mod = obj.modifiers.new("Lattice", 'LATTICE')
mod.object = lattice
```

### Shrinkwrap (snap to surface)
**Use:** Clothing, decals, wrapping objects around others.
```python
mod = obj.modifiers.new("Shrinkwrap", 'SHRINKWRAP')
mod.target = target_object
mod.wrap_method = 'NEAREST_SURFACEPOINT'  # or 'PROJECT', 'NEAREST_VERTEX'
mod.offset = 0.01
```

### Cast (spherize/cylindrize)
**Use:** Round off shapes gradually.
```python
mod = obj.modifiers.new("Cast", 'CAST')
mod.cast_type = 'SPHERE'  # SPHERE, CYLINDER, CUBOID
mod.factor = 0.5
```

### Wave (animated waves)
**Use:** Water surface, flags, fabric animation.
```python
mod = obj.modifiers.new("Wave", 'WAVE')
mod.height = 0.5
mod.width = 1.5
mod.speed = 1.0
```

## Applying Modifiers
```python
# Apply a specific modifier
bpy.ops.object.modifier_apply(modifier="ModifierName")

# Apply all modifiers
for mod in list(obj.modifiers):
    try:
        bpy.ops.object.modifier_apply(modifier=mod.name)
    except:
        pass
```
