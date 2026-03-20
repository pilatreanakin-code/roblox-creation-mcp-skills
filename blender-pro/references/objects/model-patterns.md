# Model Patterns — Design Principles (NOT step-by-step recipes)

These are PRINCIPLES and QUESTIONS to guide your blueprint. Do NOT copy these as build steps. Every model must be designed uniquely based on the user's answers.

## CHARACTERS / CREATURES / PETS

### Proportion Guides (adapt based on user's style choice)
- **Chibi/Cute (2-3 heads tall):** Head = 40% height, body round, limbs stubby, eyes = 30% of head
- **Stylized (4-5 heads):** Head = 25%, broad shoulders, thin waist, medium limbs
- **Realistic (7-8 heads):** Head = 12%, standard anatomy, detailed hands
- **Pet (cute):** Head = 50-60% of total, body tiny and round, eyes huge, limbs tiny or absent

### Design Questions to Ask
- Species/creature type? Humanoid, animal, fantasy, robot, hybrid?
- Body shape? Round, slim, muscular, lanky, stocky?
- Head shape? Round, angular, long, flat, triangular?
- Eyes? Big/small, round/almond, how many, what color, glow?
- Ears? Pointy, floppy, round, horns instead, none?
- Limbs? How many, long/short, thick/thin, what do they end in (hands, paws, claws, hooves)?
- Tail? Type, length, bushy/thin/armored?
- Clothing/accessories? Hat, armor, scarf, belt, jewelry?
- Expression/mood? Happy, fierce, sleepy, curious?
- Color palette? Warm/cool, how many colors, any patterns?

### Techniques to Consider
- Mirror modifier for symmetry (most characters are symmetric)
- Subdivision Surface for organic shapes
- Separate objects for eyes (always — never model eyes into the head mesh)
- Solidify modifier for clothing shells
- BMesh vertex editing for fine shaping

## VEHICLES / MACHINES

### Design Questions to Ask
- Vehicle type? Car, truck, motorcycle, spaceship, boat, mech, aircraft, cart?
- Era? Ancient, medieval, modern, futuristic, steampunk, post-apocalyptic?
- Shape language? Rounded/aerodynamic or boxy/angular?
- How many wheels/legs/thrusters? Size relative to body?
- Windows? Large/small, tinted, visor-style, portholes?
- Special features? Weapons, cargo, lights, exhaust flames, spoiler, antenna?
- Damage/wear? New and clean, battle-scarred, rusty, weathered?
- Color scheme? Solid, two-tone, camo, racing stripes?

### Techniques to Consider
- Boolean for wheel arches, windows, panel cutouts
- Mirror modifier for left/right symmetry
- Array for repeated elements (rivets, panels, exhaust pipes)
- Bevel modifier with angle limit for panel edges
- Torus for tires, cylinders for rims

## BUILDINGS / ARCHITECTURE

### Design Questions to Ask
- Building type? House, castle, tower, shop, temple, ruin, skyscraper, hut?
- Architectural style? Medieval, modern, fantasy, sci-fi, ancient, rustic, art deco?
- How many floors? Approximate height?
- Roof type? Pitched, flat, dome, pagoda, thatched, turrets?
- Windows? Shape (rectangular, arched, circular, narrow slits), how many, pattern?
- Door? Size, style (wooden, glass, gate, drawbridge)?
- Materials? Stone, wood, brick, glass, metal, mixed?
- Surroundings? Garden, balcony, chimney, porch, fence, path?
- Condition? New, old, ruined, overgrown?
- Interior needed or exterior only?

### Techniques to Consider
- Solidify for wall thickness
- Boolean for windows and doors
- Array for window rows, fence posts, bricks
- Separate roof object (easier to shape independently)
- Bevel for stone/brick edge detail

## PROPS / ITEMS / WEAPONS

### Design Questions to Ask for Weapons
- Weapon type? Sword, axe, hammer, staff, bow, dagger, spear, scythe?
- Blade shape? Straight/curved, single/double-edged, wide/narrow, serrated?
- Blade length relative to handle? Short dagger ratio vs long greatsword ratio?
- Guard/crossguard? Shape (cross, disc, S-curve, basket, none)?
- Handle? Length (one-hand, two-hand), wrap style (leather, wire, bare)?
- Pommel? Shape (sphere, diamond, skull, gem, flat)?
- Any decorations? Engravings, runes, gems embedded, glowing effects?
- Material? Metal, wood, bone, crystal, obsidian, magical energy?
- Condition? Pristine, battle-worn, ancient, ceremonial?

### Design Questions to Ask for General Props
- What is the object? Be specific about real-world reference
- Size relative to a character? Handheld, furniture-sized, building-sized?
- Material? Wood, metal, stone, glass, fabric, magical, organic?
- Condition? New, old, worn, broken, enchanted?
- Any moving parts or special features?
- Any glow, particles, or magical effects?
- Functional or decorative?

### Techniques to Consider
- Start from closest primitive to final shape
- Subdivision + edge flow for smooth shapes
- Boolean for cutouts and inlays
- Ico Sphere with flat shading for gems/crystals
- Separate objects for distinct material parts

## ENVIRONMENTS / TERRAIN / MAPS

### Design Questions to Ask
- Biome? Forest, desert, snow, ocean, volcanic, swamp, city, space, underground?
- Size? Small area (room), medium (yard/garden), large (landscape)?
- Time of day? Dawn, midday, sunset, night, overcast?
- Mood? Peaceful, eerie, epic, cozy, dangerous, mysterious?
- Key landmarks? Any specific features (mountain, river, bridge, ruins, giant tree)?
- Vegetation level? Dense, sparse, dead, alien?
- Water? River, lake, ocean, waterfall, puddles, none?
- Paths/roads? Dirt, stone, paved, none?
- Props to include? Rocks, fences, signs, lampposts, benches, crates?
- For a game? What's the player doing here (exploring, fighting, building)?

### Techniques to Consider
- Grid/Plane with Subdivision + Displace modifier for terrain
- Texture-based displacement (Clouds, Musgrave) for natural hills
- Separate water plane with transparent/refractive material
- Batch-create trees/rocks with randomized sizes and positions
- Bezier curves with bevel for paths/roads
- Array modifier for fences, street lights, building rows

## UNIVERSAL ASSEMBLY — Bounding Box Snap (use for ALL models)
```python
from mathutils import Vector

def get_bounds(obj):
    bbox = [obj.matrix_world @ Vector(c) for c in obj.bound_box]
    return {
        'min_z': min(v.z for v in bbox),
        'max_z': max(v.z for v in bbox),
        'height': max(v.z for v in bbox) - min(v.z for v in bbox),
        'center_x': (min(v.x for v in bbox) + max(v.x for v in bbox)) / 2,
        'center_y': (min(v.y for v in bbox) + max(v.y for v in bbox)) / 2,
    }

def stack_on_top(base_obj, top_obj, overlap):
    """Position top_obj on base_obj. overlap = how many Blender units to sink in."""
    base = get_bounds(base_obj)
    top = get_bounds(top_obj)
    top_obj.location.z += base['max_z'] - top['min_z'] - overlap

def stack_below(top_obj, bottom_obj, overlap):
    """Position bottom_obj below top_obj. overlap = units of penetration."""
    top = get_bounds(top_obj)
    bot = get_bounds(bottom_obj)
    bottom_obj.location.z += top['min_z'] - bot['max_z'] + overlap

def center_on(target_obj, moving_obj):
    """Center moving_obj on target_obj's X and Y"""
    t = get_bounds(target_obj)
    m = get_bounds(moving_obj)
    moving_obj.location.x += t['center_x'] - m['center_x']
    moving_obj.location.y += t['center_y'] - m['center_y']
```

**OVERLAP — HOW TO DECIDE:**
The `overlap` value is NOT a fixed number. The AI must decide it for each connection by thinking:

- **How would this joint look in real life?** A gem is pressed deep into a socket. A blade passes through the guard. A handle is wedged firmly into the hilt. Nothing in reality just rests perfectly flush.
- **Smaller part into larger part = more overlap.** A tiny gem into a big pommel needs significant embedding. Two similarly-sized parts need less.
- **Structural connections = more overlap.** Handle into guard = deep. Decorative element sitting on top = shallow.
- **When in doubt, try a value, take a screenshot, and adjust.** If it looks like it's floating, increase. If it looks like it's clipping too much, decrease. Iterate visually.

The AI should ALWAYS screenshot after assembly and ask itself: "Do these connections look solid and natural, like a real physical object? Or does anything look like it's just balanced on top?" If the latter, fix it before showing the user.
