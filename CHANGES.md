# Change log

This file is the change log required by NOSA 1.3 §3C. It identifies each alteration to the
NASA Original Software, the date it was made and who made it. All alterations listed here are
Modifications derived from Original Software provided by NASA. Their originators consent
to that characterization.

Baseline: NASA release, commit `09ba559` of <https://github.com/nasa/Three-Dimensional-Nozzle-Design-Code>.

---

## [1.0.0] - 2026-10-10 - MRK Reddy

### Fixed: `MOC_Grid_BDE/MOC_GridCalc_BDE.cpp` (transonic throat solution)

The initial data line at the throat is computed from the series solutions of
Hall (1962, [doi:10.1093/qjmam/15.4.487](https://doi.org/10.1093/qjmam/15.4.487)) and
Kliegel & Levine (1969, [doi:10.2514/3.5355](https://doi.org/10.2514/3.5355)).
These were re-checked against the papers and corrected.

**`CalcHallLine()`**

- `z*(r*r - 5/8)` -> `z*(r*r - 5.0/8.0)`. In C++, `5/8` is integer division and evaluates to 0.
- `mach = q*sqrt((g+1)/2 - (g-1)/2*q*q)` -> uses `2.0` divisors to make the floating-point intent explicit.

**`KLThroat()`, axisymmetric branch (Kliegel & Levine, toroidal coordinates)**

- `u[2]`: `5/8` -> `5.0/8.0` (integer-division bug)
- `u[3]`: coefficient denominator `/34` -> `/384`
- `v[3]`: `(...)*y^5/1728 * (388G² + 1161G + 1181)*y²/576` ->
  `(...)*y^5/1728 - (388G² + 1161G + 1881)*y³/576`. This was a wrong operator, a wrong constant (1181 -> 1881) and a wrong power of *y*.

**`KLThroat()`, planar branch (Hall)**

- `u[1]`: `1/6` -> `1.0/6.0` (integer-division bug; the term evaluated to 0)
- `u[2]`: `(y+6)` -> `(G+6)`
- `u[3]`: `(782G² + 5523 + 2*G*2887)` -> `(782G² + 5523G + 22887)`
- `v[3]`: `* (194G² + 837G + 1665)` -> `- (194G² + 837G + 1665)` (sign/operator error)
- `v[3]`: `(26G² + 51G + 189)/144` -> `(26G² + 51G + 189)*y/144` (missing factor *y*)

**`KLThroat()`, both branches**

- The series give the velocity ratio *q = V/a\** (speed normalised by the critical speed of sound), not the Mach number. The original assigned `mach = Q`. It now converts correctly:
  `M = q / sqrt((γ+1)/2 - (γ-1)/2 · q²)`.

**`CalcContouredNozzle()`**

- `while (A == LOW || A == HIGH && i++ < 20)` -> `while ((A == LOW || A == HIGH) && i++ < 20)`.
  Because `&&` binds tighter than `||`, the 20-iteration limit was skipped for `SEC_FAIL_LOW`, so the loop could run without bound.

**Impact.** These corrections change the starting line and therefore every downstream result: the wall contour, the grid, the streamlines and the performance numbers. The sample outputs in `outputs_*` folders are from the **original** NASA code. They are kept for reference and were not regenerated with the corrected code.

### Changed: build system (all three programs)

- Added Visual Studio 2022 (toolset v143) solution and project files for `MOC_Grid_BDE` and `STT2001`, with Win32 and x64 configurations.
- Release configurations of all three programs link MFC and the C runtime **statically** (`UseOfMfc=Static`, `/MT`), so the executables run without Visual Studio or the VC++ redistributable.
- `3D_MOC/3DCalc.vcxproj`: added the missing MFC setting and compiler/linker options for Release|x64.
- `MOC_Grid_BDE`: regenerated MIDL output (`MOC_Grid_h.h`, `MOC_Grid_i.c`) for x64. Moved `MOC_Grid.rc2` into `res/`, which is where `MOC_Grid.rc` includes it from.
- `STT2001/STT2001.rc`: removed the `#include "res\STT2001.rc2"` line, because that file was never included in the NASA release.
- Added `.gitignore` for Visual Studio build output.

### Added: documentation

- `README.md` (top level), and a `README.md` in each program folder
- `NOTICE.md`, `CHANGES.md`, `CITATION.cff`

---

## 2022-04-22 - Bleialf (commit author "KJa")

From <https://github.com/Bleialf/Three-Dimensional-Nozzle-Design-Code>, commits `070eabe`, `bd9dc2f` and `52d8f09`:

- `3D_MOC`: added Visual Studio project files (`3DCalc.sln`, `.vcxproj`, `.filters`), `res/3D_MOC.rc`, and the sample inputs `cone10.geo` and `Initial Wall.plt` at the project root.
- `3D_MOC/nr.h`: commented out unused `arithcode`/`huffcode` types. Replaced the K&R prototypes with `namespace NR` declarations matching `nrtypes_nr.h`. Reformatted.
- `3D_MOC/3D_MOCGrid.hpp`: added `#include "nrtypes_nr.h"`.
- Added `.gitignore`.
