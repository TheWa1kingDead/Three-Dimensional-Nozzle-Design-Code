# Three-Dimensional Nozzle Design Code (corrected, standalone-build edition)

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23269365.svg)](https://doi.org/10.5281/zenodo.23269365)

A maintained edition of NASA's **Three-Dimensional Nozzle Design Code** (NASA software designation LEW-20180, original author **Tharen Rice**, JHU/APL). It fixes errors in the transonic throat solution and builds as standalone Windows executables that run on any 64-bit PC without Visual Studio installed.

> **This is a modified version of NASA Original Software.** It is distributed under the
> [NASA Open Source Agreement v1.3](license.txt). Modifications © 2026 MRK Reddy, see
> [NOTICE.md](NOTICE.md) and [CHANGES.md](CHANGES.md). This project is not endorsed by NASA,
> JHU/APL, or the original author.

Original repository: <https://github.com/nasa/Three-Dimensional-Nozzle-Design-Code>

---

## The three programs

The package contains three programs that are designed to be used together. Each has its own folder, source code, sample case and detailed README.

| Program | Folder | What it does |
|---|---|---|
| **MOC_Grid_BDE** | [`MOC_Grid_BDE/`](MOC_Grid_BDE/README.md) | 2D method of characteristics (planar or axisymmetric). Designs a supersonic nozzle contour (perfect, Rao/optimum-thrust, cone/wedge, or fixed end-point) and writes the full characteristic grid and streamlines. |
| **STT2001** | [`STT2001/`](STT2001/README.md) | Streamline Tracing Tool. Traces streamlines through a solution grid (usually from MOC_Grid_BDE), trims them to a chosen inlet/exit shape and integrates thrust, Isp, friction and area properties. |
| **3D_MOC** (3DCalc) | [`3D_MOC/`](3D_MOC/README.md) | 3D method of characteristics. Computes the 3D flowfield inside a given nozzle wall, which may be non-axisymmetric (e.g. a modified MOC_Grid_BDE contour or a streamline-traced shape from STT2001). |

### Typical workflow

```
MOC_Grid_BDE  -->  MOC_Grid.plt, MOC_SL.plt, summary.out
     |
     |  (Run STT button, or copy the files by hand)
     v
STT2001       -->  traced / trimmed 3D nozzle surface (*.xyz, *.dat, *.plt)

wall geometry (*.geo)
     |
     v
3D_MOC        -->  3D flowfield, wall, streamlines (*.plt)
```

All plot files (`*.plt`) are written in **Tecplot ASCII** format. They can also be read by ParaView, by MATLAB or Python with a short parser, or by any text editor.

---

## What's different from the NASA original

Summary only. See [CHANGES.md](CHANGES.md) for the full, dated list with line-level details.

1. **Corrected transonic throat solution in MOC_Grid_BDE.** The series solutions of Hall (1962) and Kliegel & Levine (1969), which set up the initial data line at the throat, have been checked term by term against the published papers. Mistyped coefficients, sign errors, missing powers of *y*, C++ integer-division truncation (`5/8 == 0`, `1/6 == 0`) and a missing velocity-ratio to Mach conversion have been corrected. These change the computed starting line and therefore every downstream result.
2. **Loop guard fix in MOC_Grid_BDE.** An operator-precedence bug meant the 20-iteration limit applied to only one of two failure conditions.
3. **Builds out of the box in Visual Studio 2022+.** Includes solution and project files for all three programs and x64 support. Release builds link MFC and the C runtime statically, so the `.exe` runs on PCs without Visual Studio or the VC++ redistributable.

The 3D_MOC build fixes (`nr.h` / `nrtypes_nr.h`, `.vcxproj`) were contributed by **Bleialf** (2022) in
<https://github.com/Bleialf/Three-Dimensional-Nozzle-Design-Code>. That history is preserved in this repository.

---

## Quick start

### Option A: use the pre-built executables

Download `MOC_Grid_BDE.exe`, `STT2001.exe` and `3DCalc.exe` from the [Releases](../../releases) page. No installation is needed. Put each `.exe` in a working folder and run it; output files are written to the folder the program is started from.

### Option B: build from source

Requirements:

- Windows 10/11, 64-bit
- **Visual Studio 2022 or newer** with the *Desktop development with C++* workload
- The individual component **"C++ MFC for latest build tools (x86 & x64)"**, added via Visual Studio Installer > Modify > Individual components

Steps, for each program:

1. Open the solution file: `MOC_Grid_BDE/MOC_Grid_BDE.sln`, `STT2001/STT2001_solution.sln` or `3D_MOC/3DCalc.sln`.
2. If Visual Studio offers to **retarget** the project (e.g. VS 2026 / toolset v145), accept it.
3. Set the toolbar configuration to **Release | x64**.
4. Run **Build > Rebuild Solution**.
5. The executable is written to `x64\Release\` inside that program's folder.

Debug builds depend on Visual Studio's debug DLLs and only run on the machine that built them.

---

## Repository layout

```
|-- MOC_Grid_BDE/        2D MOC grid generator (source, VS project, sample case)
|   `-- outputs_M3.5Perf/    sample: Mach 3.5 perfect nozzle
|-- STT2001/             Streamline Tracing Tool (source, VS project, sample case)
|   `-- outputs_M3.5Perf/    sample: tracing the Mach 3.5 nozzle
|-- 3D_MOC/              3D MOC flowfield solver (source, VS project, sample cases)
|   |-- outputs_M4Perfect/   sample: Mach 4 perfect nozzle
|   |-- outputs_M4RAO/       sample: Mach 4 Rao nozzle
|   `-- outputs_cone10/      sample: 10° cone
|-- CITATION.cff         citation metadata (GitHub "Cite this repository")
|-- CHANGES.md           dated change log required by NOSA 1.3 §3C
|-- NOTICE.md            copyright, attribution and modification notice
`-- license.txt          NASA Open Source Agreement v1.3 (unchanged)
```

---

## How to cite

If you use this software, please cite **both** this edition and the original NASA/JHU-APL report. This edition is archived on Zenodo: cite the concept DOI [10.5281/zenodo.23269365](https://doi.org/10.5281/zenodo.23269365) for the software in general, or the version DOI [10.5281/zenodo.23269366](https://doi.org/10.5281/zenodo.23269366) to refer to v1.0.0 exactly. GitHub's **"Cite this repository"** button (right sidebar) generates APA/BibTeX from [`CITATION.cff`](CITATION.cff).

```bibtex
@software{reddy_nozzle_design_code_2026,
  author  = {Reddy, MRK},
  title   = {Three-Dimensional Nozzle Design Code: corrected transonic throat
             solution and standalone Windows builds (modified version of
             NASA LEW-20180)},
  year    = {2026},
  version = {1.0.0},
  doi     = {10.5281/zenodo.23269365},
  url     = {https://doi.org/10.5281/zenodo.23269365},
  note    = {Modified from NASA Original Software by T. Rice (JHU/APL);
             NASA Open Source Agreement v1.3}
}

@techreport{rice_2003_moc,
  author      = {Rice, Tharen},
  title       = {{2D} and {3D} Method of Characteristic Tools for Complex Nozzle Development},
  institution = {The Johns Hopkins University Applied Physics Laboratory},
  number      = {RTDC-TPS-481},
  year        = {2003},
  url         = {https://ntrs.nasa.gov/citations/20030067852}
}
```

## References

- Rice, T., *2D and 3D Method of Characteristic Tools for Complex Nozzle Development*, JHU/APL Report RTDC-TPS-481, 2003. [NTRS 20030067852](https://ntrs.nasa.gov/citations/20030067852). This is the theory and user manual for all three programs.
- Hall, I. M., "Transonic flow in two-dimensional and axially-symmetric nozzles," *Quarterly Journal of Mechanics and Applied Mathematics* 15(4), 1962. [doi:10.1093/qjmam/15.4.487](https://doi.org/10.1093/qjmam/15.4.487)
- Kliegel, J. R. and Levine, J. N., "Transonic flow in small throat radius of curvature nozzles," *AIAA Journal* 7(7), 1969. [doi:10.2514/3.5355](https://doi.org/10.2514/3.5355)

## License

Distributed under the **NASA Open Source Agreement v1.3** ([license.txt](license.txt)). Under that agreement:

- Any redistribution must include `license.txt`.
- If you distribute executables, you must make the source code available.
- Your own modifications must be identified in a change log.

> Copyright 2020 United States Government as represented by the Administrator of the National
> Aeronautics and Space Administration. No copyright is claimed in the United States under
> Title 17, U.S. Code. All Other Rights Reserved.

NASA requests that users of this software register at <https://github.com/nasa/Three-Dimensional-Nozzle-Design-Code> (NOSA §3F).

## Disclaimer

Provided "AS IS" without warranty of any kind (NOSA §4). Results from these tools should be independently verified before use in hardware design. Export of technical data may be subject to U.S. export control regulations (NOSA §3J).

## Contact

Maintainer of this edition: **MRK Reddy** · ORCID [0009-0006-8420-6672](https://orcid.org/0009-0006-8420-6672) · GitHub [@TheWa1kingDead](https://github.com/TheWa1kingDead)

Please report bugs in this edition via [Issues](../../issues), not to NASA.
