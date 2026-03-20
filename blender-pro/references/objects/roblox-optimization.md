# Roblox Optimization — Export Constraints

## Triangle Budgets
| Asset | Target | Max |
|---|---|---|
| Character | 1,000-3,000 | 5,000 |
| Pet | 500-1,500 | 3,000 |
| Weapon/Tool | 200-800 | 2,000 |
| Small Prop | 100-500 | 1,000 |
| Vehicle | 1,500-4,000 | 8,000 |
| Building | 2,000-5,000 | 10,000 |
| Tree | 200-800 | 1,500 |
| Terrain chunk | 5,000-10,000 | 20,000 |

## Check Tri Count
```python
for obj in bpy.context.scene.objects:
    if obj.type == 'MESH':
        depsgraph = bpy.context.evaluated_depsgraph_get()
        eval_obj = obj.evaluated_get(depsgraph)
        mesh = eval_obj.to_mesh()
        tris = sum(len(p.vertices) - 2 for p in mesh.polygons)
        print(f"{obj.name}: {tris} tris")
        eval_obj.to_mesh_clear()
```

## Pre-Export Checklist
```python
obj = bpy.context.active_object

# 1. Apply all modifiers
for mod in list(obj.modifiers):
    try:
        bpy.ops.object.modifier_apply(modifier=mod.name)
    except:
        pass

# 2. Apply transforms
bpy.ops.object.transform_apply(location=True, rotation=True, scale=True)

# 3. Triangulate
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.quads_convert_to_tris()
bpy.ops.object.mode_set(mode='OBJECT')

# 4. Cleanup
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.mesh.remove_doubles(threshold=0.001)
bpy.ops.mesh.normals_make_consistent(inside=False)
bpy.ops.object.mode_set(mode='OBJECT')
```

## FBX Export Settings
```python
bpy.ops.export_scene.fbx(
    filepath="model.fbx",
    use_selection=True,
    mesh_smooth_type='FACE',
    add_leaf_bones=False,
    apply_scale_options='FBX_SCALE_ALL',
    axis_forward='-Z',
    axis_up='Y',
)
```

## Key Constraints
- Max ~10,000 vertices per MeshPart
- Max 50MB per upload
- One texture per MeshPart
- Procedural textures must be baked to PNG before export
- Texture size: 1024x1024 standard, 512x512 for small props
