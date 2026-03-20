# Shader Nodes — Complete Reference

Setup: `mat = bpy.data.materials.new("Name"); mat.use_nodes = True; nodes = mat.node_tree.nodes; links = mat.node_tree.links`

## SHADER NODES (output BSDF)

### Principled BSDF (USE THIS 99% OF THE TIME)
```python
p = nodes.get("Principled BSDF")  # already exists by default
p.inputs['Base Color'].default_value = (R, G, B, 1.0)  # 0-1 range
p.inputs['Metallic'].default_value = 0.0      # 0=non-metal, 1=metal
p.inputs['Roughness'].default_value = 0.5     # 0=mirror, 1=matte
p.inputs['Specular IOR Level'].default_value = 0.5
p.inputs['Subsurface Weight'].default_value = 0.0  # skin/wax: 0.1-0.5
p.inputs['Subsurface Color'].default_value = (R, G, B, 1)
p.inputs['Emission Color'].default_value = (R, G, B, 1)
p.inputs['Emission Strength'].default_value = 0.0
p.inputs['Alpha'].default_value = 1.0         # 0=invisible, 1=opaque
p.inputs['Transmission Weight'].default_value = 0.0  # glass: 0.95
p.inputs['IOR'].default_value = 1.45           # glass index of refraction
# Connect Normal input for bump/normal maps
```

### Other Shaders (less common, use Principled instead when possible)
```python
nodes.new('ShaderNodeBsdfDiffuse')       # pure diffuse (Lambert)
nodes.new('ShaderNodeBsdfGlossy')        # pure mirror/reflection
nodes.new('ShaderNodeBsdfGlass')         # refractive glass
nodes.new('ShaderNodeEmission')          # pure light emitter
nodes.new('ShaderNodeBsdfTransparent')   # invisible (for mixing)
nodes.new('ShaderNodeBsdfTranslucent')   # backlit thin surfaces
nodes.new('ShaderNodeBsdfVelvet')        # fabric
nodes.new('ShaderNodeBsdfToon')          # cartoon cel-shading
nodes.new('ShaderNodeSubsurfaceScattering')  # wax, skin, leaves
```

### Mix/Add Shaders
```python
mix = nodes.new('ShaderNodeMixShader')
mix.inputs['Fac'].default_value = 0.5  # blend factor
links.new(shader1.outputs[0], mix.inputs[1])
links.new(shader2.outputs[0], mix.inputs[2])

add = nodes.new('ShaderNodeAddShader')  # additive blend
```

## TEXTURE NODES (generate patterns)

### Noise Texture (organic variation, clouds, dirt)
```python
noise = nodes.new('ShaderNodeTexNoise')
noise.inputs['Scale'].default_value = 5.0       # feature size
noise.inputs['Detail'].default_value = 6.0       # complexity
noise.inputs['Roughness'].default_value = 0.5    # sharpness
noise.inputs['Distortion'].default_value = 0.0   # warping
# Outputs: 'Fac' (grayscale), 'Color' (colored)
```

### Voronoi Texture (cells, scales, cracks, stone)
```python
voronoi = nodes.new('ShaderNodeTexVoronoi')
voronoi.inputs['Scale'].default_value = 8.0
voronoi.feature = 'F1'         # F1=cells, F2=ridges, SMOOTH_F1=smooth
voronoi.distance = 'EUCLIDEAN' # or MANHATTAN, CHEBYCHEV, MINKOWSKI
# Outputs: 'Distance', 'Color', 'Position'
```

### Musgrave Texture (terrain, natural surfaces)
```python
musgrave = nodes.new('ShaderNodeTexMusgrave')
musgrave.musgrave_type = 'FBM'  # FBM, MULTIFRACTAL, RIDGED_MULTIFRACTAL, HYBRID_MULTIFRACTAL, HETERO_TERRAIN
musgrave.inputs['Scale'].default_value = 3.0
musgrave.inputs['Detail'].default_value = 8.0
musgrave.inputs['Lacunarity'].default_value = 2.0
```

### Wave Texture (wood grain, stripes, rings)
```python
wave = nodes.new('ShaderNodeTexWave')
wave.wave_type = 'BANDS'  # or 'RINGS'
wave.bands_direction = 'X'
wave.inputs['Scale'].default_value = 5.0
wave.inputs['Distortion'].default_value = 3.0
wave.inputs['Detail'].default_value = 4.0
```

### Checker Texture (checkerboard)
```python
checker = nodes.new('ShaderNodeTexChecker')
checker.inputs['Scale'].default_value = 5.0
checker.inputs['Color1'].default_value = (0, 0, 0, 1)
checker.inputs['Color2'].default_value = (1, 1, 1, 1)
```

### Brick Texture (walls)
```python
brick = nodes.new('ShaderNodeTexBrick')
brick.inputs['Scale'].default_value = 3.0
brick.inputs['Color1'].default_value = (0.5, 0.2, 0.1, 1)
brick.inputs['Color2'].default_value = (0.6, 0.3, 0.15, 1)
brick.inputs['Mortar'].default_value = (0.3, 0.3, 0.3, 1)
brick.inputs['Mortar Size'].default_value = 0.02
```

### Gradient Texture
```python
gradient = nodes.new('ShaderNodeTexGradient')
gradient.gradient_type = 'LINEAR'  # LINEAR, QUADRATIC, EASING, DIAGONAL, SPHERICAL, QUADRATIC_SPHERE, RADIAL
```

### Image Texture (loaded from file)
```python
tex = nodes.new('ShaderNodeTexImage')
tex.image = bpy.data.images.load("/path/to/image.png")
# Output: 'Color', 'Alpha'
```

## COLOR NODES

### Color Ramp (map values to colors — EXTREMELY USEFUL)
```python
ramp = nodes.new('ShaderNodeValToRGB')
ramp.color_ramp.elements[0].position = 0.0
ramp.color_ramp.elements[0].color = (0.1, 0.1, 0.1, 1)
ramp.color_ramp.elements[1].position = 1.0
ramp.color_ramp.elements[1].color = (0.9, 0.9, 0.9, 1)
# Add more stops:
elem = ramp.color_ramp.elements.new(0.5)
elem.color = (0.5, 0.3, 0.1, 1)
```

### MixRGB
```python
mix = nodes.new('ShaderNodeMixRGB')
mix.blend_type = 'MIX'  # MIX, ADD, MULTIPLY, SCREEN, OVERLAY, SUBTRACT, etc.
mix.inputs['Fac'].default_value = 0.5
mix.inputs['Color1'].default_value = (1, 0, 0, 1)
mix.inputs['Color2'].default_value = (0, 0, 1, 1)
```

### Hue/Saturation/Value
```python
hsv = nodes.new('ShaderNodeHueSaturation')
hsv.inputs['Hue'].default_value = 0.5
hsv.inputs['Saturation'].default_value = 1.0
hsv.inputs['Value'].default_value = 1.0
```

### Bright/Contrast, Gamma, Invert
```python
nodes.new('ShaderNodeBrightContrast')
nodes.new('ShaderNodeGamma')
nodes.new('ShaderNodeInvert')
```

## VECTOR NODES

### Bump (fake surface detail without geometry)
```python
bump = nodes.new('ShaderNodeBump')
bump.inputs['Strength'].default_value = 0.3
bump.inputs['Distance'].default_value = 0.1
# Connect: Texture.Fac → Bump.Height, Bump.Normal → Principled.Normal
```

### Normal Map (from image)
```python
nmap = nodes.new('ShaderNodeNormalMap')
nmap.inputs['Strength'].default_value = 1.0
```

### Mapping (transform texture coordinates)
```python
mapping = nodes.new('ShaderNodeMapping')
mapping.inputs['Location'].default_value = (0, 0, 0)
mapping.inputs['Rotation'].default_value = (0, 0, 0)
mapping.inputs['Scale'].default_value = (1, 1, 1)
```

### Texture Coordinate (source of UV/position data)
```python
coord = nodes.new('ShaderNodeTexCoord')
# Outputs: 'Generated', 'Normal', 'UV', 'Object', 'Camera', 'Window', 'Reflection'
# Most common: coord.outputs['UV'] or coord.outputs['Object']
```

## CONVERTER NODES

### Math
```python
math = nodes.new('ShaderNodeMath')
math.operation = 'MULTIPLY'  # ADD, SUBTRACT, MULTIPLY, DIVIDE, POWER, SINE, COSINE, etc.
math.inputs[0].default_value = 1.0
math.inputs[1].default_value = 0.5
```

### Map Range
```python
mr = nodes.new('ShaderNodeMapRange')
mr.inputs['Value'].default_value = 0.5
mr.inputs['From Min'].default_value = 0.0
mr.inputs['From Max'].default_value = 1.0
mr.inputs['To Min'].default_value = 0.2
mr.inputs['To Max'].default_value = 0.8
```

### Separate/Combine XYZ, RGB, HSV
```python
sep = nodes.new('ShaderNodeSeparateXYZ')  # or SeparateRGB, SeparateHSV
comb = nodes.new('ShaderNodeCombineXYZ')  # or CombineRGB, CombineHSV
```

## INPUT NODES

### Fresnel (edge glow effect)
```python
fresnel = nodes.new('ShaderNodeFresnel')
fresnel.inputs['IOR'].default_value = 1.45
# Output: 'Fac' (1 at edges, 0 at center — use to mix shaders)
```

### Layer Weight
```python
lw = nodes.new('ShaderNodeLayerWeight')
lw.inputs['Blend'].default_value = 0.5
# Outputs: 'Fresnel', 'Facing'
```

### Object Info, Geometry, Camera Data, Light Path, Ambient Occlusion
```python
nodes.new('ShaderNodeObjectInfo')    # Random, Location, Object Index
nodes.new('ShaderNodeNewGeometry')   # Position, Normal, Tangent, True Normal
nodes.new('ShaderNodeCameraData')    # View Vector, View Z Depth
nodes.new('ShaderNodeLightPath')     # Is Camera Ray, Is Shadow Ray, Ray Depth
nodes.new('ShaderNodeAmbientOcclusion')  # Color, AO
```

## CONNECTING NODES
```python
# Always: output_socket → input_socket
links.new(noise.outputs['Fac'], ramp.inputs['Fac'])
links.new(ramp.outputs['Color'], principled.inputs['Base Color'])
links.new(bump.outputs['Normal'], principled.inputs['Normal'])
links.new(principled.outputs['BSDF'], nodes['Material Output'].inputs['Surface'])
```
