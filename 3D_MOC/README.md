# 3D_MOC (3DCalc): 3D Method of Characteristics Flowfield Solver

Part of the [Three-Dimensional Nozzle Design Code](../README.md) (modified edition of NASA LEW-20180).

3D_MOC computes the **three-dimensional supersonic flowfield inside a given nozzle wall**. The wall does not have to be axisymmetric. It marches a 3D method-of-characteristics solution from an initial supersonic plane down the nozzle. At each step it finds new points from bicharacteristics traced back to the previous plane and along streamlines. Wall points are placed where the streamline meets a spline fit of the wall surface.

Typical uses:

- **Check a modified contour.** Take a MOC_Grid_BDE design, change it (offset, scarf, ovalise) and see how the flow responds.
- **Analyse a non-axisymmetric nozzle.** Compute the flowfield of a non-axisymmetric or streamline-traced nozzle (from [STT2001](../STT2001/README.md)).
- **Validate the solver.** Compare it with a 2D/axisymmetric MOC result on the same geometry. The `outputs_M4Perfect`, `outputs_M4RAO` and `outputs_cone10` samples are cases of this kind.

Full theory: Rice (2003), JHU/APL RTDC-TPS-481.

---

## Running the program

Run `3DCalc.exe`. The **3D Method of Characteristics Tool** dialog opens. Output files are written to the current working folder, and the geometry file is read from there too.

### Geometry input file (`*.geo`)

The wall is described as a stack of **circular cross-sections**. Each section can have its own radius and its own centre offset, which makes off-axis and non-axisymmetric shapes possible.

```
162                       <- number of axial stations (nZ)
Z   R0   X0   Y0          <- header line (ignored)
0         1        0  0   <- z, section radius, centre x, centre y
0.000212  1        0  0
...
```

Units are consistent length units; the samples use inches. The first station is the initial plane. Examples: `cone10.geo` (in this folder), `outputs_M4Perfect/M4perfect.geo`, `outputs_M4RAO/m4Rao.geo`.

### Inputs (dialog)

| Group | Field | Meaning | Default |
|---|---|---|---|
| **File Input/Output** | Geometry Input File | `*.geo` wall definition | `ideal.geo` |
| | Output File | Summary output name | `Summary.out` |
| **Initial Plane Properties** | Pressure (psia), Temperature (R) | Static conditions on the initial plane | 1000, 530 |
| | Mach Number | Initial-plane Mach number (must be >= 1) | 1.1 |
| | Mol. Wt., Gamma | Gas properties | 28.96, 1.4 |
| | Theta, Psi (deg) | Initial flow angles | 0, 0 |
| | Axial Location (in) | Axial position of the initial plane | 0 |
| **Grid Setup (cylindrical)** | Radial Divisions | Points around the circumference of each wall section | 36 |
| | Number of Ray Pts | Points along each radial ray | 0 |
| | Axial Stations | Number of marching planes | 11, range 10-1000 |
| **Surface Fit** | drop-down | Wall interpolation: *All Point Spline* or *9 Point Spline* | All Point Spline |
| **Print Output Parameters** | Every *n* point(s) in X / Y / Z | Output thinning | 1 / 1 / 10 |
| | Every *n* Step Number | Step-output frequency | 999 |

Press **Calculate Nozzle**. The calculation runs in a worker thread, and *Step Number / Total Steps* show progress.

### Outputs

| File | Contents |
|---|---|
| `Initial Wall.plt` | Tecplot: wall built from the `.geo` file (check this first) |
| `full_mesh.plt` | Tecplot: complete 3D solution mesh with flow properties |
| `axialStations.plt` | Tecplot: solution on each axial marching plane |
| `Wall.plt` | Tecplot: wall surface with properties (pressure, Mach, ...) |
| `Streamlines.plt` | Tecplot: streamlines through the field |
| `streamtube.plt` | Tecplot: stream tube through the streamlines listed in `SL.inp` (first line = count, then streamline numbers), written only if `SL.inp` is in the working folder |
| `z=0.out` | Properties in the *z = 0* symmetry plane |
| `outfile.out` | Point-by-point solver output (grid indices and coordinates) |

---

## Sample cases

| Folder | Case |
|---|---|
| `outputs_M4Perfect/` | Mach 4 perfect axisymmetric nozzle (`M4perfect.geo`) |
| `outputs_M4RAO/` | Mach 4 Rao nozzle (`m4Rao.geo`), with stream-tube example `SL.inp` and Tecplot layouts (`*.lay`) |
| `outputs_cone10/` | 10° conical nozzle (`cone10.geo`) |

These were produced by the original NASA code. To rerun one, copy its `.geo` file next to `3DCalc.exe`, enter the file name in the dialog, and press **Calculate Nozzle**.

---

## Building from source

Requirements: Visual Studio 2022+ with *Desktop development with C++* and **C++ MFC for latest build tools (x86 & x64)**.

1. Open `3DCalc.sln`. Accept the retarget prompt if one appears.
2. Choose **Release | x64** and run **Build > Rebuild Solution**.
3. The output is `x64\Release\3DCalc.exe`, a standalone exe.

The project files and the Numerical Recipes interface changes (`nr.h`, `3D_MOCGrid.hpp`) that make this compile on a modern compiler were contributed by **Bleialf** (2022). The statically linked Release|x64 configuration was added in this edition.

---

## Source files

| File | Role |
|---|---|
| `3D_MOCGrid.cpp/.hpp` | Core 3D MOC solver: initial plane, field and wall point unit processes, surface fits, output |
| `3D_MOCGridThread.cpp/.h` | Worker thread that runs the march |
| `3D_MOCDlg.cpp/.h` | Main dialog |
| `point.cpp/.hpp` | Grid point data structure |
| `fmin.cpp`, `lnsrch.cpp`, `lubksb.cpp`, `ludcmp.cpp`, `newt.cpp`, `sort2.cpp`, `nr*.h` | Numerical Recipes routines (Newton solver, LU decomposition, sorting) |
| `engineering_constants.hpp`, `dummyStruct.h`, `UserMessages.h` | Constants and helpers |
| `3D_MOC.cpp/.h`, `res/3D_MOC.rc`, `resource.h` | MFC application and resources |

> The Numerical Recipes routines are included as in the NASA release. Numerical Recipes code
> carries its own license terms from the book's publisher. Check those terms before reusing
> the routines outside this program.

## Reference

Rice, T. (2003). *2D and 3D Method of Characteristic Tools for Complex Nozzle Development*. JHU/APL RTDC-TPS-481. [NTRS 20030067852](https://ntrs.nasa.gov/citations/20030067852)

License: NASA Open Source Agreement v1.3. See [../license.txt](../license.txt) and [../NOTICE.md](../NOTICE.md).
