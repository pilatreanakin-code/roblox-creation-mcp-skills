# UV Mapping — Complete Reference

All methods require Edit Mode with faces selected.

## Smart UV Project (QUICK, works for most objects)
```python
bpy.ops.object.mode_set(mode='EDIT')
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.uv.smart_project(angle_limit=66.0, island_margin=0.02)
bpy.ops.object.mode_set(mode='OBJECT')
```
Best for: complex shapes, quick results, organic models.

## Cube Projection (boxy objects)
```python
bpy.ops.uv.cube_project(cube_size=1.0)
```
Best for: buildings, boxes, furniture.

## Cylinder Projection (tubular objects)
```python
bpy.ops.uv.cylinder_project(direction='ALIGN_TO_OBJECT')
```
Best for: limbs, pillars, pipes, barrels, trunks.

## Sphere Projection (round objects)
```python
bpy.ops.uv.sphere_project()
```
Best for: heads, planets, balls.

## Unwrap with Seams (maximum control)
```python
bpy.ops.object.mode_set(mode='EDIT')
# Select edges for seams (typically hidden edges: back of head, bottom of props)
bpy.ops.mesh.mark_seam()
bpy.ops.mesh.select_all(action='SELECT')
bpy.ops.uv.unwrap(method='ANGLE_BASED', margin=0.02)
# method options: 'ANGLE_BASED' (organic), 'CONFORMAL' (preserve angles)
bpy.ops.object.mode_set(mode='OBJECT')
```

## Follow Active Quads (uniform grids)
```python
bpy.ops.uv.follow_active_quads()
```
Best for: evenly subdivided surfaces.

## Lightmap Pack (baking)
```python
bpy.ops.uv.lightmap_pack(PREF_CONTEXT='ALL_FACES')
```

## Project from View
```python
bpy.ops.uv.project_from_view(camera_bounds=False, scale_to_bounds=True)
```

## Mark/Clear Seams
```python
bpy.ops.mesh.mark_seam(clear=False)   # mark
bpy.ops.mesh.mark_seam(clear=True)    # clear
```
