# Mesh Primitives — Complete Reference

Every primitive available in Blender with all parameters and use cases.

## Cube
**Use for:** Bodies, walls, boxes, building blocks, any boxy shape. Most common starting point.
```python
bpy.ops.mesh.primitive_cube_add(size=2.0, location=(0,0,0), rotation=(0,0,0),
                                 calc_uvs=True, enter_editmode=False, align='WORLD')
```
- `size`: edge length (default 2.0)
- `calc_uvs`: auto-generate UVs (default True)
- `enter_editmode`: open in edit mode immediately

## UV Sphere
**Use for:** Rounded objects, heads, eyes, planets, balls. Configurable density.
```python
bpy.ops.mesh.primitive_uv_sphere_add(segments=32, ring_count=16, radius=1.0,
                                      location=(0,0,0), calc_uvs=True)
```
- `segments`: vertical slices (more = smoother, default 32)
- `ring_count`: horizontal rings (default 16)
- `radius`: sphere radius

## Cylinder
**Use for:** Limbs, pillars, barrels, pipes, handles, trunks. Very versatile.
```python
bpy.ops.mesh.primitive_cylinder_add(vertices=32, radius=1.0, depth=2.0,
                                     end_fill_type='NGON', location=(0,0,0), calc_uvs=True)
```
- `vertices`: number of sides (8 for low-poly, 32 for smooth)
- `radius`: cylinder radius
- `depth`: height
- `end_fill_type`: cap style — `'NGON'` (single face), `'TRIFAN'` (triangles), `'NOTHING'` (open)

## Cone
**Use for:** Tails, horns, spikes, roofs, nails, funnels. Truncated cone if radius2 > 0.
```python
bpy.ops.mesh.primitive_cone_add(vertices=32, radius1=1.0, radius2=0.0, depth=2.0,
                                 end_fill_type='NGON', location=(0,0,0))
```
- `vertices`: sides
- `radius1`: bottom radius
- `radius2`: top radius (0 = pointed cone, >0 = truncated)
- `depth`: height

## Torus
**Use for:** Rings, donuts, tires, guards, bracelets, O-rings.
```python
bpy.ops.mesh.primitive_torus_add(major_segments=48, minor_segments=12,
                                  major_radius=1.0, minor_radius=0.25,
                                  location=(0,0,0), generate_uvs=True)
```
- `major_radius`: ring center radius (overall size)
- `minor_radius`: tube thickness
- `major_segments`: smoothness of ring
- `minor_segments`: smoothness of tube cross-section

## Ico Sphere
**Use for:** Gems, crystals, rocks, organic spheres, low-poly balls. More uniform faces than UV sphere.
```python
bpy.ops.mesh.primitive_ico_sphere_add(subdivisions=2, radius=1.0,
                                       location=(0,0,0), calc_uvs=True)
```
- `subdivisions`: detail level (1=very low poly, 2=default, 4=smooth)
- `radius`: sphere radius
- **Key difference from UV Sphere:** faces are more uniform, better for sculpting and low-poly styles

## Plane
**Use for:** Floors, walls, leaves, panels, cards, flat surfaces.
```python
bpy.ops.mesh.primitive_plane_add(size=2.0, location=(0,0,0), calc_uvs=True)
```
- `size`: half the side length

## Grid
**Use for:** Terrain, subdivided floors, fabric base, displacement surfaces.
```python
bpy.ops.mesh.primitive_grid_add(x_subdivisions=10, y_subdivisions=10, size=1.0,
                                 location=(0,0,0))
```
- `x_subdivisions`, `y_subdivisions`: resolution (more = more vertices for sculpting/displacement)

## Circle
**Use for:** Profiles for spin/screw, references, halos, selection rings. Not filled by default.
```python
bpy.ops.mesh.primitive_circle_add(vertices=32, radius=1.0, fill_type='NOTHING',
                                   location=(0,0,0))
```
- `fill_type`: `'NOTHING'` (open ring), `'NGON'` (filled disc), `'TRIFAN'` (triangle fan fill)

## Monkey (Suzanne)
**Use for:** Material/lighting tests only. Not for production models.
```python
bpy.ops.mesh.primitive_monkey_add(size=1.0, location=(0,0,0))
```

## After Creating Any Primitive
The new object is automatically the active object. Access it:
```python
obj = bpy.context.active_object
obj.name = "MyName"
obj.scale = (1, 1, 1.5)  # modify scale
bpy.ops.object.transform_apply(scale=True)  # apply scale to mesh data
```
