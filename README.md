# Oblique-6 Hypersonic RANS Case (HiSA)

This repository contains an OpenFOAM/HiSA case for a **RANS simulation of a symmetrical diamond wedge at Mach 6**.  
The case uses the **HiSA (High Speed Aerodynamics Solver)** with the **AUSMPlusUp** flux scheme for compressible, high-speed flow calculations.

## Overview

The setup models external hypersonic flow over a 2D diamond wedge geometry (implemented as a thin extrusion with `frontAndBack` set to `empty`).

- **Flow regime:** Compressible, hypersonic
- **Target condition:** Mach ≈ 6 freestream
- **Solver:** `hisa`
- **Turbulence model:** RANS `SpalartAllmaras`
- **Flux scheme:** `AUSMPlusUp`
- **Thermophysical model:** Perfect gas, Sutherland viscosity, constant `Cp`

## Case Configuration

### Freestream conditions
Defined in `simulation/0.org/include/freestreamConditions`:

- `U = (1800 0 0) m/s`
- `p = 1600 Pa`
- `T = 224.65 K`

These values correspond to approximately Mach 6 for air.

### Numerical setup

- **Time integration:** `bounded dualTime ... steadyState` (pseudo-time marching)
- **Pseudo-time control:** local time stepping enabled
- **Linearization / flow solver:** `GMRES` with `LUSGS` preconditioning
- **Key reconstruction:** weighted van Leer (`wVanLeer`) for `rho`, `U`, and `T`

### Turbulence

From `simulation/constant/turbulenceProperties`:

- `simulationType RAS`
- `RASModel SpalartAllmaras`

### Mesh and boundaries

Mesh is generated using `blockMesh` from `mesh/system/blockMeshDict`.

- 4 structured blocks with near-body refinement
- Nominal block resolution per block: `(90 160 1)`
- Wedge wall split into two faces (`viscousWall`) to represent ramp-up/ramp-down diamond profile
- Symmetry segments upstream/downstream via `inviscidWall` (`symmetryPlane`)
- Farfield and inlet/outlet as external boundaries
- `frontAndBack` are `empty` (2D setup)

## Repository Layout

- `setupMesh` — builds the mesh in `mesh/` using `blockMesh`
- `cleanMesh` — cleans generated mesh/case artifacts in `mesh/`
- `runSim` — links mesh into `simulation/`, resets initial fields, decomposes, and runs in parallel
- `cleanSim` — removes simulation results/logs and cleans `simulation/`
- `mesh/` — mesh-generation case
- `simulation/` — HiSA simulation case, including `0.org`, `constant`, and `system`

## How to Run

From the repository root:

1. Build mesh
   ```bash
   ./setupMesh
   ```
2. Run simulation (parallel)
   ```bash
   ./runSim
   ```

`runSim` performs the following automatically:

- Creates symbolic links to `mesh/constant/polyMesh` inside `simulation/constant/polyMesh`
- Restores initial fields (`0.org -> 0`)
- Runs `decomposePar`
- Executes `hisa` in parallel
- Post-processes `MachNo` and `yPlus`

> Parallel decomposition is configured in `simulation/system/decomposeParDict` with `numberOfSubdomains 8` and `method scotch`.

## Output and Post-Processing

- Time directories are written in `simulation/`
- Force integration is enabled through `simulation/system/forces` on patch `viscousWall`
- Optional line sampling is configured in `simulation/system/sampleDict`
- Existing database/log helper files are present (`foamLog.db.*`)

### Assets

The files in `assets/` provide contour visualization data for **pressure**, **density**, **temperature**, and **Mach number**:

- `assets/results.vtu` — VTU dataset for local ParaView/VTK workflows
- `assets/overview.vtkjs` — VTK.js scene package for quick web viewing

Open locally in ParaView, or load either file in ParaView Glance: https://kitware.github.io/glance/

## Notes

- This is a focused benchmark-style case for hypersonic wedge-flow behavior using HiSA numerics.
- For clean reruns:
  - `./cleanSim` to reset simulation outputs
  - `./cleanMesh` to reset mesh outputs
