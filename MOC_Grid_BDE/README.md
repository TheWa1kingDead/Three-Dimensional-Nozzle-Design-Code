# MOC_Grid_BDE: 2D Method of Characteristics Nozzle Design

Part of the [Three-Dimensional Nozzle Design Code](../README.md) (modified edition of NASA LEW-20180).

MOC_Grid_BDE designs the divergent contour of a supersonic nozzle, either **axisymmetric** or **planar (2D)**, using the method of characteristics for an isentropic, calorically perfect gas. It starts from a transonic solution at the throat, marches the characteristic net downstream, and computes the wall contour that meets the chosen design condition. It also writes the full characteristic grid and a set of streamlines, which can be passed on to [STT2001](../STT2001/README.md) or used to build a wall for [3D_MOC](../3D_MOC/README.md).

---

## Method in brief

1. **Throat / initial data line.** The sonic-region flow is computed from the series solutions of **Hall (1962)** for planar flow and **Kliegel & Levine (1969)** for axisymmetric flow. Kliegel & Levine is Hall's expansion recast in toroidal coordinates, so it stays accurate for small throat radius of curvature. The inputs are the upstream and downstream throat wall radii of curvature, *R<sub>u</sub>/R\** and *R<sub>d</sub>/R\**. The series give the velocity components normalised by the critical speed *a\**, and these are converted to Mach number to start the MOC.
2. **Kernel.** Right-running characteristics are marched from the initial line along the circular-arc throat wall up to the attachment angle *θ<sub>B</sub>*.
3. **Contour.** The wall downstream of the kernel is found by mass balance along the final characteristic, according to the nozzle type and design parameter chosen.
4. **Streamlines.** Interior streamlines are interpolated through the grid at evenly spaced mass-flow fractions.

The full theory is in Rice (2003), JHU/APL RTDC-TPS-481.

> **Corrected in this edition:** the Hall and Kliegel-Levine series coefficients,
> integer-division truncation, and the velocity-ratio to Mach conversion. See
> [../CHANGES.md](../CHANGES.md). Results will differ from the NASA original and from the
> sample outputs in `outputs_M3.5Perf/`, which were produced by the original code.

---

## Running the program

Run `MOC_Grid_BDE.exe`, either the pre-built release or built from source as described below. A single dialog window opens.

**Output files are written to the current working folder**, normally the folder that contains the `.exe`. Put the exe in a dedicated run folder for each case.

### Inputs (dialog)

| Group | Field | Meaning | Default / limits |
|---|---|---|---|
| **Nozzle Geometry** | Axisymmetric / Planar | Flow geometry | Axisymmetric |
| **Nozzle Type** | Perfect | Ideal contour giving uniform, parallel exit flow at the design Mach | |
| | RAO (Optimum Thrust) | Rao-type thrust-optimised contour | default |
| | Cone/Wedge | Straight-wall divergence at **Half Angle (deg)** | 15°, range 0-45 |
| | Set End Point | Contour forced through **XEnd/R\*, REnd/R\*** | 4.308, 1.6 |
| **Design Parameter** | Exit Mach | Design exit Mach number | 4.0, range 1.1-100 |
| | Area Ratio | Exit-to-throat area ratio ε | 15 |
| | Xend/R\* | Nozzle length, normalised by throat radius | 5.03 |
| | Ptotal/Pexit | Total-to-exit pressure ratio | 13.69 |
| **Throat Geometry** | UpStream Radius/R\* | Upstream wall radius of curvature *R<sub>u</sub>/R\** | 1.0 |
| | DownStream Radius/R\* | Downstream wall radius of curvature *R<sub>d</sub>/R\** | 1.0 |
| **Flow Properties** | Pressure (psia), Temperature (R) | Chamber (stagnation) conditions, or throat static conditions if **Throat Conditions?** is ticked | 1000 psia, 530 R |
| | Mol. Wt, Gamma | Gas molecular weight and ratio of specific heats | 28.96, 1.4 |
| | P ambient (psia) | Ambient pressure, used for thrust | 0 |
| | Velocity (ft/s) | Throat velocity, enabled when throat conditions are given | 3022 |
| | Ideal Isp (lbf·s/lbm) | Reference Isp for efficiency output | 100 |
| **MOC Limiters** | THETAB Guess | Initial guess for wall attachment angle *θ<sub>B</sub>* (deg) | 25, range 0.1-40 |
| | # of RRC above BD | Number of right-running characteristics above the B-D line | 100, range 20-1000 |
| | DTHETAB Max (deg) | Maximum flow-angle step between characteristics | 0.5, range 0-10 |
| | Number of Starting Characteristics | Points on the initial data line | 101, range 2-301 |
| **Number of Output Streamlines** | Radial / Axial | Streamline resolution passed to STT2001 | 10 / 50 |
| **Print Option** | Normal / Full | **Full** writes every output file and enables the STT button | Normal |

Then:

- **Calculate MOC Grid** runs the solution. When it finishes, a plot of the nozzle contour is shown.
- **Run Streamline Tracing Tool** is available only with Print = Full. It copies `summary.out`, `MOC_Grid.plt` and `MOC_SL.plt` into `..\STT2001` and launches `STT2001.exe` there (see the note below).
- **File > Open / Save / Save As** loads or saves all dialog inputs as a `*.inp` text file.

### Outputs

| File | Contents |
|---|---|
| `summary.out` | Human-readable summary: inputs, throat conditions, θ<sub>B</sub>, exit Mach, area ratio, length, thrust/Isp |
| `Summary.plt` | Tecplot: summary data along the wall |
| `MOC_Grid.plt` | Tecplot: full characteristic grid with flow properties (Mach, p, T, ρ, θ) |
| `MOC_SL.plt` | Tecplot: interpolated streamlines (input to STT2001) |
| `wall.out`, `wall_i.out` | Wall contour coordinates (*x/R\*, r/R\**) and properties |
| `center.out`, `axis_i.out` | Centreline / axis distribution |
| `ThetaB.out` | Iteration history for the attachment angle |
| `LastKernel.out` | Last kernel characteristic |
| `TT'.out`, `TT'BF_Kernel.out`, `BFE_Kernel.out`, `UncroppedKernel.out` | Intermediate kernel data (Full print) |
| `rao.dat` | Rao contour data (RAO type) |

---

## Sample case: `outputs_M3.5Perf/`

A Mach 3.5 axisymmetric perfect nozzle, as distributed by NASA and computed with the **original, uncorrected** code. Use it to check that the program runs and to see what the output files look like, not as a numerical reference.

---

## Building from source

Requirements: Visual Studio 2022+ with *Desktop development with C++* and **C++ MFC for latest build tools (x86 & x64)**.

1. Open `MOC_Grid_BDE.sln`. Accept the retarget prompt if one appears.
2. Choose **Release | x64** and run **Build > Rebuild Solution**.
3. The output is `x64\Release\MOC_Grid_BDE_solution.exe`, a standalone exe with MFC and the CRT linked statically. You may rename it to `MOC_Grid_BDE.exe`.

The project includes an OLE-automation type library (`MOC_Grid.odl`). MIDL regenerates `MOC_Grid_h.h` and `MOC_Grid_i.c` during the build.

### Note on the STT button

The button writes and runs a batch file `$$.bat`. That file assumes `STT2001.exe` sits in a sibling folder named `STT2001` (`..\STT2001`, set in `MOC_GridDlg.cpp`, `OnSTTRunBUTTON`). If you use the exes standalone, either keep that folder layout:

```
work\
|-- MOC_Grid_BDE\MOC_Grid_BDE.exe
`-- STT2001\STT2001.exe
```

or skip the button. Instead, copy `summary.out`, `MOC_Grid.plt` and `MOC_SL.plt` to STT2001's folder yourself.

---

## Source files

| File | Role |
|---|---|
| `MOC_GridCalc_BDE.cpp/.h` | Core MOC solver: throat solutions (`CalcHallLine`, `KLThroat`, `Sauer`), kernel, contour design, streamlines |
| `MOC_GridCalc_BDE_IO.cpp` | All file output |
| `MOC_GridDlg.cpp/.h` | Main dialog, input handling, `.inp` file I/O, STT launcher |
| `MOCPlotDialog.cpp/.h`, `Chart.cpp/.h` | Built-in contour plot |
| `MOC_Grid.cpp/.h`, `DlgProxy.cpp/.h`, `MOC_Grid.odl` | MFC application and automation boilerplate |
| `engineering_constants.hpp`, `dummyStruct.h` | Constants and helper struct |

## References

- Rice, T. (2003). *2D and 3D Method of Characteristic Tools for Complex Nozzle Development*. JHU/APL RTDC-TPS-481. [NTRS 20030067852](https://ntrs.nasa.gov/citations/20030067852)
- Hall, I. M. (1962). Transonic flow in two-dimensional and axially-symmetric nozzles. *Q. J. Mech. Appl. Math.* 15(4). [doi:10.1093/qjmam/15.4.487](https://doi.org/10.1093/qjmam/15.4.487)
- Kliegel, J. R. & Levine, J. N. (1969). Transonic flow in small throat radius of curvature nozzles. *AIAA J.* 7(7). [doi:10.2514/3.5355](https://doi.org/10.2514/3.5355)

License: NASA Open Source Agreement v1.3. See [../license.txt](../license.txt) and [../NOTICE.md](../NOTICE.md).
