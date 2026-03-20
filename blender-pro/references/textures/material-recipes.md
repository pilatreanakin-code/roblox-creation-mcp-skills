# Material Recipes — Common Surface Patterns

These are NODE CHAIN patterns. When generating your blueprint, describe the exact chain for each material. Adapt colors/values to match the user's request.

## Helper Function
```python
def create_material(name, color, roughness=0.5, metallic=0.0):
    mat = bpy.data.materials.new(name)
    mat.use_nodes = True
    p = mat.node_tree.nodes.get("Principled BSDF")
    p.inputs['Base Color'].default_value = (*color, 1.0)
    p.inputs['Roughness'].default_value = roughness
    p.inputs['Metallic'].default_value = metallic
    return mat, p, mat.node_tree.nodes, mat.node_tree.links

def assign_material(obj, mat):
    if obj.data.materials:
        obj.data.materials[0] = mat
    else:
        obj.data.materials.append(mat)
```

## METALS

### Steel
Chain: Noise(50,4) → ColorRamp(0.6,0.6,0.65 → 0.75,0.75,0.8) → Base Color. Same Noise → Math(MUL 0.15) → Math(ADD 0.2) → Roughness.
```
Principled: metallic=1.0, roughness=0.25, base_color=(0.65,0.65,0.7)
```

### Gold
```
Principled: metallic=1.0, roughness=0.2, base_color=(1.0, 0.76, 0.33)
```

### Chrome
```
Principled: metallic=1.0, roughness=0.05, base_color=(0.8, 0.8, 0.8)
```

### Bronze
```
Principled: metallic=1.0, roughness=0.3, base_color=(0.72, 0.45, 0.2)
```

### Rusty Metal
Chain: Noise(3,6) → ColorRamp(dark_iron → orange_rust) → Base Color. Voronoi(8) → Math(MUL 0.4) → Math(ADD 0.5) → Roughness.
```
Principled: metallic=1.0, roughness=0.6, base_color=(0.4,0.25,0.15)
```

## ORGANIC

### Wood
Chain: Noise(6,8,distortion=1.5) with Mapping(scale=1,1,8 for grain stretch) → ColorRamp(dark_wood → light_wood) → Base Color.
```
Principled: metallic=0.0, roughness=0.75
Typical colors: Pine(0.6,0.4,0.2), Oak(0.4,0.25,0.1), Walnut(0.15,0.08,0.03), Birch(0.8,0.7,0.5)
```

### Skin
Chain: Base color + Noise(15,4) mixed at 0.05 for subtle variation.
```
Principled: metallic=0.0, roughness=0.5, subsurface_weight=0.3, subsurface_color=(0.9,0.3,0.2)
Light: (0.85,0.65,0.55), Medium: (0.6,0.4,0.3), Dark: (0.35,0.2,0.12)
```

### Leather
Chain: Noise(80,6) → Bump(strength=0.3) → Normal.
```
Principled: metallic=0.0, roughness=0.85, base_color=(0.12,0.06,0.03)
```

### Fabric/Cloth
Chain: Noise(50,4) → Roughness (variation 0.7-1.0).
```
Principled: metallic=0.0, roughness=0.9, base_color=(user color)
```

## MINERALS

### Stone/Rock
Chain: Voronoi(5) mixed with Noise(12,8) via MixRGB(0.5) → ColorRamp(dark_grey → light_grey) → Base Color. Noise → Roughness.
```
Principled: metallic=0.0, roughness=0.85
Colors: Granite(0.35,0.33,0.3), Sandstone(0.6,0.5,0.35), Slate(0.2,0.2,0.22)
```

### Crystal/Gem
```
Principled: metallic=0.0, roughness=0.0, transmission=0.95, IOR=2.4
Add Emission(strength=2.0) for magical glow
Colors: Ruby(0.8,0.05,0.05), Emerald(0.05,0.6,0.1), Sapphire(0.05,0.1,0.8), Diamond(0.95,0.95,1.0)
```

### Glass
```
Principled: metallic=0.0, roughness=0.0, transmission=0.95, IOR=1.45
Set material blend_method='HASHED' for Eevee transparency
Colors: Clear(0.9,0.95,1.0), Tinted(0.1,0.3,0.5)
```

## CONSTRUCTION

### Plastic
```
Principled: metallic=0.0, roughness=0.4, specular=0.6
Any bright saturated color works
```

### Ceramic/Porcelain
```
Principled: metallic=0.0, roughness=0.15, base_color=(0.9,0.9,0.88)
Add subtle Subsurface(0.05) for depth
```

### Concrete
Chain: Noise(2,4) → ColorRamp(0.35,0.35,0.33 → 0.55,0.53,0.5) → Base Color. Noise(8) → Bump(0.2) → Normal.
```
Principled: metallic=0.0, roughness=0.95
```

### Brick
Use Brick Texture node → ColorRamp for variation → Base Color. Brick.Fac → Bump → Normal.
```
Principled: metallic=0.0, roughness=0.8, base_color=(0.5,0.2,0.1)
```

## EFFECTS

### Emissive/Glow
```
Principled: emission_color=(R,G,B), emission_strength=5.0
Or: pure Emission node for light-emitting objects
```

### Water Surface
```
Principled: roughness=0.05, transmission=0.8, IOR=1.33, base_color=(0.1,0.3,0.5)
Add Wave Texture → Bump for ripples
```

### Ice
```
Principled: roughness=0.1, transmission=0.7, IOR=1.31, subsurface=0.1
base_color=(0.7,0.85,1.0), subsurface_color=(0.5,0.7,0.9)
```

## APPLYING TO MULTIPLE OBJECTS
When multiple parts should have the SAME material, create it ONCE and assign to all:
```python
mat = create_material("SharedWood", (0.4, 0.25, 0.1), roughness=0.75)[0]
for obj_name in ["Blade", "Guard", "Handle", "Pommel"]:
    obj = bpy.data.objects.get(obj_name)
    if obj:
        assign_material(obj, mat)
```
NEVER create duplicate materials for parts that should look identical.
