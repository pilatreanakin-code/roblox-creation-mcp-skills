# Edit Mode Operations — Complete Reference

All mesh editing operations. MUST be in Edit Mode (`bpy.ops.object.mode_set(mode='EDIT')`).

## Extrude
**Use:** Extend faces/edges/vertices outward to create new geometry.
```python
# Extrude selected faces and move
bpy.ops.mesh.extrude_region_move(TRANSFORM_OT_translate={"value":(0, 0, 1)})

# Extrude individual faces (each face separately)
bpy.ops.mesh.extrude_faces_move(TRANSFORM_OT_shrink_fatten={"value":0.5})

# Extrude vertices only
bpy.ops.mesh.extrude_vertices_move(TRANSFORM_OT_translate={"value":(0, 0, 0.5)})
```

## Inset Faces
**Use:** Create a smaller face inside selected faces (for windows, panels, details).
```python
bpy.ops.mesh.inset_faces(thickness=0.1, depth=0.0)
```
- `thickness`: inset distance inward
- `depth`: push inset face in/out

## Bevel
**Use:** Round edges, create chamfers, add detail to sharp edges.
```python
bpy.ops.mesh.bevel(offset=0.1, segments=3, profile=0.5, vertex_only=False)
```
- `offset`: bevel width
- `segments`: subdivisions (more = smoother)
- `profile`: shape (0.5 = semicircle, 0 = sharp, 1 = sharp opposite)
- `vertex_only`: True = only bevel vertices, not edges

## Loop Cut
**Use:** Add edge loops for detail, control subdivision, create joints.
```python
bpy.ops.mesh.loopcut_slide(MESH_OT_loopcut={"number_cuts":1},
                            TRANSFORM_OT_edge_slide={"value":0.0})
```
- `number_cuts`: how many parallel cuts
- `value`: slide position (-1 to 1, 0 = center)

## Subdivide
**Use:** Split selected edges/faces into smaller pieces.
```python
bpy.ops.mesh.subdivide(number_cuts=2, smoothness=0.0)
```
- `number_cuts`: divisions per edge
- `smoothness`: how much to smooth after subdividing

## Unsubdivide
**Use:** Reverse subdivision, reduce geometry.
```python
bpy.ops.mesh.unsubdivide(iterations=1)
```

## Merge Vertices
**Use:** Combine selected vertices into one point.
```python
bpy.ops.mesh.merge(type='CENTER')  # options: CENTER, COLLAPSE, CURSOR
```

## Remove Doubles
**Use:** Merge vertices that are very close together (cleanup).
```python
bpy.ops.mesh.remove_doubles(threshold=0.001)
```

## Fill
**Use:** Close holes by creating faces between selected edges.
```python
bpy.ops.mesh.fill(use_beauty=True)
```

## Grid Fill
**Use:** Fill between two edge loops with a quad grid (cleaner than Fill).
```python
bpy.ops.mesh.fill_grid(span=4, offset=0)
```

## Bridge Edge Loops
**Use:** Connect two open edge loops with faces (tubes, connections).
```python
bpy.ops.mesh.bridge_edge_loops(number_cuts=0, use_merge=False)
```

## Knife Project
**Use:** Cut a shape from another object onto the mesh.
```python
bpy.ops.mesh.knife_project(cut_through=True)
```

## Split
**Use:** Separate selected geometry from the rest (creates loose edges).
```python
bpy.ops.mesh.split()
```

## Separate
**Use:** Move selected faces to a new object entirely.
```python
bpy.ops.mesh.separate(type='SELECTED')  # options: SELECTED, MATERIAL, LOOSE
```

## Dissolve (smart delete)
**Use:** Remove geometry while preserving surrounding shape.
```python
bpy.ops.mesh.dissolve_edges(use_verts=True)   # dissolve selected edges
bpy.ops.mesh.dissolve_faces()                   # dissolve selected faces
bpy.ops.mesh.dissolve_verts()                   # dissolve selected vertices
bpy.ops.mesh.dissolve_limited(angle_limit=0.087)  # auto-dissolve flat areas
```

## Poke
**Use:** Convert each face into a triangle fan from its center.
```python
bpy.ops.mesh.poke(offset=0.0)
```

## Triangulate
**Use:** Convert all faces to triangles (required for game export).
```python
bpy.ops.mesh.quads_convert_to_tris(quad_method='BEAUTY', ngon_method='BEAUTY')
```

## Smooth Vertices
**Use:** Relax vertex positions for smoother surface.
```python
bpy.ops.mesh.vertices_smooth(factor=1.0, repeat=10)
```

## Recalculate Normals
**Use:** Fix inside-out faces (essential cleanup step).
```python
bpy.ops.mesh.normals_make_consistent(inside=False)
```

## Mark Sharp / Mark Seam
**Use:** Control shading edges / UV seam placement.
```python
bpy.ops.mesh.mark_sharp()          # mark selected edges as sharp
bpy.ops.mesh.mark_sharp(clear=True)  # clear sharp
bpy.ops.mesh.mark_seam()           # mark as UV seam
bpy.ops.mesh.mark_seam(clear=True)   # clear seam
```

## Spin
**Use:** Create revolution shapes by rotating selected vertices around an axis.
```python
bpy.ops.mesh.spin(steps=32, angle=3.1416, center=(0,0,0), axis=(0,0,1))
```

## Screw
**Use:** Create helical shapes (screws, springs, spiral stairs).
```python
bpy.ops.mesh.screw(steps=24, turns=3, axis=(0,0,1), center=(0,0,0))
```

## Rip
**Use:** Tear vertices apart, creating open edges.
```python
bpy.ops.mesh.rip(mirror=False)
```

## Transform Operations (in Edit Mode)
```python
bpy.ops.transform.translate(value=(x, y, z))           # move
bpy.ops.transform.rotate(value=radians, orient_axis='Z') # rotate
bpy.ops.transform.resize(value=(sx, sy, sz))            # scale
bpy.ops.transform.shrink_fatten(value=0.1)              # push along normals
bpy.ops.transform.tosphere(value=1.0)                   # spherize
bpy.ops.transform.shear(value=0.5, orient_axis='X')     # shear
```

## BMesh (Advanced Vertex Manipulation)
**Use:** When you need precise per-vertex control (but prefer modifiers when possible).
```python
import bmesh
obj = bpy.context.active_object
bpy.ops.object.mode_set(mode='EDIT')
bm = bmesh.from_edit_mesh(obj.data)
bm.verts.ensure_lookup_table()

for v in bm.verts:
    if v.co.z > 0.5:      # select by position
        v.co.x *= 0.5     # modify position

bmesh.update_edit_mesh(obj.data)
bpy.ops.object.mode_set(mode='OBJECT')
```
