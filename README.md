# Molecular Dynamics Simulation of Ballistic Penetration of a Tungsten Projectile into a TPMS-Core Aluminum Sandwich Panel

![TPMS sandwich panel](tpms_panel_preview.png)

![LAMMPS](https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge)
![Projectile](https://img.shields.io/badge/Tungsten-Projectile-silver?style=for-the-badge)
![Target](https://img.shields.io/badge/Al-TPMS%20Sandwich%20Panel-orange?style=for-the-badge)
![Core](https://img.shields.io/badge/Core-Gyroid%20%7C%20Primitive%20%7C%20Diamond-yellow?style=for-the-badge)
![Impact](https://img.shields.io/badge/Ballistic-Penetration-red?style=for-the-badge)
![Potential](https://img.shields.io/badge/EAM-CuAlW%20Potential-purple?style=for-the-badge)
![OVITO](https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge)
[![DOI](https://img.shields.io/badge/DOI-10.5281%2Fzenodo.23081503-blue?style=for-the-badge)](https://doi.org/10.5281/zenodo.23081503)
[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge)](http://creativecommons.org/licenses/by-nc/4.0/)

A fully atomistic **molecular dynamics simulation of ballistic penetration** of a BCC tungsten spherical projectile into an aluminum **sandwich panel with a triply periodic minimal surface (TPMS) core**, using LAMMPS. The panel consists of two solid FCC Al face sheets bonded to an Al TPMS lattice core (Gyroid, Schwarz Primitive, or Schwarz Diamond). The faces and the core are carved from a single continuous FCC lattice, so the panel is monolithic, as in an additively manufactured part.

The simulation captures projectile deceleration, energy absorption, stress wave transmission from the front face through the lattice core to the back face, core wall buckling and crushing, face sheet petaling, and spallation, using the *EAM/alloy potential (CuAlW.txt)*. Von Mises stress and hydrostatic pressure are computed per atom, and additionally averaged separately for the top face, the TPMS core, and the bottom face.

![Uploading imp.png…]()


---

## Physics

The simulation captures the full sequence of ballistic penetration of a sandwich structure at the atomistic scale:

- FCC aluminum sandwich panel and BCC tungsten sphere modeled with the EAM/alloy interatomic potential (CuAlW.txt)
- TPMS core generated directly inside LAMMPS from a level-set function using atom-style variables (no external geometry files)
- Sheet-network TPMS: core atoms are kept only where |f(x, y, z)| < t, so the level-set parameter t controls the core relative density
- Face sheets and core share one lattice, giving a perfectly bonded face/core interface
- Tungsten projectile assigned an impact velocity of 1500 m/s directed normal to the panel
- Clamped boundary conditions on the panel edges through the full thickness
- NVE integration for projectile and panel during the impact phase for full momentum conservation
- Per-atom stress tensor via `compute stress/atom`; von Mises stress and hydrostatic pressure extracted at all phases
- Component-resolved stress (top face, core, bottom face) to show how load is shared through the sandwich
- Projectile centre-of-mass position, velocity, and translational kinetic energy tracked to give residual velocity and absorbed energy

---

## Impact Regime

| Regime               | Velocity Range    | This Simulation |
| -------------------- | ----------------- | --------------- |
| Low velocity impact  | < 100 m/s         | No              |
| High velocity impact | 100 to 1000 m/s   | No              |
| **Ballistic impact** | **1000 to 3000 m/s** | **Yes (1500 m/s)** |
| Hypervelocity impact | > 3000 m/s        | No              |

---

## TPMS Core Geometries

The core is defined by one of three level-set functions, with X = kx, Y = ky, Z = k(z - z_core_bottom) and k = 2 pi / L:

| TPMS             | Script keyword | Level-set function f(X, Y, Z)                                                      |
| ---------------- | -------------- | ---------------------------------------------------------------------------------- |
| Gyroid           | `gyr`          | sin X cos Y + sin Y cos Z + sin Z cos X                                            |
| Schwarz Primitive| `pri`          | cos X + cos Y + cos Z                                                              |
| Schwarz Diamond  | `dia`          | sin X sin Y sin Z + sin X cos Y cos Z + cos X sin Y cos Z + cos X cos Y sin Z      |

Atoms of the core block are deleted where |f| > t. Approximate core relative density for a cell size L = 40.5 Angstrom:

| TPMS      | t = 0.3 | t = 0.4 | t = 0.5 | t = 0.6 | t = 0.8 |
| --------- | ------- | ------- | ------- | ------- | ------- |
| Gyroid    | 0.20    | 0.26    | 0.32    | 0.38    | 0.52    |
| Primitive | 0.14    | 0.26    | 0.30    | 0.32    | 0.41    |
| Diamond   | 0.30    | 0.30    | 0.39    | 0.58    | 0.68    |

The script prints the exact relative density of the generated core at startup.

---

## Geometry

```
Simulation box (units: Angstrom):

z = 146.7 +------------------------------------------+  <- box top
          |                                          |
z =  96.7 |          ( W ball, r = 25 Ang )          |  <- center (101.25, 101.25, 96.7)
          |                 | v = -15 Ang/ps         |
          |                 v                        |
z =  56.7 +==========================================+  <- top face sheet top
          |        TOP FACE SHEET (8.1 Ang)          |
z =  48.6 +------------------------------------------+
          |  ~~~~  TPMS CORE (Gyroid, 40.5 Ang) ~~~~ |
          |  ~~~~  sheet-network lattice     ~~~~~~~ |
z =   8.1 +------------------------------------------+
          |       BOTTOM FACE SHEET (8.1 Ang)        |
z =   0.0 +==========================================+  <- panel bottom
          |                                          |
z = -100  +------------------------------------------+  <- box bottom

Box XY: 0 to 202.5 Angstrom (periodic in x and y, 50 Al lattice constants)
Box Z:  -100 to 146.7 Angstrom (free in z)
TPMS cells: 5 x 5 in-plane, 1 through the core thickness (L = 40.5 Ang)
Panel edges: clamped (setforce 0) - 4 Ang border through full thickness
```

- **Face sheets**: FCC Al, 202.5 x 202.5 x 8.1 Angstrom each (2 lattice constants)
- **TPMS core**: Al Gyroid, cell size L = 40.5 Angstrom (10 lattice constants), core height 40.5 Angstrom, t = 0.6
- **Total panel thickness**: 56.7 Angstrom
- **W ball**: BCC tungsten sphere, radius 25 Angstrom, released 15 Angstrom above the panel
- **Impact velocity**: -15 Angstrom/ps (1500 m/s) in the -z direction
- **Boundary**: periodic in x and y; free in z

The box width is an exact multiple of both the Al lattice constant and the TPMS cell size, so the lattice and the core are both continuous across the periodic boundaries.

### Default System Size

| Component         | Atom type | Atoms   |
| ----------------- | --------- | ------- |
| Al face sheets    | 1         | 50,000  |
| Al TPMS core      | 2         | 36,400  |
| W projectile      | 3         | 4,128   |
| **Total**         | -         | **90,528** |

Core relative density with the default settings: 0.383.

---

## Simulation Phases

| Phase                  | Run Steps  | Simulated Time | Description                                                      |
| ---------------------- | ---------- | -------------- | ---------------------------------------------------------------- |
| Minimization           | variable   | -              | Panel relaxed with ball frozen; TPMS free surfaces relax         |
| Equilibration          | 3,000      | 3 ps           | Panel thermalized at 300 K (NVT); ball frozen in place           |
| Approach               | ~1,000     | ~1 ps          | Ball crosses the 15 Angstrom gap to first contact                |
| Impact and Penetration | ~19,000    | ~19 ps         | Front face perforation, core crushing, back face loading or exit |
| Post-impact relaxation | 5,000      | 5 ps           | Panel thermostatted at 300 K; stress waves dissipate             |
| **Total MD**           | **28,000** | **28 ps**      | -                                                                |

The approach and impact phases run together as one 20,000-step NVE run.

---

## Key Event Timeline

Times are measured from the release of the ball and assume constant velocity before contact. After contact the ball decelerates, so later events occur later than a constant-velocity estimate; use `ball_history.dat` for the actual times.

| Event                                  | Time after release | Ball center z (Angstrom) |
| -------------------------------------- | ------------------ | ------------------------ |
| Ball released                          | 0 ps               | 96.7                     |
| Ball bottom touches top face           | ~1.0 ps            | 81.7                     |
| Ball center reaches top face surface   | > 2.7 ps           | 56.7                     |
| Ball center reaches core midplane      | > 4.3 ps           | 28.4                     |
| Ball center reaches panel bottom       | > 6.4 ps           | 0.0                      |

Whether the ball perforates the panel, embeds in the core, or rebounds depends on the impact velocity, TPMS type, relative density, and face sheet thickness.

---

## Material Parameters

### Aluminum Panel

| Parameter              | Symbol | Value                   | Unit     |
| ---------------------- | ------ | ----------------------- | -------- |
| Crystal structure      | -      | FCC                     | -        |
| Lattice constant       | a      | 4.05                    | Angstrom |
| Atomic mass            | m      | 26.982                  | g/mol    |
| Panel in-plane size    | -      | 202.5 x 202.5           | Angstrom |
| Face sheet thickness   | t_f    | 8.1                     | Angstrom |
| Core thickness         | h_c    | 40.5                    | Angstrom |
| TPMS cell size         | L      | 40.5                    | Angstrom |
| TPMS level-set value   | t      | 0.6                     | -        |

### Tungsten Projectile

| Parameter         | Symbol | Value  | Unit                     |
| ----------------- | ------ | ------ | ------------------------ |
| Crystal structure | -      | BCC    | -                        |
| Lattice constant  | a      | 3.165  | Angstrom                 |
| Atomic mass       | m      | 183.84 | g/mol                    |
| Sphere radius     | r      | 25     | Angstrom                 |
| Impact velocity   | v      | -15    | Angstrom/ps (1500 m/s)   |

### Interatomic Potential

| Parameter       | Value                          |
| --------------- | ------------------------------ |
| Potential file  | CuAlW.txt                      |
| Potential style | EAM/alloy                      |
| Type 1          | Al (face sheets)               |
| Type 2          | Al (TPMS core)                 |
| Type 3          | W (projectile)                 |
| pair_coeff      | `* * CuAlW.txt Al Al W`        |

Types 1 and 2 are the same element. They are kept separate only so that the face sheets and the core can be colored and analyzed independently.

### Boundary Conditions

| Group                     | Condition        | Description                                |
| ------------------------- | ---------------- | ------------------------------------------ |
| Panel edges (4 Angstrom)  | `setforce 0 0 0` | Clamped through the full panel thickness   |
| Panel inner               | NVE              | Full impact dynamics                       |
| W ball                    | NVE              | Conserves projectile momentum              |
| Z boundary                | Free (f)         | Allows the ball and debris to leave below  |

---

## Governing Equations

### TPMS Core Construction

```
k = 2*pi / L

keep core atom   if  |f(kx, ky, k(z - z_core_bottom))| <= t
delete core atom if  |f(kx, ky, k(z - z_core_bottom))| >  t

relative density  rho* = N_core(after carving) / N_core(solid block)
```

### Ballistic Velocity and Energy

```
v_impact = -15 Ang/ps = 1500 m/s

Translational kinetic energy of the ball (centre of mass):
KE_cm = 0.5 * M_ball * |v_cm|^2 * 1.036427e-4   [eV]

where:
  M_ball = N_W * 183.84 amu
  v_cm   = centre-of-mass velocity of the ball (Ang/ps)
  1 amu*Ang^2/ps^2 = 1.036427e-4 eV

Energy absorbed by the panel:
E_abs = KE_cm(release) - KE_cm(t)

Energy absorbed per panel atom:
E_abs_n = E_abs / N_Al
```

The centre-of-mass kinetic energy is used instead of the total kinetic energy of the ball, so the ball's own heating is not counted as absorbed energy.

### EAM/Alloy Potential Energy

```
E_i = F_alpha( sum_{j != i} rho_beta(r_ij) ) + 0.5 * sum_{j != i} phi_alpha_beta(r_ij)

where:
  F_alpha = embedding energy function
  rho     = electron density contribution
  phi     = pair interaction potential
```

### Per-Atom Stress Tensor (LAMMPS units: bar * Ang^3)

```
sigma_ab = (1/V_i) * [ -m_i * v_ia * v_ib + 0.5 * sum_j (r_iab * f_ijb) ]
```

### Von Mises Stress

```
sigma_VM = sqrt( 0.5 * [ (sxx-syy)^2 + (syy-szz)^2 + (szz-sxx)^2
                        + 6*(sxy^2 + sxz^2 + syz^2) ] )
```

### Hydrostatic Pressure

```
P = -(sxx + syy + szz) / 3

Compression: P > 0  (beneath impact zone)
Tension:     P < 0  (spallation zone, back face)
```

---

## Group Definitions

```
al_panel      <- all Al atoms (types 1 and 2)
face_sheets   <- Al face sheet atoms (type 1)
face_top      <- top face sheet atoms   (z >= core top)
face_bot      <- bottom face sheet atoms (z <= core bottom)
tpms_core     <- Al TPMS core atoms (type 2)
w_ball        <- W sphere atoms (type 3)
sheet_inner   <- Al atoms excluding the 4 Angstrom edge border (free NVE)
sheet_edge    <- Al border atoms, clamped with setforce 0 0 0
```

---

## Output Files

| File                        | Frequency       | Content                                                                                   |
| --------------------------- | --------------- | ----------------------------------------------------------------------------------------- |
| `panel_initial.data`        | once            | Panel and ball geometry before minimization (inspect the TPMS core in OVITO)              |
| `impact_stress.lammpstrj`   | every 100 steps | id, type, x, y, z, vx, vy, vz, von Mises, pressure, full stress tensor                    |
| `impact_vonmises.lammpstrj` | every 100 steps | id, type, x, y, z, von Mises, hydrostatic pressure                                        |
| `stress_history.dat`        | every 100 steps | avg/max von Mises, avg/min/max pressure, avg von Mises in top face, core, bottom face     |
| `ball_history.dat`          | every 100 steps | ball z, ball speed (m/s), centre-of-mass KE, total KE, absorbed energy                    |
| `impact_final.data`         | end             | Final atomic positions and velocities (LAMMPS data format)                                |
| `impact_final.restart`      | end             | Binary restart file for continuation runs                                                 |

At the end of the run the log reports the initial and residual ball speed, the final ball position, the absorbed energy (total and per Al atom), and the final stress values for each panel component.

---

## Repository Structure

```
tpms_sandwich_ballistic_md/
|
|-- tungsten_impact_tpms_sandwich.lammps   # Main LAMMPS simulation script
|-- CuAlW.txt                              # EAM potential file (required)
|-- tpms_panel_preview.png                 # Geometry preview
|-- README.md                              # This file
|
|-- outputs/                               # Generated on run
    |-- panel_initial.data
    |-- impact_stress.lammpstrj
    |-- impact_vonmises.lammpstrj
    |-- stress_history.dat
    |-- ball_history.dat
    |-- impact_final.data
    |-- impact_final.restart
```

---

## How to Run

### Requirements

- LAMMPS (any recent version with the MANYBODY package): <https://www.lammps.org>
- EAM potential file `CuAlW.txt` in the same directory as the script
- OVITO or VMD for trajectory visualization: <https://www.ovito.org>

### Step 1: Choose the core

Edit the TPMS parameters at the top of the script:

```
variable        tpms      string gyr      # gyr | pri | dia
variable        t_level   equal 0.6       # level-set value (relative density)
variable        n_cell_a  equal 10        # TPMS cell size in lattice units
variable        n_cells_x equal 5         # cells in x and y
variable        n_cells_z equal 1         # cells through the core thickness
variable        n_face_a  equal 2         # face sheet thickness in lattice units
```

### Step 2: Run the simulation

Serial:

```
lmp -in tungsten_impact_tpms_sandwich.lammps
```

Parallel with MPI (14 physical cores):

```
mpirun -np 14 lmp -in tungsten_impact_tpms_sandwich.lammps
```

For MPI runs, add `processors * * 1` after `atom_style atomic`. This splits the domain only in x and y, so every processor holds part of the panel instead of empty space above or below it.

On WSL, running from the Linux home directory is faster than running from a mounted Windows drive (`/mnt/c`, `/mnt/d`), because the trajectory dumps are large.

### Step 3: Visualize in OVITO

1. Open OVITO, then `File > Load File` and select `panel_initial.data` to check the TPMS core
2. Load `impact_vonmises.lammpstrj` for the impact trajectory
3. Color by **Particle Type** to separate face sheets (1), TPMS core (2), and projectile (3)
4. Add **Color Coding** on `v_von_mises` to follow the stress wave
5. Add **Slice** at x = 101.25 Angstrom (normal along x, slab width about 5 Angstrom) for the penetration cross-section
6. Use **Common Neighbor Analysis (CNA)** to identify disordered Al around the penetration channel and in crushed core walls
7. Plot `ball_history.dat` (ball speed against time) to see the deceleration in each layer

---

## What to Look for in Results

### Front Face Perforation

At first contact a compressive stress wave spreads radially through the top face sheet. The ball shears and petals the top face before it reaches the core.

### Core Crushing and Wave Dispersion

The TPMS walls under the projectile buckle and crush progressively. Because the core is mostly empty space, the stress wave reaching the back face is weaker and more spread out than in a solid plate of equal mass. Compare `c_vm_top`, `c_vm_core`, and `c_vm_bot` in `stress_history.dat` to see this attenuation.

### Back Face Response

The back face is loaded by the crushed core and the projectile. Depending on the remaining projectile energy, it bulges, petals, or is perforated. Negative hydrostatic pressure on the back face indicates spallation driven by reflected tensile waves.

### Energy Absorption

`ball_history.dat` gives the absorbed energy over time. The slope of the speed curve shows which layer removes the most energy. Dividing the absorbed energy by the number of Al atoms gives a mass-normalized measure for comparing different TPMS types and relative densities.

### Residual Stress

After the ball exits or stops, residual stress remains around the hole in each face sheet and in the deformed core walls.

---

## Common Errors and Fixes

| Error or symptom                              | Cause                                         | Fix                                                                        |
| --------------------------------------------- | --------------------------------------------- | -------------------------------------------------------------------------- |
| `Cannot open potential file CuAlW.txt`        | Potential file not found                      | Place `CuAlW.txt` in the run directory                                      |
| `Number of element mappings does not match`   | pair_coeff mapping does not match 3 types     | Use `pair_coeff * * CuAlW.txt Al Al W`                                      |
| Ball speed is about 3 times the set value     | `velocity set` interpreted in lattice units   | Keep `units box` on the `velocity w_ball set` command                       |
| Thin TPMS walls fall apart during minimization| Walls thinner than about two atomic layers    | Increase `t_level` or use a larger cell (`n_cell_a`)                        |
| Core does not tile across the boundary        | Box width not a multiple of the cell size     | Keep `n_cells_x * n_cell_a * 4.05` equal to the box width                   |
| `Lost atoms` error                            | Spalled debris leaves the free z boundary     | Already handled by `thermo_modify lost ignore`                              |
| Ball stops inside the core                    | Panel absorbs all projectile energy           | Expected for dense cores; raise `v_impact` or `ball_r` to study perforation |
| `1 by 1 by 1 MPI processor grid` with mpirun  | LAMMPS binary not built with MPI              | Use an MPI-enabled LAMMPS build                                             |

---

## Extending the Model

| Extension                          | What to change                                                              |
| ---------------------------------- | --------------------------------------------------------------------------- |
| Different TPMS                     | Set `tpms` to `gyr`, `pri`, or `dia`                                        |
| Different relative density         | Change `t_level`                                                            |
| Finer or coarser lattice           | Change `n_cell_a` and adjust `n_cells_x` to keep the box periodic           |
| Thicker core                       | Increase `n_cells_z`                                                        |
| Graded core                        | Make `t_level` a function of z inside the `tpms_void` variable              |
| Solid-network TPMS                 | Replace `abs(v_f_${tpms})>${t_level}` with `v_f_${tpms}>${t_level}`         |
| Thicker or asymmetric face sheets  | Change `n_face_a` (or define separate top and bottom thicknesses)           |
| Higher velocity                    | Change `v_impact`                                                           |
| Oblique impact                     | Set non-zero x or y components in `velocity w_ball set`                     |
| Different panel material (Cu)      | Change the lattice, masses, and pair_coeff element mapping                  |
| Monolithic plate comparison        | Set `t_level` large enough to keep the whole core, then compare at equal mass |

---

## Citation

If you use this code in your research, please cite:

```
@software{mishra_2026_tpms_sandwich_ballistic,
  author    = {Mishra, Akshansh},
  title     = {Molecular Dynamics Simulation of Ballistic Penetration of a
               Tungsten Projectile into a TPMS-Core Aluminum Sandwich Panel},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.23081503},
  url       = {https://doi.org/10.5281/zenodo.23081503}
}
```

Plain text citation:

> Mishra, A. (2026). *Molecular Dynamics Simulation of Ballistic Penetration of a Tungsten Projectile into a TPMS-Core Aluminum Sandwich Panel* [Computer software]. Zenodo. <https://doi.org/10.5281/zenodo.23081503>

---

## Author

**Akshansh Mishra**
GitHub: <https://github.com/akshansh11>

---

## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

You are free to:

- **Share**: copy and redistribute the material in any medium or format
- **Adapt**: remix, transform, and build upon the material

Under the following terms:

- **Attribution**: You must give appropriate credit to Akshansh Mishra and provide a link to this repository
- **NonCommercial**: You may not use the material for commercial purposes

Copyright 2026 Akshansh Mishra. All rights reserved for commercial use.
