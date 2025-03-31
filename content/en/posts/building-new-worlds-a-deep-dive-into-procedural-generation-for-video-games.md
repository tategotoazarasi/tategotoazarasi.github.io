---
date: '2025-03-28T19:04:14+08:00'
draft: false
title: 'Building New Worlds: A Deep Dive into Procedural Generation for Video Games'
tags: [ "procedural-content-generation","video-game-worlds","terrain-generation","noise-functions","perlin-noise","simplex-noise","fractal-noise","heightmaps","plate-tectonics","erosion-simulation","hydraulic-erosion","thermal-erosion","river-networks","climate-simulation","biomes","map-geometry","tiling-systems","data-structures","game-development","world-building","pcg","game-design","procedural-generation","terrain-algorithms","noise","geology-simulation","climate-modeling","biome-distribution","voronoi-diagrams","hex-grids","worldengine","mapgen4","tectonics-js","minecraft","no-mans-sky","dwarf-fortress","cplusplus","future-trends","interactive-worlds","digital-worldsmithing" ]
params:
  math: true
---

Ever stepped into a vast, sprawling game world and wondered, "How did they *build* all this?" From the infinite blocky
landscapes of *Minecraft* to the galaxy-spanning planets of *No Man's Sky* or the intricate simulated histories of
*Dwarf Fortress*, the answer often lies in a fascinating field: **Procedural Content Generation (PCG)**.

Instead of hand-crafting every mountain, river, and cave, developers use algorithms – sets of rules and instructions –
to generate game content automatically. This isn't just about saving time (though it certainly helps!); it's about
creating experiences that feel boundless, unique, and endlessly replayable. Imagine exploring a world that's different
every single time you start a new game, a world generated just for you, with its own unique geography, climate, and
maybe even history. That's the power and allure of PCG.

This post is a deep dive into the technologies, algorithms, and methods used to generate these fictional worlds,
focusing primarily on the large-scale environmental aspects: the maps, the terrain, the climate, and the biomes that
bring these virtual places to life. We'll cover:

* **The Fundamentals:** What PCG is, the magic of seeds, and the crucial role of noise functions (Perlin, Simplex, and
  friends).
* **Shaping the World:** Different map geometries (flat vs. spherical) and tiling systems (squares, hexes, irregular
  shapes).
* **Raising the Land:** Techniques for creating continents and oceans, from simple noise heightmaps to sophisticated
  plate tectonics simulations.
* **Carving the Details:** How erosion (water and thermal) and river systems turn basic terrain into believable
  landscapes.
* **Breathing Life into It:** Simulating climate (temperature, rainfall, wind) and distributing biomes (deserts,
  forests, tundras).
* **Under the Hood:** The data structures used to represent these complex worlds.
* **Learning from the Masters:** Quick looks at how systems like WorldEngine, Mapgen4, and Tectonics.js put these ideas
  into practice.
* **Code Corner:** Practical C++ snippets illustrating core concepts like noise and erosion.
* **The Horizon:** Where world generation might be heading next.

Whether you're a seasoned game developer, an aspiring world-builder, or just curious about the magic behind your
favorite games, grab a virtual pickaxe, and let's dig in!

## The Building Blocks: Fundamentals of Procedural World Generation

At its heart, procedural generation is about using algorithms to create content instead of manually authoring every
detail. Think of it like giving the computer a recipe rather than a finished cake. The recipe (the algorithm) defines
the steps, and the computer follows them to bake a unique cake (the game world) each time, potentially with slight
variations based on the ingredients (parameters and randomness).

### The Seed of Creation

A key concept is the **seed**. Most procedural generation relies on pseudorandom number generators (PRNGs). These
algorithms produce sequences of numbers that *appear* random but are actually deterministic. If you start a PRNG with
the same initial value, called the *seed*, it will always produce the exact same sequence of numbers.

In game development, this is incredibly powerful. You can generate a massive, complex world using algorithms driven by a
PRNG. Instead of storing gigabytes of map data, you just need to store the generation algorithms and a single seed
number (often just a 32-bit or 64-bit integer). When a player wants to play that specific world again or share it with a
friend, they just need the seed. This is how games like *Minecraft* allow players to share specific world layouts [1].

### Why Go Procedural?

The benefits are compelling:

#### Content Variety & Replayability

Generate near-infinite unique worlds, levels, or items, keeping the experience fresh [1].

#### Development Efficiency & Scalability

Create vast amounts of content with potentially less manual effort, allowing small teams to build large worlds.

#### Reduced Storage/Download Size

Store the generator (code) and a seed, not the massive output data [1].

#### Emergent Possibilities

Complex interactions between simple procedural rules can lead to unexpected and interesting results.

### The Challenges

Of course, it's not magic. Getting PCG right involves challenges:

#### Coherence and Quality

Ensuring generated content makes sense, looks good, and is playable. Randomness needs structure.

#### Avoiding Repetition

Making sure the generated content doesn't feel monotonous or obviously algorithmic.

#### Artistic Control

Giving designers enough control to guide the generation towards a specific vision, rather than accepting whatever the
algorithm spits out. This often involves hybrid approaches, where procedural elements are combined with hand-authored
content or guided by designer-specified constraints.

#### Debugging

Finding bugs in content that only appears under certain random seeds can be tricky.

### Noise: The Canvas of Creation

One of the most fundamental tools in the procedural generation toolbox, especially for terrain and textures, is **noise
**. We're not talking about audio noise, but rather mathematical functions that generate pseudo-random, yet structured,
patterns. Unlike pure `rand()`, which gives unrelated values at each point, noise functions produce values that vary
smoothly across space.

#### Perlin Noise

Developed by Ken Perlin in the 1980s (earning him an Academy Award!), Perlin noise is a type of *gradient noise*. It
works by setting up a grid and assigning a random gradient (direction) vector to each grid point. To get the noise value
at any location *within* a grid cell, you calculate vectors from the location to the cell's corners, compute the dot
product with the corner gradients, and then smoothly interpolate these values [2]. The result is a smooth, continuous,
organic-looking pattern often used for terrain heightmaps, clouds, fire effects, and wood grain textures [1]. However,
because it's based on a square/cubic grid, Perlin noise can sometimes exhibit subtle directional artifacts, especially
noticeable at lower frequencies.

#### Simplex Noise

Also developed by Ken Perlin (in 2001) to address some of Perlin noise's limitations, Simplex noise uses a simpler
lattice structure (triangles in 2D, tetrahedra in 3D) instead of a square/cubic grid [3]. This results in several
advantages:

* Lower computational complexity, especially in higher dimensions.
* No significant directional artifacts (more isotropic).
* Smoother visual appearance.
  For these reasons, Simplex noise is often preferred for modern terrain generation [4].

#### Value Noise

A simpler approach where random *values* (not gradients) are assigned to grid points, and the noise value at any
location is found by smoothly interpolating the values at the surrounding grid points [5]. It's computationally cheaper
than gradient noise but can sometimes look blockier or less "natural."

#### Fractional Brownian Motion (fBm) / Fractal Noise

This isn't a single noise function but a technique for combining multiple layers (called *octaves*) of a base noise
function (like Perlin or Simplex) at different frequencies and amplitudes [6].

* The first octave (low frequency, high amplitude) creates large, broad features.
* Subsequent octaves add progressively higher frequencies and lower amplitudes, layering finer and finer details on top.

The key parameters are:

##### Persistence

Controls how much the amplitude decreases for each successive octave (typically < 1).

##### Lacunarity

Controls how much the frequency increases for each successive octave (typically > 1). By summing these layers, you get
a "fractal" appearance – statistical self-similarity across different scales, much like real mountains or coastlines.
fBm is the workhorse for generating realistic-looking base terrain heightmaps.

Here's a conceptual C++ snippet showing how fBm (or "octave noise") might be implemented using a placeholder `noise2D`
function (which could be Perlin or Simplex):

```cpp
#include <vector>
#include <cmath>
#include <random> // For seeding, though actual noise uses deterministic hashing

// Placeholder noise function: returns noise value in [-1,1] for coordinates (x,y).
// In reality, this would involve permutation tables, gradient vectors, interpolation etc.
// See libraries like FastNoiseLite, libnoise, or implement Perlin/Simplex yourself.
double noise2D(double x, double y);

// Generate fractal noise (fBm) by summing octaves of noise
double fractalBrownianMotion(double x, double y, int octaves, double persistence = 0.5, double lacunarity = 2.0) {
    double totalValue = 0.0;
    double frequency = 1.0;
    double amplitude = 1.0;
    double maxValue = 0.0; // Used for normalization

    for (int i = 0; i < octaves; ++i) {
        totalValue += amplitude * noise2D(x * frequency, y * frequency);
        maxValue += amplitude; // Accumulate max possible amplitude

        amplitude *= persistence; // Decrease amplitude for next octave
        frequency *= lacunarity;  // Increase frequency for next octave
    }

    // Normalize the result to be roughly in [-1, 1] (assuming noise2D is in [-1, 1])
    if (maxValue > 0) {
        return totalValue / maxValue;
    } else {
        return 0.0; // Avoid division by zero if octaves=0 or amplitude=0
    }
}

int main() {
    const int width = 512, height = 512;
    std::vector<std::vector<double>> heightmap(height, std::vector<double>(width));

    // --- Noise Initialization Would Go Here ---
    // (e.g., seeding the PRNG, setting up permutation tables for Perlin/Simplex)

    // --- Generate Heightmap using fBm ---
    int numOctaves = 8;
    double noisePersistence = 0.5; // Standard 1/f noise characteristic
    double noiseLacunarity = 2.0;
    double baseFrequency = 2.0; // Controls the overall scale of the largest features

    for (int y = 0; y < height; ++y) {
        for (int x = 0; x < width; ++x) {
            // Map pixel coordinates to noise input coordinates
            // Dividing by width/height scales input; baseFrequency adjusts overall scale
            double nx = (double(x) / width) * baseFrequency;
            double ny = (double(y) / height) * baseFrequency;

            // Generate fBm noise value
            double elevation = fractalBrownianMotion(nx, ny, numOctaves, noisePersistence, noiseLacunarity);

            // Map the noise value (e.g., [-1, 1]) to your desired height range
            // Example: Map to [0, 1]
            heightmap[y][x] = (elevation + 1.0) * 0.5;
        }
    }

    // --- Use the Heightmap ---
    // (e.g., render it, determine land/water based on a threshold, etc.)
    // ...

    return 0;
}
```

Noise functions are the fundamental texture, the raw material from which many procedural worlds are sculpted. But just
applying noise isn't enough to create a believable world. We need structure, geology, and the effects of natural
processes.

## Shaping the Canvas: Map Geometry and Representation

Before we can paint mountains and rivers, we need a canvas. How do we represent the game world? The choice of map
geometry has profound implications for everything that follows.

### Flat Maps (Planar Worlds)

The simplest approach is to treat the world as a 2D plane.

#### Infinite Continuous Planes

Games like *Minecraft* conceptually use an infinite plane. The world isn't pre-generated; chunks are generated on the
fly as the player explores, often using noise functions evaluated at the chunk's coordinates. Techniques like *periodic
noise* or careful coordinate management are needed to ensure chunks stitch together seamlessly [7].

#### Bounded Planes

Simpler maps might just have hard edges (invisible walls or an "edge of the world"). This is easy but can feel
artificial.

#### Wrapped Planes (Toroidal Worlds)

To eliminate edges, flat maps can be "wrapped." Going off the east edge brings you to the west edge, and going off the
north edge brings you to the south. Topologically, this creates a *torus* (a donut shape), not a sphere. This is common
in older strategy games or simulations where a finite but borderless world is desired [8].

### Spherical Maps (Planetary Worlds)

For simulating planets, strategy games spanning a globe, or space exploration games, a spherical representation is more
realistic. However, this introduces complexity. How do you map a sphere onto data structures and display it on a 2D
screen?

#### Challenges

Representing a sphere without significant distortion or awkward singularities (like the poles in latitude-longitude
grids) is tricky. An *equirectangular projection* (mapping latitude/longitude directly to a rectangular grid) is simple
but severely stretches areas near the poles and collapses everything to a single point *at* the poles.

#### Common Solutions

##### Cube Mapping

Project the sphere onto the six faces of a cube. Each face can be treated as a regular square grid, minimizing
distortion compared to equirectangular. The main challenge is handling the seams/edges between the cube faces smoothly.

##### Icosahedral Subdivision (Geodesic Grids)

Start with an icosahedron (20 triangular faces) inscribed within the sphere. Project its vertices onto the sphere. Then,
recursively subdivide each triangle into smaller triangles, projecting new vertices onto the sphere. This creates a
*geodesic dome* structure. The dual of this triangular mesh (connecting the centers of adjacent triangles) results in a
grid composed mostly of hexagons, with exactly 12 pentagons located at the original icosahedron's vertices (think of a
soccer ball pattern) [4]. This **hex-pent grid** provides relatively uniform cell sizes and shapes across the sphere,
making it popular for global climate models and some planetary generators [4]. The main complexity lies in the data
structures needed to handle the 12 pentagons and track neighbor relationships.

##### Fibonacci Lattice / Spiral Points

Algorithms exist to distribute points quasi-uniformly on a sphere's surface using spiral patterns related to the
Fibonacci sequence or golden ratio [4]. These points can then be used as centers for Voronoi regions or vertices for a
Delaunay triangulation, creating an irregular but relatively even mesh covering the sphere. Amit Patel's influential
planet generation experiments often start by distributing points this way, then slightly "jittering" them to avoid
unnatural regularity [4].

### Tiling Schemes: Squares vs. Hexes vs. Irregular

Whether the map is flat or spherical (represented facet by facet, like a cube map or geodesic grid), we often divide it
into discrete units or *tiles* for gameplay or simulation purposes.

#### Square Tiles

The simplest. Easy to address (row, column), easy to map to pixel grids. Neighbors are straightforward (4 cardinal,
possibly 4 diagonal). The main drawback, especially for movement or area-of-effect calculations, is the difference
between orthogonal and diagonal distances/connectivity.

#### Hexagonal Tiles

A favorite for many strategy games (*Civilization V/VI*, *RimWorld*'s planet map). Why?

##### Uniform Adjacency

Each hex has 6 neighbors, all equidistant from the center. This eliminates the diagonal vs. orthogonal awkwardness of
squares, making movement costs and distances more consistent.

##### Connectivity

The 6-way connectivity can lead to more organic-looking shapes for landmasses and region boundaries.

##### Implementation

Requires slightly more complex coordinate systems (axial or cube coordinates are common) and careful handling of
neighbor finding and wrapping, especially on a sphere. Excellent guides exist, notably from Red Blob Games [9] [8].

##### Irregular Polygons (Voronoi/Graph-Based)

Instead of uniform tiles, partition the map using irregular polygons, often generated via a **Voronoi diagram**.

1. Scatter a set of points (seeds) across the map (plane or sphere).
2. For each seed, define its region (cell) as all locations closer to that seed than any other.
3. The result is a tessellation of the map into irregular polygonal cells.

Amit Patel's *Polygon Map Generation* tutorials are seminal work in this area [10].

###### Advantages

Can produce much more natural-looking coastlines, region boundaries, and river paths (which can flow along the edges of
the Voronoi graph). The underlying graph structure is great for simulating flows or relationships between regions. It
provides a "good skeleton" for placing features [10]. Systems like *Mapgen4* and *Tectonics.js* utilize this
approach [4].

###### Disadvantages:

Computationally more expensive to generate and work with. Geometric operations (like finding neighbors or calculating
distances/areas) are more complex than on a regular grid.

The choice of map geometry and tiling impacts everything downstream, from how noise is applied to how simulations like
erosion or climate are run. A **spherical Voronoi mesh** arguably offers the highest potential for realism on a
planetary scale, but simpler grids (flat or wrapped, square or hex) are often chosen for performance and ease of
implementation.

## Raising the Land: Generating Continents and Oceans

With a canvas defined, the next step is painting the broad strokes: where is land, and where is sea? How do continents
form? Two main paradigms dominate: noise-based heightmaps and plate tectonics simulation.

### Fractal Noise Heightmaps: The Quick and Dirty Approach

The most common method is to use fractal noise (like fBm described earlier) to generate an **elevation map** or *
*heightmap**. This is typically a 2D grid where each cell stores an elevation value.

#### Generate Noise

Use multiple octaves of Perlin or Simplex noise to create a heightfield $H(x, y)$. Low frequencies create broad
continents/basins, high frequencies add detail.

#### Set Sea Level

Choose a threshold value. All points on the heightmap below this value become ocean; all points above become land. A
common target is ~70% ocean, similar to Earth, which might correspond to picking the 30th percentile of height values as
sea level [1].

This quickly produces a world with land and sea. However, pure noise has limitations:

#### Lack of Structure

Real continents have long, linear mountain ranges, vast flat plains, and specific shapes dictated by geological history.
Noise-based terrain tends to produce more random, blobby, or uniformly hilly landscapes. Mountain ranges don't align
meaningfully [1].

#### Island Problem

Naively thresholding noise often results in worlds that are either mostly ocean with scattered small islands, or mostly
land with scattered lakes, rather than a few large continents.

#### Techniques to Improve Noise-Based Continents

##### Shaping Functions

Multiply the noise heightmap by another function (e.g., a radial gradient that lowers elevation near the edges of the
map) to force oceans around a central landmass. WorldEngine uses a trick where they normalize the heightmap so the
lowest point is at the border and highest is central, then flood from the edges to ensure a central continent [1].

##### Domain Warping

A clever technique popularized by Inigo Quilez and others. Instead of evaluating noise at `(x, y)`, you evaluate it at
coordinates that have been distorted by *another* noise function. For example:
`height = noise1(x + offset * noise2(x, y), y + offset * noise3(x, y))`. This "warps" the coordinate space, creating
swirling, folded, and branching patterns in the output noise [7]. Domain warping can produce features that *look*
remarkably like eroded terrain – twisty ridges, river-like valleys – without actually simulating erosion [7]. It's
computationally cheap (just more noise lookups) and adds significant visual complexity, making basic noise terrain look
much more interesting.

##### Diamond-Square Algorithm

A classic (though somewhat dated) fractal algorithm specifically for generating heightmaps. It works by recursively
subdividing squares, setting corner points, then calculating midpoints with random offsets. It tends to produce
characteristic square-aligned artifacts but is simple to implement.

Noise provides the fundamental texture and detail, but for realistic *structure*, we often need to simulate the
processes that build that structure.

### Plate Tectonics Simulation: Building Worlds Geologically

On Earth, continents and mountains are formed by the slow dance of tectonic plates. Simulating this process can produce
far more plausible large-scale world structures. While full geophysical simulation is complex, simplified models are
feasible for world generation. Notable projects exploring this include *WorldEngine*, *Tectonics.js*, *PyTectonics*, and
academic work like "Procedural Tectonic Planets" [1] [11] [12].

#### A Simplified Tectonic Algorithm

##### Initialize Plates

Divide the world (represented as a grid or spherical mesh) into a number of distinct regions, representing tectonic
plates (e.g., 6-20 plates). This can be done randomly (e.g., Voronoi partitioning) or based on initial noise patterns.
Assign each plate an initial velocity (direction and speed).

##### Move Plates

Simulate the movement of each plate over a time step according to its velocity. On a sphere, this movement follows great
circle paths.

##### Detect Interactions

Identify where plates are colliding (convergent boundary), pulling apart (divergent boundary), or sliding past each
other (transform boundary).

##### Modify Terrain

###### Convergence (Collision)

Where plates collide, uplift the terrain significantly, forming mountain ranges. If one plate is denser (oceanic vs.
continental), it might *subduct* (slide underneath), leading to uplift on the overriding plate and potentially volcanic
activity or deep ocean trenches.

###### Divergence (Rifting)

Where plates pull apart, lower the terrain, creating rift valleys on land or mid-ocean ridges under the sea where new
crust forms.

###### Transform

Minimal vertical change, but can create fault lines.

##### Iterate

Repeat steps 2-4 for many simulated time steps (representing millions of years). Plate velocities might change, plates
might merge or break apart.

#### More Sophisticated Models (e.g., Tectonics.js)

*Tectonics.js* implements a more physics-inspired model on a spherical grid [11]:

* It tracks crust properties like density and age. Oceanic crust becomes denser as it ages [11].
* Dense oceanic crust tends to subduct under lighter continental or younger oceanic crust [11].
* Plate velocities are calculated based on forces pulling plates towards subduction zones [11]. Plates are dynamically
  identified by grouping areas with similar velocities.
* At convergent boundaries, overlapping crust is effectively deleted (simulating subduction). At divergent boundaries,
  gaps are filled with new crust [11].
  This leads to emergent, realistic features: long mountain chains, island arcs, spreading ocean basins [11]. The
  downside is computational cost – it's not instantaneous [11].

#### Benefits of Tectonic Simulation

##### Plausible Structure

Generates continents, oceans, and mountain ranges with shapes and alignments that resemble real geology. Features that
noise alone struggles with.

##### Inherent History

The generated world has a "story." You can trace why a mountain range exists (e.g., collision of plates X and Y). This
is great for lore and deeper simulation [11] [13].

##### Emergent Land/Sea Distribution

The amount and shape of land isn't predefined but emerges naturally from the simulation.

#### Hybrid Approaches

Pure tectonic simulation can sometimes produce terrain that lacks fine detail or looks "bland" between the major
features. Many systems, like *WorldEngine*, use a hybrid approach:

1. Start with an initial heightmap generated by noise (providing base detail).
2. Run a plate tectonics simulation to deform this heightmap, creating large-scale structures.
3. Apply *more* noise afterward to add roughness and smaller features back in.
   [1]. This combination often yields the best results: realistic large-scale structure with natural-looking small-scale
   variation.

After generating the basic landforms via noise, tectonics, or a hybrid, the next step is often to refine the terrain
with processes that shape it over time.

## Carving the Details: Erosion and River Simulation

A freshly generated heightmap, even one based on tectonics and noise, often looks too sharp, too uniform, too...
*computer-generated*. Real landscapes are heavily sculpted by erosion – the relentless wearing away of rock and soil by
water, wind, ice, and gravity. Simulating erosion is crucial for adding realism and creating features like river
valleys, smooth hills, and depositional plains. As the WorldEngine developers state, *"If you do not simulate erosion
you will never obtain realistic maps."* [1].

### Hydraulic Erosion (Water Power)

This is the most significant type of erosion for shaping large landscapes. It involves water (rain, rivers) dislodging
soil/rock particles, transporting them downhill, and depositing them where the water slows down. There are two main
algorithmic approaches [14]:

#### Eulerian (Grid-Based)

Treats water as a fluid layer covering the terrain grid. Each cell tracks water depth and sediment concentration. Water
flows between cells based on height differences (pressure gradients), carrying sediment with it. Sediment is eroded from
the ground where flow is high and deposited where it's low. These models often use simplified versions of fluid dynamics
equations (like the shallow water equations). They can capture large-scale effects but can be complex and
computationally intensive.

#### Lagrangian (Particle-Based / "Rain Droplet" Model):

Simulates the paths of individual water droplets. This is very popular in game development due to its conceptual
simplicity and ability to create intricate channel networks. The basic idea:

##### Spawn Droplet

Create a "raindrop" particle at a random location on the heightmap with a small amount of water and initially no
sediment.

##### Flow Downhill

Move the droplet in the direction of the steepest downhill slope, calculated from the heightmap gradient at its current
position. Its velocity increases on steeper slopes.

##### Erode/Deposit

The droplet's capacity to carry sediment depends on factors like its water volume, velocity, and the slope it's on.

* If the droplet is moving fast/downhill (high capacity) and currently holds less sediment than its capacity, it
  *erodes* material from the ground, decreasing terrain height and increasing its sediment load.
* If the droplet slows down (e.g., on flatter ground, low capacity) and holds more sediment than it can carry, it
  *deposits* sediment, increasing terrain height and decreasing its load.
  A simple capacity model might be: `capacity = k * velocity * slope` [14]. The amount eroded or deposited is then
  proportional to the difference between capacity and current sediment load [14].

##### Evaporate

The droplet gradually loses water over time/distance.

##### Terminate

The simulation for that droplet ends when it runs out of water, flows off the map, or gets stuck in a pit.

##### Repeat

Simulate thousands or millions of these droplets. Each path contributes incrementally to carving channels and building
up depositional areas.

Here's the C++ pseudocode snippet (adapted from Article 1 and common implementations) illustrating the core loop for a
single droplet:

```cpp
#include <vector>
#include <cmath>
#include <algorithm> // For std::max, std::min

// Assumes existence of a HeightMap class/struct
// with methods like:
// double getHeight(double x, double y); // Interpolated height
// Vector2 getGradient(double x, double y); // Steepest downhill gradient vector
// void addHeight(int gridX, int gridY, double delta); // Modify height at grid cell
// int width(); int height();
struct HeightMap { /* ... definition ... */ };
struct Vector2 { double x, y; };

struct Droplet {
    double x, y;      // Current position (continuous)
    double vx, vy;    // Current velocity
    double water;     // Amount of water remaining
    double sediment;  // Amount of sediment carried
};

// Simulation parameters (tune these!)
const int maxSteps = 64;         // Max lifetime of a droplet
const double timeStep = 1.0;     // Simulation step duration
const double friction = 0.1;     // Slows down velocity over time
const double evaporationRate = 0.01; // Water lost per step
const double erosionRate = 0.01;  // How readily soil is eroded
const double depositRate = 0.01; // How readily sediment is deposited
const double sedimentCapacityFactor = 10.0; // Scales overall capacity
const double minSlopeForErosion = 0.01; // Don't erode on near-flat ground
const double minWater = 0.001;    // Stop when droplet is too small

void simulateDroplet(Droplet& d, HeightMap& H) {
    for (int step = 0; step < maxSteps; ++step) {
        // Get current grid cell indices and interpolated position
        int gridX = static_cast<int>(d.x);
        int gridY = static_cast<int>(d.y);

        // Boundary check
        if (gridX < 0 || gridX >= H.width() - 1 || gridY < 0 || gridY >= H.height() - 1) {
            break; // Droplet flowed off map
        }

        // Calculate height and gradient at the droplet's interpolated position
        double currentHeight = H.getHeight(d.x, d.y);
        Vector2 gradient = H.getGradient(d.x, d.y); // Points downhill

        // Update velocity based on gradient (gravity) and friction
        d.vx = (d.vx * (1.0 - friction)) + gradient.x * timeStep;
        d.vy = (d.vy * (1.0 - friction)) + gradient.y * timeStep;

        // Store old position
        double oldX = d.x;
        double oldY = d.y;

        // Update position based on velocity
        d.x += d.vx * timeStep;
        d.y += d.vy * timeStep;

        // Boundary check again after move
        if (d.x < 0 || d.x >= H.width() - 1 || d.y < 0 || d.y >= H.height() - 1) {
             // Deposit remaining sediment at last valid position before leaving map
             H.addHeight(gridX, gridY, d.sediment);
             break;
        }

        // Calculate height difference between old and new position
        double newHeight = H.getHeight(d.x, d.y);
        double deltaHeight = currentHeight - newHeight; // Positive if moving downhill

        // Calculate sediment capacity
        // Capacity depends on water volume, speed, and slope (deltaHeight)
        // Simple model: capacity proportional to water * deltaHeight (steeper drop = more capacity)
        // Clamp deltaHeight to avoid negative capacity if moving uphill slightly
        double speed = std::sqrt(d.vx * d.vx + d.vy * d.vy);
        double capacity = std::max(0.0, deltaHeight) * d.water * sedimentCapacityFactor * speed;

        // Erode or deposit sediment at the *previous* cell (gridX, gridY)
        // This prevents digging holes directly under the droplet
        if (deltaHeight > minSlopeForErosion) { // Only erode/deposit significantly if moving downhill
             double sedimentDiff = capacity - d.sediment;
             if (sedimentDiff > 0) {
                 // Can carry more: Erode
                 double amountToErode = std::min(sedimentDiff * erosionRate, deltaHeight); // Don't erode more than available height diff
                 amountToErode = std::min(amountToErode, H.getHeight(gridX + 0.5, gridY + 0.5) * 0.1); // Limit erosion based on current height too
                 amountToErode = std::max(0.0, amountToErode); // Ensure non-negative

                 H.addHeight(gridX, gridY, -amountToErode); // Remove from terrain
                 d.sediment += amountToErode;              // Add to droplet
             } else {
                 // Carrying too much: Deposit
                 double amountToDeposit = std::min(-sedimentDiff * depositRate, d.sediment); // Deposit difference, up to amount carried
                 amountToDeposit = std::max(0.0, amountToDeposit); // Ensure non-negative

                 H.addHeight(gridX, gridY, amountToDeposit); // Add to terrain
                 d.sediment -= amountToDeposit;           // Remove from droplet
             }
        }


        // Evaporate water
        d.water *= (1.0 - evaporationRate);
        if (d.water < minWater) {
             // Deposit remaining sediment if droplet evaporates
             H.addHeight(gridX, gridY, d.sediment);
             break;
        }
    }
}

// --- In main() or simulation loop ---
// HeightMap terrain = ...; // Initial heightmap
// int numDroplets = 100000;
// std::mt19937 rng(seed); // Random number generator
// std::uniform_real_distribution<double> distX(0.0, terrain.width() - 1.0);
// std::uniform_real_distribution<double> distY(0.0, terrain.height() - 1.0);

// for (int i = 0; i < numDroplets; ++i) {
//     Droplet drop = {
//         distX(rng), distY(rng), // Random start position
//         0.0, 0.0,             // Initial velocity
//         1.0,                  // Initial water
//         0.0                   // Initial sediment
//     };
//     simulateDroplet(drop, terrain);
// }
// --- Terrain now contains eroded features ---
```

**Performance Note:** Simulating millions of droplets can be slow. Optimizations include:

* GPU acceleration (as researched by Mei et al. [1]).
* Simulating larger "streams" or using grid-based flow accumulation models instead of individual droplets.

### Thermal Erosion (Weathering / Mass Wasting)

This simulates the effect of gravity causing material on steep slopes to crumble and slide downwards, accumulating at
the base (forming *talus slopes*). It's much simpler than hydraulic erosion.

A common algorithm:

1. Iterate through each cell `(x, y)` of the heightmap.
2. Compare its height `H(x, y)` to its neighbors `H(nx, ny)`.
3. For each neighbor *lower* than the current cell, calculate the height difference `d = H(x, y) - H(nx, ny)`.
4. If this difference `d` exceeds a *threshold* (representing the material's "angle of repose" – the steepest angle it
   can maintain), then move some material from the higher cell `(x, y)` to the lower neighbor `(nx, ny)`.
5. The amount moved is typically proportional to the excess difference `(d - threshold)`. For example, move
   `c * (d - threshold)` amount of height, where `c` is a small constant [15] [16]. Ensure total height is conserved (or
   approximately conserved) by distributing the removed height among all lower neighbors exceeding the threshold.
6. Repeat this process for several iterations until the terrain stabilizes (no more slopes exceed the threshold
   significantly).

Thermal erosion is computationally cheap and effective at smoothing out unnaturally sharp peaks and cliffs left by noise
generation or tectonic uplift, giving mountains a more weathered look. It's often applied as a final smoothing pass.

### River Network Generation and Watersheds

While hydraulic erosion *creates* river channels, sometimes you want to explicitly define major rivers for gameplay or
ensure a realistic drainage network exists.

#### Flow Accumulation Analysis:

A standard GIS technique adaptable for games.

1. For each cell, determine its *flow direction* – which neighbor is steepest downhill?
2. Calculate *flow accumulation* for each cell: how many upstream cells eventually drain *through* this cell? This is
   often done recursively or iteratively, passing flow counts downstream.
3. Cells with a high flow accumulation value represent potential river paths. Define a threshold – cells above it are
   part of a river network. This naturally creates branching tributary systems flowing from highlands to lowlands or
   oceans.

#### Explicit River Carving

Start potential rivers at high points (e.g., mountain springs, areas of high simulated rainfall). Simulate the river
flowing downhill, actively lowering the terrain height along its path to "carve" a valley. Rules are needed to handle
hitting flat areas (meander) or depressions (form lakes). Amit Patel's Mapgen4 allows users to *draw* rivers, and the
system then carves them into the terrain [17].

#### Simulation-Driven Rivers

In systems like WorldEngine, rivers emerge more directly from the coupled simulation. After calculating precipitation
and running erosion, they explicitly trace water flow paths from source to sea, calculate water volume in each segment,
and designate tiles with significant flow as rivers [1].

#### Watersheds

The flow direction map also allows identifying *watersheds* (or drainage basins) – the area of land that drains into a
particular river or outlet point. These watersheds, separated by ridges (drainage divides), form natural geographical
regions often useful for defining political borders, biome zones, or AI territories.

In summary, erosion and river simulation transform a static heightmap into a landscape that feels shaped by natural
forces over time. They carve the valleys, create the river networks, and deposit the fertile plains that make a world
feel lived-in and geologically plausible. Even simplified erosion or faked effects like domain warping can add
significant realism compared to raw noise terrain.

## Breathing Life into the World: Climate, Weather, and Biomes

We have land, water, mountains, and rivers. But what makes a desert different from a jungle, or a tundra from a
temperate forest? Climate. Simulating climate patterns – temperature, precipitation, wind – allows us to determine which
**biomes** (ecological regions) should exist where.

### Simulating Climate

Full global climate modeling (GCM) like that used by climate scientists is computationally prohibitive for most game
world generation. Instead, simplified, heuristic models are used to capture the most important factors influencing
climate:

#### Latitude

The primary driver of temperature. Closer to the equator = more direct sunlight = warmer. Temperature generally
decreases towards the poles. A simple model might be `Temp = BaseTemp - TempDrop * abs(latitude)` or using a sine
function.

#### Altitude

Temperature decreases with height (the *lapse rate*, roughly 6.5°C per 1000m). Mountains are colder than lowlands at the
same latitude.

#### Land vs. Water

Water heats and cools more slowly than land. Coastal areas tend to have more moderate temperatures (less seasonal
variation) than continental interiors (*continentality*). Large bodies of water also act as moisture sources.

#### Prevailing Winds

Global atmospheric circulation creates dominant wind patterns (e.g., easterly trade winds in the tropics, westerlies in
mid-latitudes). Winds transport moisture from oceans over land.

#### Ocean Currents (Advanced)

Warm currents (like the Gulf Stream) can significantly warm coastal regions; cold currents can cool them. Simulating
these adds realism but is often skipped in simpler models.

#### Orographic Precipitation (Rain Shadows)

This is crucial! When moist air carried by wind hits a mountain range, it's forced upwards. As it rises, it cools, and
its capacity to hold moisture decreases. This causes rain or snow to fall on the *windward* side of the mountains. The
air that descends on the *leeward* side is now dry, creating a **rain shadow** – an area of low precipitation (often
deserts or steppes) [18]. Any plausible climate simulation *must* account for this effect.

#### Simplified Climate Modeling Approach

##### Temperature Map

Calculate base temperature based on latitude. Adjust for altitude using the lapse rate. Optionally, add modifications
for distance from coast (continentality).

##### Prevailing Winds

Define basic wind directions for different latitude bands (e.g., West in mid-latitudes, East in tropics).

#### Moisture & Precipitation

* Simulate moisture transport. A simple way (used in Mapgen4) is to process cells in order along the wind
  direction [18].
* Start with moisture over oceans (evaporation).
* As air moves over land, it might pick up some moisture (less than ocean) or gradually lose it.
* When air hits mountains (rising elevation along wind path), reduce its moisture-holding capacity. If capacity drops
  below current moisture, the excess falls as rain/snow on the windward slope [18]. The air passed to the leeward side
  is now drier.
* Incorporate general global patterns (e.g., high rainfall near the equator - ITCZ, dry zones around 30° latitude).

#### Output

Generate maps of average annual temperature and average annual precipitation.

*WorldEngine* uses a simplified approach: Temperature from latitude/altitude, Precipitation from temperature plus noise,
with specific erosion/flow steps calculating river discharge and humidity [1]. They explicitly chose *not* to simulate
seasons to keep complexity manageable, focusing on annual averages [1].

For higher fidelity (perhaps for sci-fi games aiming for realism), tools like **ExoPlaSim** exist. It's a simplified but
physically based 3D global climate model that can simulate atmospheric circulation, heat transport, and precipitation
for planets with different parameters (rotation, atmosphere, star type) [19] [20]. Running such a model is more
intensive but yields highly realistic climate patterns.

### Assigning Biomes

Once you have temperature and precipitation maps, you can classify different regions into **biomes**. Biomes represent
major ecological communities characterized by dominant plant types and adapted to the prevailing climate (e.g., Tropical
Rainforest, Temperate Grassland, Arctic Tundra, Subtropical Desert).

#### Biome Classification Schemes

The simplest way is to use a lookup diagram based on temperature and precipitation.

##### Whittaker Biome Diagram

A classic ecological chart plotting average annual temperature vs. average annual precipitation, dividing the space into
major biome types.

##### Holdridge Life Zones

A more detailed system used by *WorldEngine*. It considers temperature, precipitation, and also *potential
evapotranspiration* (related to energy availability) to define a finer set of life zones (~38 zones) [1]. WorldEngine
implemented around 40 specific biomes based on this, from "Subpolar Dry Tundra" to "Tropical Rainforest" [1].

#### Algorithm

1. For each land cell on your map, get its calculated temperature and precipitation values.
2. Use the chosen classification scheme (Whittaker, Holdridge, or a custom one) to determine the corresponding biome
   type.
3. Assign the biome type to the cell.
4. (Optional) Apply smoothing or filtering: A single desert tile in the middle of a rainforest might be a noise
   artifact. You could use a majority filter or smooth biome boundaries to make transitions look more natural.

#### Adding Nuance

More advanced systems might consider:

##### Soil Moisture/Drainage

A flat, wet area might become a swamp or marsh, even if the temperature/precipitation alone suggest forest. WorldEngine
simulates water flow and permeability to identify potentially marshy areas [1].

##### Seasonality

If seasons were simulated, the *variation* in temperature and precipitation could influence biomes (e.g.,
differentiating deciduous from coniferous forests).

##### Soil Type/Fertility

This could emerge from erosion simulation (sediment deposition = fertile) and influence vegetation density or type.

Biome generation is often the final step in creating the environmental backdrop. It gives the world its visual character
and dictates the types of resources, flora, and fauna players might encounter. The beauty is seeing how the underlying
geology (tectonics creating mountains) influences climate (rain shadows) which in turn dictates the biomes (deserts
behind mountains).

## Under the Hood: Map Data Structures

How is all this complex world information actually stored in memory or on disk? The choice of data structure impacts
performance, flexibility, and the types of simulations that are easy to implement.

### Regular Grids (Raster Data)

The most common approach, especially for heightmaps. Use 2D arrays (or 3D for voxels like *Minecraft*) to store values (
elevation, temperature, biome ID, etc.) for each cell.

* **Pros:** Simple addressing (`map[row][col]`), efficient neighborhood lookups (crucial for erosion, smoothing,
  cellular automata), aligns well with texture mapping for rendering.
* **Cons:** Can be memory-intensive for large, high-resolution maps. Represents discrete steps, less natural for smooth
  features (though interpolation helps). Spherical representation using grids suffers from distortion (equirectangular)
  or edge seams (cube map). Hex grids require special coordinate mapping onto the 2D array.

### Graph-Based Representations (Irregular Meshes)

Store the world as a network of nodes and edges, often based on Voronoi diagrams or Delaunay triangulations.

* Nodes (e.g., Voronoi cell centers) store properties (elevation, biome).
* Edges represent adjacency and can store information like flow rates (for rivers) or boundary types.
* Red Blob Games' Polygon Map Generation tutorials detail storing data at corners, edges, *and* centers of the Voronoi
  polygons for different purposes [10].

#### Pros

Excellent for representing irregular, natural boundaries. Flexible resolution possible. Good for pathfinding or flow
simulations along graph edges.

#### Cons

More complex data structures. Neighborhood operations can be slower/more complex than grid lookups. Physics
simulations (like fluid dynamics) are harder to implement on irregular meshes.

### Spherical Subdivisions (DGGS)

For true global representation, use specialized structures like icosahedral geodesic grids (hex/pent meshes) or other
Discrete Global Grid Systems (DGGS). These aim for near-uniform cell size/shape across the sphere with no singularities.

#### Pros

Best for accurate global simulations (climate, tectonics). Minimal distortion.

#### Cons

Complex implementation, especially handling neighbor relationships and coordinates across the sphere.

### Multi-Layer Data

A generated world isn't just a heightmap. It's a collection of related data layers. A practical system often stores
multiple aligned grids or graphs:

* Elevation map
* Water map (ocean/lake depth, river paths/flow)
* Temperature map
* Precipitation map
* Biome map
* Tectonic plate ID map
* Vegetation density map, etc.

*WorldEngine*, for example, explicitly stores many such layers per tile [1]. This allows different systems (rendering,
AI, gameplay logic) to access the specific information they need. It also allows exporting different map views (like a
climate map or political map).

### Implicit/Functional Representation

For truly infinite or extremely detailed worlds, storing everything explicitly is impossible. Instead, use functions (
like noise functions) to *calculate* terrain properties on demand at any given coordinate `(x, y, z)`. Games like *No
Man's Sky* rely heavily on generating planet surfaces locally from mathematical formulas as the player approaches,
rather than storing the entire planet's geometry.

The ideal data structure depends on the scale of the world, the required level of detail, the types of simulations being
run, and performance constraints. Many systems use a combination – perhaps a coarse graph for global structure and finer
grids for local detail.

## Learning from the Masters: Quick Case Studies

Let's briefly look at how some notable projects and games combine these techniques:

### WorldEngine (Open Source Generator)

A prime example of a full pipeline [1]. Its steps roughly follow our discussion:

1. Initial heightmap (Simplex noise).
2. Plate tectonics simulation (deforms heightmap, creates mountains).
3. Flooding to set sea level.
4. Climate simulation (temperature based on latitude/altitude, precipitation based on temp + noise).
5. Hydraulic erosion (droplet model, carving rivers).
6. Hydrology (calculating river flow, humidity).
7. Biome assignment (Holdridge life zones).
   It emphasizes the synergy: noise for detail, tectonics for structure, erosion for realism. It outputs multiple data
   layers exportable to game engines or GIS tools.

### Mapgen4 (Red Blob Games - Web Tool)

Focuses on interactive fantasy map generation [17]. Uses a Voronoi graph structure. Key features:

* Fast, interactive updates (user paints features, map regenerates). Achieved via optimized data structures (e.g.,
  spatial partitioning for rivers) and possibly multithreading.
* Simplified climate simulation (wind, orographic rain) driving river formation and biomes.
* Height calculation uses distance fields for smooth blending of user-painted mountains.
* Stylized rendering to look like hand-drawn maps.
  It shows how core concepts can be implemented efficiently even in a web browser for interactive use.

### Tectonics.js / PyTectonics (Carl Davidson et al.)

Focuses specifically on high-fidelity plate tectonics simulation on a sphere [11]. Uses spherical Voronoi meshes and
simulates crust properties, subduction, and velocity fields based on physical principles. Produces very realistic
continental configurations but is computationally intensive. Often used as a *starting point* – generate the tectonic
base map, then use other tools for erosion and detailing.

### Dwarf Fortress (Bay 12 Games)

Famous for its incredibly deep simulation. World generation involves:

* Generating a base fractal heightmap.
* Simulating geology (mineral distribution).
* Simulating rainfall, drainage, rivers, and lakes.
* Simulating temperature and biomes.
* Crucially, simulating *thousands of years of history*: rise and fall of civilizations, wars, mythical beasts, heroes –
  all leaving traces on the world and creating rich, unique lore tied to the generated geography [1]. It highlights how
  procedural generation can extend far beyond just terrain into history and culture.

### Minecraft (Mojang Studios)

Uses chunk-based generation on a conceptually infinite plane. Core terrain uses layered 3D Perlin/Simplex noise. Biomes
influence terrain height variation, decorations (trees, structures), and block types. Features like caves, ravines, and
ore veins are added using separate procedural algorithms within each chunk. Focuses on exploration and emergence from
relatively simple block-based rules.

### No Man's Sky (Hello Games)

Generates an entire galaxy of planets using deterministic procedural formulas from a single seed. Planet surfaces are
generated on the fly using noise functions and other algorithms as the player approaches. Emphasizes scale, variety, and
seamless exploration from space to ground.

These examples show there's no single "right" way. The best approach depends on the game's goals: deep simulation vs.
fast interaction, realism vs. stylized appearance, planetary scale vs. local detail. Often, the most successful systems
are hybrids, carefully combining noise, simulation, and heuristics.

## Peeking into the Future: Where World Generation is Headed

The field is constantly evolving. Here are some exciting directions and possibilities:

### Tighter Integration & Coupling

Simulating feedback loops. How does massive erosion affect tectonic uplift over millions of years? How does climate
change (e.g., an ice age simulated historically) impact erosion rates and biome distribution? Current systems often run
steps sequentially; future ones might have more interplay.

### More Sophisticated Simulation

Incorporating more physics: advanced fluid dynamics for erosion and rivers, better atmospheric modeling for climate (
including seasons, dynamic weather), simulation of volcanism (hotspots like Hawaii), ecological succession modeling (how
biomes evolve and compete over time).

### AI and Machine Learning

#### Generative Models:

Training AI (like GANs or diffusion models) on real-world terrain or climate data to produce plausible fictional
outputs (e.g., "generate a landscape in the style of the Scottish Highlands").

#### Parameter Tuning

Using ML to automatically find generation parameters that produce worlds meeting specific design criteria (e.g., "a
world with 3 continents, mostly temperate forests, and navigable rivers").

#### Smart Content Placement

AI could learn plausible locations for resources, settlements, or points of interest based on the generated environment.

### Real-Time & Interactive Generation

Moving beyond pre-computation. Imagine worlds that visibly evolve based on player actions (e.g., a magical cataclysm
triggers tectonic shifts, large-scale engineering projects alter river flows and climate). This requires highly
efficient, incremental algorithms. Cortial et al.'s "Procedural Tectonic Planets" explored interactive *design* using
tectonics [12].

### Bridging Scales (Procedural Zoom)

Seamlessly connecting large-scale planetary generation with fine-grained local detail. Generate the planet coarsely,
then use different, higher-frequency procedural techniques (or even rule-based systems) to add detail dynamically as the
player zooms in or moves closer, ensuring consistency across scales.

### History and Culture Simulation

Expanding on the *Dwarf Fortress* model. Tightly integrating the generation of civilizations, historical events, ruins,
and lore with the physical world generation, so the environment shapes the history, and the history leaves visible marks
on the environment.

### Unified & Modular Frameworks

Creating flexible pipelines where developers can easily swap different modules (e.g., choose between fast fake erosion
or slow physical simulation, plug in different climate models) based on project needs. The framework passes world data
layers between modules.

The ultimate goal? To generate worlds that feel not just visually plausible but *alive*, with depth, history, and
internal consistency – worlds that tell stories through their very landscapes.

## Conclusion: The Art and Science of WorldSmithing

Procedural world generation is a captivating blend of art and science. It draws on mathematical noise, geological
simulation, climate science, and ecological principles, all orchestrated by algorithms to conjure digital universes from
little more than a seed value.

We've journeyed from the basic concepts of noise and seeds through the intricacies of shaping continents with tectonics,
carving details with erosion, breathing life with climate and biomes, and storing it all efficiently. We've seen that
the most compelling results often come from a **synthesis of techniques**: the raw detail of noise, the structural
foundation of simulation, and the refining touch of processes like erosion.

The tools and algorithms are becoming increasingly sophisticated, allowing even small teams or solo developers to create
worlds of staggering scale and complexity. While challenges remain in achieving perfect realism, controllability, and
performance, the future points towards even more powerful and integrated systems, potentially leveraging AI and
real-time dynamics.

Ultimately, procedural generation empowers creators not just to build worlds, but to become digital worldsmiths,
crafting universes that surprise, delight, and immerse players in ways previously unimaginable. By understanding the
algorithms and harnessing the processes that shape our own planet, we unlock the potential to create countless others,
each waiting to be explored.

## References

* [1] F. Tomassetti, ‘Diving Into Procedural Content Generation, With WorldEngine’, Smashing Magazine. [Online].
  Available: https://www.smashingmagazine.com/2016/03/procedural-content-generation-introduction/
* [2] ‘Perlin noise’, Wikipedia. [Online]. Available: https://en.wikipedia.org/wiki/Perlin_noise
* [3] ‘Simplex noise’, Wikipedia. [Online]. Available: https://en.wikipedia.org/wiki/Simplex_noise
* [4] A. J. Patel, ‘Procedural map generation on a sphere’, Red Blob Games. [Online].
  Available: https://www.redblobgames.com/x/1843-planet-generation/
* [5] ‘Value noise’, Wikipedia. May 21, 2021. [Online].
  Available: https://en.wikipedia.org/w/index.php?title=Value_noise&oldid=1024311499
* [6] ‘Brownian surface’, Wikipedia. Oct. 16, 2024. [Online].
  Available: https://en.wikipedia.org/w/index.php?title=Brownian_surface&oldid=1251552796
* [7] F. Gennari, ‘3DWorld: Domain Warping Noise’, 3DWorld. [Online].
  Available: https://3dworldgen.blogspot.com/2017/05/domain-warping-noise.html
* [8] A. J. Patel, ‘Wraparound hexagon tile maps on a sphere’, Red Blob Games. [Online].
  Available: https://www.redblobgames.com/x/1640-hexagon-tiling-of-sphere/
* [9] A. J. Patel, ‘Hexagonal Grids’, Red Blob Games, 2013. [Online].
  Available: https://www.redblobgames.com/grids/hexagons/
* [10] A. J. Patel, ‘Polygonal Map Generation, HTML5 version’, Red Blob Games. [Online].
  Available: https://simblob.blogspot.com/2017/09/mapgen2-html5.html
* [11] C. Davidson, ‘Tectonics.js: 3D Plate Tectonics in your web browser’. [Online].
  Available: https://davidson16807.github.io/tectonics.js/blog/
* [12] Y. Cortial, A. Peytavie, E. Galin, and E. Guérin, ‘Procedural Tectonic Planets’, Computer Graphics Forum, vol.
  38, no. 2, pp. 1–11, May 2019, doi: 10.1111/cgf.13614.
* [13] ‘Unless it’s modeling islands, I find most terrain generators unnatural, at least...’, Hacker News. [Online].
  Available: https://news.ycombinator.com/item?id=14794095
* [14] M. Mujtaba, ‘Simulating Hydraulic Erosion of Terrain’, gameidea. [Online].
  Available: https://gameidea.org/2023/12/22/simulating-hydraulic-erosion-of-terrain/
* [15] A. Paris, ‘Terrain Erosion on the GPU’, Make it Shaded. [Online].
  Available: https://makeitshaded.github.io/terrain-erosion/
* [16] B. Benes and R. Forsbach, ‘Layered data representation for visual simulation of terrain erosion’, in Proceedings
  Spring Conference on Computer Graphics, IEEE, 2001, pp. 80–86. [Online].
  Available: https://ieeexplore.ieee.org/abstract/document/945341/
* [17] A. J. Patel, mapgen4. (Apr. 2023). TypeScript. [Online]. Available: https://github.com/redblobgames/mapgen4
* [18] A. J. Patel, ‘Mapgen4: rainfall’, Red Blob Games. [Online].
  Available: https://simblob.blogspot.com/2018/09/mapgen4-rainfall.html
* [19] A. Paradise, E. Macdonald, K. Menou, C. Lee, and B. Fan, ‘Enabling new science with the ExoPlaSim 3D climate
  model’, Bulletin of the American Astronomical Society, vol. 53, no. 3, p. 1140, 2021.
* [20] A. Paradise, E. Macdonald, K. Menou, C. Lee, and B. L. Fan, ‘ExoPlaSim: Extending the planet simulator for
  exoplanets’, Monthly Notices of the Royal Astronomical Society, vol. 511, no. 3, pp. 3272–3303, 2022.
* [21] X. Mei, P. Decaudin, and B.-G. Hu, ‘Fast hydraulic erosion simulation and visualization on GPU’, in 15th Pacific
  Conference on Computer Graphics and Applications (PG’07), IEEE, 2007, pp. 47–56. Accessed: Mar. 30, 2025. [Online].
  Available: https://ieeexplore.ieee.org/abstract/document/4392715/
* [22] K. F. Fraedrich, H. Jansen, E. Kirk, U. Luksch, and F. Lunkeit, ‘The Planet Simulator: Towards a user friendly
  model’, Meteorologische Zeitschrift, vol. 14, no. 3, pp. 299–304, 2005.
* [23] J. Olsen, ‘Realtime procedural terrain generation’, 2004, Accessed: Mar. 30, 2025. [Online].
  Available: https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=5961c577478f21707dad53905362e0ec4e6ec644
* [24] L. Viitanen, ‘Physically based terrain generation: Procedural heightmap generation using plate tectonics’, 2012,
  Accessed: Mar. 30, 2025. [Online].
  Available: https://www.theseus.fi/bitstream/handle/10024/40422/Viitanen_Lauri_2012_03_30.pdf
* [25] G. Cordonnier et al., ‘Large Scale Terrain Generation from Tectonic Uplift and Fluvial Erosion’, Computer
  Graphics Forum, vol. 35, no. 2, pp. 165–175, May 2016, doi: 10.1111/cgf.12820.