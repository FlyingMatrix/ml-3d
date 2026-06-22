# Machine Learning for 3D

A curated collection of machine learning models, pipelines, and tools designed for 3D data generation, processing and understanding.

### 🚀 Generative 3D Pipelines

```
[ Input: Single Image / Text Prompt ]
                 │
                 ▼
┌─────────────────────────────────┐
│     1. MULTI-VIEW DIFFUSION     │
├─────────────────────────────────┤
│ • Generates consistent novel    │
│   views (e.g., 4 to 6 angles)   │
│ • Predicts camera matrices      │
└─────────────────────────────────┘
                 │
                 │  (Multi-view Images + Camera Poses)
                 ▼
┌─────────────────────────────────┐
│      2. GAUSSIAN SPLATTING      │
├─────────────────────────────────┤
│ • Initializes a 3D point cloud  │
│ • Optimizes millions of 3D      │
│   Gaussians via differentiable  │
│   rasterization                 │
└─────────────────────────────────┘
                 │
                 │  (3D Gaussian Scene / Density Field)
                 ▼
┌─────────────────────────────────┐
│     3. MESHES (MARCHING CUBES)  │
├─────────────────────────────────┤
│ • Converts Gaussians to a       │
│   continuous density/TSDF grid  │
│ • Extracts the isosurface using │
│   the Marching Cubes algorithm  │
└─────────────────────────────────┘
                 │
                 ▼
[ Output: Textured 3D Mesh (.obj/.gltf) ]
```

### 🧩 Detailed Pipeline Breakdown

1. **Multi-view Diffusion (The Concept Creator):**
   
   - **Input:** A single 2D image or a descriptive text prompt.
   
   - **Process:** A diffusion model (like MVDream, Wonder3D, or InstantMesh) hallucinates and generates multiple synthetic viewpoints of the object simultaneously, ensuring geometric consistency across the frames.

2. **Gaussian Splatting (The 3D Representation):**
   
   - **Process:** The system takes the generated multi-view images and uses them to optimize a set of 3D Gaussians. It quickly refines their position, scale, rotation, opacity, and color to match the diffusion-generated images perfectly.

3. **Meshes via Marching Cubes (The Final Asset):**
   
   - **Process:** Because 3D Gaussians are "cloud-like" and don't have a solid surface, a density field is extracted from them. The **Marching Cubes** algorithm then processes this 3D grid, creating a solid polygonal mesh topology that can be easily rigged, animated, or imported into game engines.

### 🔮 Differentiable Rasterization

In traditional computer graphics, rendering is a one-way street: you take 3D data (triangles, textures, cameras) and convert it into a 2D image of pixels. This process is called **rasterization**. 

However, standard rasterization is **non-differentiable** because it involves discrete steps—like determining exactly which triangle covers a specific pixel. In mathematical terms, this creates a discontinuous step function, making it impossible to calculate derivatives (gradients). 

**Differentiable Rasterization** fixes this limitation. It bridges the gap between traditional computer graphics and modern machine learning/computer vision, which allows us to solve **Inverse Graphics** problems: instead of turning 3D into 2D, we can take a 2D image and figure out the 3D scene that matches it.

- **3D Reconstruction:** You can feed an AI a single 2D photo of an object, and by comparing the AI's rendered guess to the real photo, the system can automatically tweak a 3D mesh until it perfectly matches the photo.

- **Texture and Lighting Estimation:** It can automatically extract the exact texture maps or lighting conditions of a real-world environment just by analyzing video footage.

- **Analysis-by-Synthesis:** It allows neural networks to "understand" the physical 3D world by rendering their own guesses and correcting their mistakes mathematically.

### 🎨 Gaussian Splatting

Gaussian Splatting is a **differentiable rasterization technique**. **Splats** are composed of millions of points, where each point is composed of four parameters:

- **Position**: where it’s located (XYZ)
- **Covariance**: how it’s stretched (3x3 matrix)
- **Color**: what color it is (RGB)
- **Alpha**: how transparent it is (α)

Then, to rasterize a splat, these points are projected into 2D. Then, for every pixel, contribute the contribution of every point. In pseudo code:

```
splat2d = splat.project_and_sort()
for point in splat2d:
    for pixel in image:
        pixel += compute_contribution(point, pixel)
```

The contribution of a point diminishes the further it is from the pixel. The points also need to be sorted, since they are blended back-to-front.

The [original paper](https://huggingface.co/papers/2308.04079) intializes the points using [Structure-from-Motion](https://en.wikipedia.org/wiki/Structure_from_motion), a traditional algorithm for 3D reconstruction. These points are then rasterized using the tile-based method, and the loss is computed by comparing the rasterized image to the ground truth. Gradient descent is applies to adjust the point parameters (position, covariance, color, alpha). The automated densification and pruning are also used to automatically add and remove points as needed.




