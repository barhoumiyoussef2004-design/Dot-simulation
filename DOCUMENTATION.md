# Cosmic Flow - Technical Documentation

This document provides a detailed explanation of the code used in each page of the Cosmic Flow project.

---

## Table of Contents
1. [index.html - Main Menu](#indexhtml---main-menu)
2. [comet.html - Comet](#comethtml---comet)
3. [saturn.html - Saturn](#saturnhtml---saturn)
4. [jupiter.html - Jupiter](#jupiterhtml---jupiter)
5. [venus.html - Venus](#venushtml---venus)
6. [mercury.html - Mercury](#mercuryhtml---mercury)
7. [earth.html - Earth](#earthhtml---earth)
8. [neptune.html - Neptune](#neptunehtml---neptune)
9. [uranus.html - Uranus](#uranushtml---uranus)
10. [mars.html - Mars](#marshtml---mars)
11. [aurora.html - Aurora](#aurorahtml---aurora)
12. [nebula.html - Nebula](#nebulahml---nebula)
13. [galaxy.html - Galaxy](#galaxyhtml---galaxy)
14. [blackhole.html - Black Hole](#blackholehtml---black-hole)
15. [supernova.html - Supernova](#supernovahtml---supernova)
16. [sun.html - Sun](#sunhtml---sun)
17. [storm.html - Storm](#stormhtml---storm)

---

## Common Technologies

### Three.js
All pages use Three.js (version 0.160.0) loaded via ES6 importmap from unpkg CDN. This provides:
- `THREE.Scene` - Container for all 3D objects
- `THREE.PerspectiveCamera` - Camera with perspective projection
- `THREE.WebGLRenderer` - WebGL renderer for hardware-accelerated graphics
- `THREE.BufferGeometry` - Efficient geometry using typed arrays
- `THREE.Points` - Point cloud rendering (particles)
- `THREE.ShaderMaterial` - Custom GLSL shaders

### Post-Processing
All pages use Three.js post-processing pipeline:
- `EffectComposer` - Manages post-processing passes
- `RenderPass` - Renders the base scene
- `UnrealBloomPass` - Bloom/glow effect (makes bright areas glow)

### OrbitControls
All pages include `OrbitControls` for user interaction - allows rotating, zooming, and panning around the scene with mouse/touch.

### BufferGeometry Attributes
Each page creates custom particle systems using BufferGeometry with these standard attributes:
- `position` (Float32Array) - XYZ coordinates
- `color` (Float32Array) - RGB values (0-1 range)
- `size` (Float32Array) - Point size for each particle
- Custom attributes like `aRandom`, `aRadius`, `aAngle` for shader effects

### ShaderMaterial
All pages use custom GLSL shaders:

**Vertex Shader** - Runs once per vertex/particle:
- Transforms positions
- Calculates point size based on distance (perspective scaling)
- Passes varying variables to fragment shader

**Fragment Shader** - Runs once per pixel:
- Draws circular points using `gl_PointCoord`
- Creates glow effect with smoothstep
- Handles alpha/transparency

Common shader techniques:
- `smoothstep(0.0, 0.5, d)` - Creates soft circular glow
- `mix(color1, color2, t)` - Linear interpolation between colors
- `sin/cos` functions - Creates waves and animations

---

## index.html - Main Menu

### Purpose
Landing page with a centered grid of icons to navigate to different cosmic visualizations.

### HTML Structure
- Title element at top with "Cosmic Flow"
- Description below: "Cosmic particle simulation"
- CSS Grid layout (4 columns) with navigation items

### CSS Styling
```css
.grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 40px;
}
.icon {
    width: 80px;
    height: 80px;
    border: 1px solid currentColor;
    border-radius: 50%;
}
```

### Navigation Items
16 items using emoji icons:
- Nebula (☁️), Galaxy (🌀), Aurora (💚), Black Hole (⚫)
- Supernova (💥), Comet (☄️), Sun (☀️), Storm (🌪️)
- Mercury (☿), Venus (♀️), Earth (🌍), Mars (🔴)
- Jupiter (🟤), Saturn (🪐), Uranus (🔵), Neptune (🔵)

No Three.js - pure HTML/CSS landing page.

---

## comet.html - Comet

### Scene Setup
- Camera at position (0, 5, 40) looking at origin
- Bloom pass with high intensity (2.5) for bright glow

### Components

#### 1. Background Stars (8000 particles)
- **Geometry**: Random positions in 300x300x300 cube
- **Colors**: Grayscale with slight blue tint variation
- **Animation**: Subtle parallax movement using sin/cos in vertex shader:
  ```glsl
  pos.x += sin(uTime * 0.3 + aRandom * 10.0) * 2.0;
  pos.y += cos(uTime * 0.2 + aRandom * 8.0) * 1.5;
  pos.z += sin(uTime * 0.25 + aRandom * 6.0) * 1.0;
  ```

#### 2. Nucleus (1000 particles)
- **Distribution**: Spherical (using spherical coordinates)
- **Colors**: White/blue tint with three color variations
- **Animation**: 
  - Pulsing effect: `pos *= sin(uTime * 4.0 + aRandom * 6.28) * 0.1 + 1.0`
  - Flickering alpha based on time

#### 3. Tail (60000 particles)
- **Distribution**: Cone shape extending backward (negative X)
- **Colors**: Gradient from white/blue (near nucleus) to dark blue (far)
- **Animation**:
  - Wave motion: `pos.y += sin(uTime * 3.0 + aOffset)`
  - Fade based on distance from nucleus

#### 4. Trail (3000 particles)
- **Distribution**: Extended trail behind tail
- **Colors**: Dim blue-purple
- **Animation**: Shimmer effect with random offset

### Key Techniques
- `THREE.AdditiveBlending` - Makes overlapping particles brighter
- Point size scaling: `size * uPixelRatio * (200.0 / -mv.z)` for depth-based sizing

---

## saturn.html - Saturn

### Scene Setup
- Camera at (0, 12, 45) - higher angle to see rings
- Moderate bloom (1.2 intensity)

### Components

#### 1. Planet Body (50000 particles)
- **Distribution**: Sphere radius 10
- **Colors**: Jupiter-like bands using latitude-based coloring:
  ```javascript
  const bandPattern = Math.sin(lat * 25);
  if (bandPattern > 0.2) color = tan;
  else if (bandPattern > -0.2) color = brown;
  else color = darker brown;
  ```
- **Poles**: Whiter color at high latitudes (> 0.75)
- **Animation**: Rotation with `atan(pos.z, pos.x)` to calculate angle, then rotate:
  ```glsl
  float angle = atan(pos.z, pos.x);
  float newAngle = angle + t;
  pos.x = cos(newAngle) * r;
  pos.z = sin(newAngle) * r;
  ```

#### 2. Rings (80000 particles)
- **Distribution**: Annulus from radius 14 to 28
- **Colors**: Multiple ring bands with different tan/beige colors
- **Gaps**: Dark gaps at radii 21-22.5 and 24-25 (Cassini Division)
- **Animation**: Same rotation as planet, plus brightness variation

### Unique Features
- Ring gaps created by setting brightness very low (0.15) for specific radius ranges
- Ring thickness: Y-position spread of 1.5 units

---

## jupiter.html - Jupiter

### Scene Setup
- Camera at (0, 0, 35) - equatorial view
- Bloom with intensity 1.2

### Components

#### 1. Planet (60000 particles)
- **Distribution**: Sphere radius 12
- **Colors**: Complex band pattern:
  - High latitudes: White (polar regions)
  - Specific latitude bands: red-brown for Great Red Spot area
  - Main bands: alternating orange-brown using `Math.sin(lat * 18)`
- **Animation**: Rotation at speed 0.08

#### 2. Great Red Spot (5000 particles)
- **Distribution**: Elliptical region at (3.0, 0, 5.0)
- **Colors**: Red-orange (high red channel, low green/blue)
- **Animation**: 
  - Orbits with planet rotation
  - Vertical oscillation: `pos.y += sin(t * 4.0) * 0.15`
  - Rotates faster than planet (t * 0.6)

#### 3. Bands (3000 particles)
- **Distribution**: Thin shell at radius 12.3
- **Colors**: White with slight yellow tint
- **Purpose**: Adds depth and atmosphere effect

### Color Scheme
```javascript
// Polar: white
// Great Red Spot area: red-brown
// Main bands: alternating orange/tan/dark brown
```

---

## venus.html - Venus

### Scene Setup
- Camera at (0, 5, 40)
- High bloom (2.0) for atmospheric glow
- `bloom.threshold = 0.0` - all particles contribute to bloom

### Components

#### 1. Surface (25000 particles)
- **Distribution**: Sphere radius 12
- **Colors**: Yellow-orange based on Y position (latitude):
  ```javascript
  if (y > 0.3) colors = yellow-tan;
  else if (y > -0.3) colors = orange-tan;
  else colors = darker orange;
  ```
- **Static**: No animation in vertex shader

#### 2. Clouds (50000 particles)
- **Distribution**: Shell at radius 12.3-13.8 (outside surface)
- **Colors**: White-yellow
- **Animation**: 
  - Rotation faster than surface (t * 0.5)
  - Vertical wave: `pos.y += sin(t * 3) * 1.5`
  - Alpha variation for shimmer effect

### Unique Aspects
- Venus rotates very slowly (retrograde) - not simulated
- Dense cloud layer creates thick atmosphere appearance

---

## mercury.html - Mercury

### Scene Setup
- Camera at (0, 5, 35)
- Bloom intensity 1.5 with threshold 0.0

### Components

#### 1. Surface (30000 particles)
- **Distribution**: Sphere radius 10-10.5
- **Colors**: Gray with slight variation (0.7-0.9 brightness)
- **Animation**: 
  - Chaotic movement using sin/cos waves
  - `pos.x += sin(t * 1.5 + aRandom * 6.28) * aRandom * 3`
  - Creates "shimmering" rock surface effect

#### 2. Craters (2000 particles)
- **Distribution**: Sphere shell at radius 10.2
- **Colors**: Dark gray (low brightness 0.3-0.6)
- **Static**: No animation, adds surface detail
- **Purpose**: Creates cratered appearance

### Visual Style
- Gray, rocky appearance
- No atmosphere - sharp, clear rendering

---

## earth.html - Earth

### Scene Setup
- Camera at (0, 5, 40)
- Bloom with threshold 0.0

### Components

#### 1. Land/Ocean (40000 particles)
- **Distribution**: Sphere radius 10
- **Colors**: Procedural continents using noise:
  ```javascript
  if (lat > 0.65 || (lat < 0.15 && Math.random() > 0.3)) 
      colors = white; // Polar ice
  else if (sin(lon * 3 + lat * 5) > 0.2 && lat < 0.6)
      colors = green; // Land
  else 
      colors = blue; // Ocean
  ```
- **Static**: No animation

#### 2. Atmosphere (3000 particles)
- **Distribution**: Shell at radius 10.2
- **Colors**: Light blue (0.5, 0.7, 1.0)
- **Static**: Thin glowing rim effect
- **Alpha**: 0.4 - subtle

#### 3. Clouds (35000 particles)
- **Distribution**: Shell at radius 10.4-10.9
- **Colors**: White
- **Animation**:
  - Rotation at different speed than planet (t * 0.3)
  - Chaos-based alpha variation
  - Clouds move independently

### Color Palette
- Ice caps: white
- Land: green (0.2, 0.6, 0.3)
- Ocean: blue (0.1, 0.3, 0.7)
- Clouds: white with alpha

---

## neptune.html - Neptune

### Scene Setup
- Camera at (0, 5, 45)
- Bloom with threshold 0.0

### Components

#### 1. Planet (40000 particles)
- **Distribution**: Sphere radius 11
- **Colors**: Blue based on latitude:
  ```javascript
  if (lat > 0.7) colors = lighter blue; // Pole
  else colors = deep blue; // Main body
  ```
- **Animation**: Rotation with chaos:
  ```glsl
  float chaos = sin(t * 2.0 + aRandom * 8.0) * 0.5 + 0.5;
  float newAngle = angle + t * 0.25 + aRandom;
  ```

#### 2. Storms (6000 particles)
- **Distribution**: Scattered storms at various radii (1-5 from center)
- **Colors**: Bright cyan-blue
- **Animation**:
  - Individual rotation speeds: `rotSpeed = 0.1 / (aDistance * 0.1 + 0.5)`
  - Vertical oscillation
  - Uses AdditiveBlending for glow

### Visual Style
- Deep blue color ( Neptune's methane atmosphere)
- Storms appear as bright spots

---

## uranus.html - Uranus

### Scene Setup
- Camera at (0, 8, 45) - tilted to see rings
- Bloom with threshold 0.0

### Components

#### 1. Planet (35000 particles)
- **Distribution**: Sphere radius 11
- **Colors**: Cyan-green (0.7, 0.95, 1.0) - Uranus's distinctive color
- **Animation**: Slow rotation (0.25 speed)

#### 2. Rings (25000 particles)
- **Distribution**: Flat annulus radius 15-23
- **Colors**: Pale cyan
- **Animation**: Very slow rotation
- **Y-spread**: Only 0.3 units - thin rings

### Unique Features
- Cyan color from methane in atmosphere
- Rings lie in equatorial plane
- Very subtle animation

---

## mars.html - Mars

### Scene Setup
- Camera at (0, 5, 40)
- Bloom intensity 1.5

### Components

#### 1. Surface (40000 particles)
- **Distribution**: Sphere radius 9-9.5
- **Colors**: 
  - Polar regions (|lat| > 0.75): white (ice caps)
  - Rest: red-orange-brown
- **Animation**: 
  - Chaotic surface movement
  - Shimmering effect

#### 2. Polar Caps (3000 particles)
- **Distribution**: Caps at top and bottom of sphere
- **Phi range**: 0-0.3 (north) and PI-0.3 to PI (south)
- **Colors**: White (0.9, 0.9, 0.95)
- **Static**: No animation

### Color Scheme
- Mars red: (shade * 0.7, shade * 0.4, shade * 0.3)
- Polar ice: white

---

## aurora.html - Aurora

### Scene Setup
- Camera at (0, 5, 40)
- Wide FOV (60 degrees)
- High bloom (2.5)

### Components

#### 1. Ground (500 particles)
- **Distribution**: Flat plane at Y = -5 to -7
- **Colors**: Very dark green (0.05, 0.08, 0.05)
- **Purpose**: Horizon reference

#### 2. Stars (3000 particles)
- **Distribution**: Background in Y range 5-35
- **Colors**: White with slight variation
- **Static**: No animation

#### 3. Aurora Curtains (50000 particles)
- **Distribution**: Vertical curtains in X range -30 to 30, height 5-30
- **Colors**: Height-based gradient:
  ```javascript
  if (height < 12) green;
  else if (height < 18) cyan;
  else if (height < 23) purple;
  else pink;
  ```
- **Animation**:
  - Wave motion: `pos.x += sin(uTime * 0.8 + aXPos * 0.15) * 5.0`
  - Vertical curtain wave
  - Shimmer: high-frequency sin for flickering
  - Alpha varies with time and height

#### 4. Rays (8000 particles)
- **Distribution**: Vertical rays from Y=0 to Y=35
- **Colors**: Green at bottom, purple at top
- **Animation**:
  - Horizontal drift
  - Flicker effect

### Visual Effect
- Creates northern lights curtain effect
- Colors transition green -> cyan -> purple -> pink with altitude

---

## nebula.html - Nebula

### Scene Setup
- Camera at (0, 15, 45)
- Fog: `THREE.FogExp2(0x000000, 0.002)` for depth
- High bloom (2.5)

### Components

#### 1. Background Stars (5000 particles)
- **Distribution**: Large cube 300x300x300
- **Static**: No animation

#### 2. Nebula Dust (60000 particles)
- **Distribution**: Three cloud types:
  - 40%: Spherical clouds (r up to 30)
  - 30%: Torus-like shapes
  - 30%: Spiral arm structures
- **Colors**: Four color types:
  - Pink/magenta (0.9, 0.2, 0.5)
  - Purple (0.6, 0.15, 0.85)
  - Orange (1, 0.4, 0.3)
  - Blue (0.3, 0.6, 1)
- **Animation**:
  - Swirl: `sin(length(pos.xz) * 0.08 + t)` rotation
  - Breathing: `sin(t * 0.5) * 0.3 + 1.0` scale
  - Wave: vertical movement based on X position
- **Blending**: AdditiveBlending for glowing effect

### Shader Techniques
```glsl
// Swirl rotation
float angle = atan(pos.z, pos.x);
float newAngle = angle + swirl * 0.02;

// Depth-based alpha
float depth = 1.0 - length(pos) / 40.0;
vAlpha = (0.4 + aRandom * 0.6) * (0.5 + depth * 0.5);
```

---

## galaxy.html - Galaxy

### Scene Setup
- Camera at (0, 50, 60) - top-down angle
- Uses `THREE.ACESFilmicToneMapping` for better HDR
- Bloom with threshold 0.05

### Components

#### 1. Spiral Arms (100000 particles = 4 arms × 25000)
- **Distribution**: 4 spiral arms using logarithmic spiral:
  ```javascript
  const spiralAngle = baseAngle + radius * 0.12 + spreadAngle;
  positions[i] = cos(spiralAngle) * (radius + offset);
  ```
- **Colors**: Based on radius and star type:
  - Core (t < 0.15): White-yellow
  - 30%: Orange (cool stars)
  - 30%: Blue (hot stars)
  - 40%: White
- **Animation**:
  - Differential rotation: inner stars rotate faster
  - `rotSpeed = 0.015 / (aRadius * 0.02 + 0.3)`
  - Vertical wave: `pos.y += sin(t * 1.5 + aRadius * 0.1)`

#### 2. Bulge (15000 particles)
- **Distribution**: Sphere radius 0-12 (dense center)
- **Colors**: Yellow-white
- **Static**: No animation

#### 3. Halo (10000 particles)
- **Distribution**: Large sphere radius 15-60
- **Colors**: Faint yellow-white, fading with distance
- **Static**: No animation

### Galaxy Structure
- 4 spiral arms with logarithmic twist
- Dense bright core (bulge)
- Faint extended halo
- Colors represent different star types

---

## blackhole.html - Black Hole

### Scene Setup
- Camera at (0, 5, 50)
- Very high bloom (3.0)
- OrbitControls with min/max distance limits (20-100)

### Components

#### 1. Background Stars (4000 particles)
- **Distribution**: Sphere radius 50-150
- **Static**: No animation

#### 2. Event Horizon (Mesh)
- **Geometry**: `THREE.SphereGeometry(4, 64, 64)` - solid black sphere
- **Material**: `MeshBasicMaterial({ color: 0x000000 })` - pure black
- **Purpose**: The black hole itself - absorbs light

#### 3. Accretion Disk (60000 particles)
- **Distribution**: Flat disk radius 5-33
- **Thickness**: Varies with radius (thicker near center)
- **Colors**: Temperature-based gradient (hotter = brighter/whiter):
  ```javascript
  if (temp > 0.75) white;
  else if (temp > 0.5) yellow;
  else if (temp > 0.25) orange;
  else purple;
  ```
- **Animation**:
  - Differential rotation (inner faster)
  - Spiral waves
  - Vertical wobble
- **Blending**: AdditiveBlending for glow

### Unique Features
- Black sphere in center (event horizon)
- Colorful accretion disk represents temperature
- Doppler effect simulated (brighter on approaching side concept)

---

## supernova.html - Supernova

### Scene Setup
- Camera at (0, 5, 50)
- Very high bloom (3.5)

### Components

#### 1. Core (500 particles)
- **Distribution**: Small sphere radius 0-1
- **Colors**: Bright white-yellow
- **Animation**: Pulsing scale

#### 2. Blast Wave (5000 particles)
- **Distribution**: Spherical shell expanding outward
- **Animation**:
  - `expand = uTime * 8.0` - linear expansion
  - `fade = max(0.0, 1.0 - expand / 30.0)` - fades over time
  - Creates expanding ring effect

#### 3. Debris (40000 particles)
- **Distribution**: Expanding sphere
- **Colors**: Various (white, orange, red, purple)
- **Animation**:
  - Outward expansion: `dist = t * 3.0`
  - Chaotic motion with sin/cos
  - Fade: `max(0.0, 1.0 - t / 8.0)`

#### 4. Ejecta (10000 particles)
- **Distribution**: Very fast expanding particles
- **Colors**: White
- **Animation**:
  - Fast expansion: `spread = t * 5.0`
  - Flickering alpha

### Visual Effect
- Continuous expansion loop
- Creates explosion outward effect
- High particle count for dense explosion look

---

## sun.html - Sun

### Scene Setup
- Camera at (0, 5, 45)
- Very high bloom (3.5)
- OrbitControls limits 25-80

### Components

#### 1. Surface (15000 particles)
- **Distribution**: Sphere radius 12
- **Colors**: Granule pattern:
  - 30%: Bright yellow
  - 35%: Medium yellow
  - 35%: Orange
- **Animation**:
  - Convection: `sin(uTime * 2.0 + position.x * 0.5)`
  - Pulsing scale

#### 2. Corona (40000 particles)
- **Distribution**: Sphere shell radius 12.5-32.5
- **Colors**: Temperature gradient (white->yellow->red)
- **Animation**:
  - Streaming: `sin(uTime * 0.5 + aAngle * 2.0) * 3.0`
  - Alpha fades with distance

#### 3. Prominences (10000 particles)
- **Distribution**: Loops extending from surface
- **Colors**: Red-orange (cooler = redder)
- **Animation**:
  - Surge effect: `sin(uTime * 1.5 + aHeight * 0.5)`
  - Flickering

#### 4. Flares (5000 particles)
- **Distribution**: Random positions at various radii
- **Colors**: White-yellow
- **Animation**:
  - Random flickering
  - `flare = sin(uTime * 8.0 + aRandom * 15.0)`

### Visual Style
- Most luminous object
- Multiple layered effects
- AdditiveBlending throughout for brightness

---

## storm.html - Storm

### Scene Setup
- Camera at (0, 0, 40)
- High bloom (2.5)

### Components

#### 1. Storm Particles (80000 particles)
- **Distribution**: Flattened sphere (Y compressed by 0.5)
- **Colors**: Three types:
  - 30%: Warm white
  - 30%: Purple
  - 40%: Red-orange
- **Animation**:
  - Rotation: `angle = t * 2.0 + aAngle`
  - Radius expansion: `newRadius = radius + sin(t + aRandom)`
  - Vertical chaos: `pos.y += sin(t * 4.0 + radius * 0.2)`
  - Overall chaos factor

### Shader Logic
```glsl
// Main rotation
float angle = t * 2.0 + aAngle;
pos.x = cos(angle) * newRadius + sin(t * 2.0 + aRandom) * 5.0;
pos.z = sin(angle) * newRadius + cos(t * 2.0 + aRandom) * 5.0;

// Vertical movement
pos.y += sin(t * 4.0 + radius * 0.2) * (2.0 + aRandom * 3.0);
```

### Visual Effect
- Chaotic, swirling storm
- Multiple colors mixed
- Creates turbulent atmosphere look

---

## Summary of Techniques Used

### Particle Distribution Methods
1. **Spherical**: Using spherical coordinates (phi, theta)
2. **Disk/Torus**: Using radius and angle
3. **Spiral**: Logarithmic spiral equations
4. **Shell**: Surface at specific radius
5. **Random cube**: Simple random XYZ
6. **Cylinder**: Radius + height

### Animation Techniques
1. **Rotation**: Using atan2 for angle, adding time, recalculating XZ
2. **Wave**: sin/cos with position or random offset
3. **Expansion**: Multiply position by time factor
4. **Chaos**: Combining multiple sin waves with random factors
5. **Pulse**: Scale oscillation using sin(time)

### Color Techniques
1. **Gradient**: Color changes based on position (radius, height, latitude)
2. **Temperature**: Hot (white/yellow) to cool (red/purple)
3. **Mix**: Multiple color types with random selection
4. **Brightness**: Multiplier for variation

### Rendering Techniques
1. **AdditiveBlending**: For glowing/emissive effects
2. **NormalBlending**: For opaque objects
3. **depthWrite: false**: For transparent particles
4. **Point size scaling**: Based on distance for perspective

### Post-Processing
- **UnrealBloomPass**: Creates glow effect
- **threshold**: Controls which brightness levels contribute
- **intensity/radius**: Control bloom strength and spread

---

*Generated for Cosmic Flow project - March 2026*
