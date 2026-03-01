# OpenCascade Mesh Boolean Operations - User Guide

## Overview
The CAD editor now supports **boolean operations on GLB/mesh models** using OpenCascade.js kernel. This means you can combine, subtract, or intersect AI-generated 3D models!

## How It Works

### 1. **Mesh to BREP Conversion**
When you import a GLB file (from AI or marketplace), the app automatically:
- Extracts polygon mesh data (vertices + indices)
- Stores it in the features list

The new `meshToShape()` function converts this mesh data into OpenCascade BREP (Boundary Representation) solids, which enables precision CAD operations.

### 2. **Boolean Operations Available**
- **Union** ➕ - Combines two models into one
- **Cut** ➖ - Subtracts tool from base (creates holes, cavities)
- **Intersect** ⚔️ - Keeps only overlapping volume

### 3. **Process Flow**
```
GLB Model → Mesh Data → BREP Shape → Boolean Op → BREP Result → Mesh Data → Three.js Render
```

## Usage Instructions

### Step 1: Generate or Import Models
1. Use **Image to 3D AI** (magic wand icon) to generate GLB models
2. Or ask **Torquy AI** to create 3D primitives (cube, sphere, cylinder)
3. Or extrude 2D sketches into 3D solids

### Step 2: Open Boolean Operations Panel
1. Click the **sidebar toggle** (right side of canvas)
2. Scroll to **"Boolean Operations (Mesh)"** panel
3. Ensure you have at least 2 mesh features

### Step 3: Select Operation Type
Choose between:
- **Union**: Merge models together
- **Cut**: Subtract one from another (useful for holes)
- **Intersect**: Keep only shared volume

### Step 4: Select Features
1. **Base Feature**: The primary object to operate on
2. **Tool Feature**: The object that modifies the base

Example:
- Base: Large cube
- Tool: Small cylinder
- Operation: Cut
- Result: Cube with cylindrical hole

### Step 5: Execute
Click **"Execute [OPERATION]"** button. The CAD kernel will:
- Convert meshes to BREP shapes
- Perform boolean operation
- Convert result back to mesh
- Display in 3D viewport

## Technical Details

### `meshToShape(meshData)` - CADGeometryService.js
Converts mesh to OpenCascade solid:
- Iterates through triangles
- Creates edges and wires for each triangle
- Uses `BRepBuilderAPI_Sewing` to join faces
- Attempts to create a closed solid

**Note**: Complex meshes with holes or non-manifold geometry may fail to create valid solids.

### `meshBooleanOperation(mesh1, mesh2, operation)`
High-level wrapper that:
1. Converts both meshes to BREP
2. Calls `booleanUnion()`, `booleanCut()`, or `booleanIntersect()`
3. Converts result back to mesh
4. Returns mesh data ready for Three.js

## Limitations & Tips

### ✅ Works Best With:
- Clean, closed meshes (watertight)
- AI-generated models from Image-to-3D
- Extruded 2D sketches
- Simple primitives (cubes, spheres, cylinders)

### ⚠️ May Fail With:
- Open meshes (non-manifold)
- Self-intersecting geometry
- Very high-poly meshes (>50k triangles)
- Models with holes or gaps

### Performance Tips:
- Use lower polygon count models when possible
- Operations are CPU-intensive and may take 5-30 seconds
- Large meshes are processed in batches (1000 triangles at a time)
- Results are automatically added as new features (original models remain)

## Example Workflows

### Example 1: Create a Hollow Box
1. Generate cube using Torquy: "Create a red cube with size 20"
2. Generate smaller cube: "Create a cube with size 15"
3. Open Boolean Operations panel
4. Select large cube as **Base**, small cube as **Tool**
5. Choose **Cut** operation
6. Execute → Result: Hollow box

### Example 2: Merge AI Models
1. Upload image of a gear to Image-to-3D
2. Upload image of a shaft to Image-to-3D
3. Use transform controls (W/E/R keys) to position them
4. Open Boolean Operations panel
5. Select gear as **Base**, shaft as **Tool**
6. Choose **Union** operation
7. Execute → Result: Merged gear+shaft assembly

### Example 3: Find Overlap
1. Create two intersecting spheres with Torquy
2. Select sphere1 as **Base**, sphere2 as **Tool**
3. Choose **Intersect** operation
4. Execute → Result: Lens-shaped intersection volume

## Export
After performing boolean operations, you can:
- **Download as STL** for 3D printing
- **Download as GLB** for sharing
- **Upload to Marketplace** for selling

## Troubleshooting

**"Mesh to shape conversion failed"**
- Try simplifying the mesh in external tool
- Ensure model is closed/watertight
- Check for duplicate vertices

**"Boolean operation failed"**
- Ensure models actually overlap (for Cut/Intersect)
- Try swapping Base and Tool
- Check that both features have valid mesh data

**Result looks broken**
- Increase sewing tolerance (hardcoded at 1e-6)
- Some geometry may be too complex for BREP conversion
- Try using simpler shapes

## Future Enhancements
Potential improvements:
- Vertex-level mesh editing
- Mesh smoothing/subdivision
- Direct mesh simplification
- Preview before executing
- Undo support for boolean operations
- Multi-step operation history

---

**Powered by OpenCascade.js** - Industry-standard CAD kernel compiled to WebAssembly
