# Summary
- GPU Triangles can trace ~3x more Rays/Second than GPU Disks but uses ~2x the amount of surface elements for a mesh.
- GPU Disks produces identical final surfaces (up to difference of using different RNGs) as the CPU (Disk) version while achieving speedups of 8x-30x.
- GPU Levelset is slower than Disks, mainly because of very poor performance when reading MaterialIDs in current implementation. Also final surfaces are not really usable:
  - holeEtching: rough surface in 2D, completely fails in 3D (not etching deep enough).
  - boschProcess: final surface is quite far off from CPU reference, much wider trench.
  - DRAM Wiggling: surprisingly matches the reference solution quite well.
- GPU Lines shows similar performance as GPU Disks (bit slower probably because of kdTree build), while final surfaces look similar to 3D Triangle counterparts with higher etch rate due to less shadowing on the edges. Sidewalls can also be a bit rough.

## General
- 4 different mesh types were analyzed:
  - Disk Mesh (2D/3D, CPU version as reference) - viennals::ToDiskMesh
  - LevelSet Mesh (2D/3D) - Directly using viennals/ps::Domain
  - Line Mesh (2D) - viennals::ToSurfaceMesh
  - Triangle Mesh (3D, already implemented) - viennals::ToSurfaceMesh
- The following ViennaPS examples were benchmarked in 2D and 3D:
  - CPU/GPU_Benchmark (ViennaPS/gpu/benchmarks/)
  - holeEtching
  - boschProcess
  - DRAMWiggling
- All examples use the same process specific parameters found in data/(exampleName)/_config.txt
- The Loglevel is set to ERROR with: Logger::setLogLevel(LogLevel::ERROR);
- One Timer around the whole process.
- NumericType = double, although Embree and OptiX use float internally.
- The final surfaces in .vtp format can be found in psResults/
- LevelSet benchmarks use an older ViennaPS version.
## CPU/GPU_Benchmark
- This benchmark compares the raw Ray Tracing performance in terms of Rays/Second with minimal influence from process specific complexity.
- MAKE_GEO Hole from BenchmarkGeometry.hpp with GRID_DELTA=0.1.
- sticking = 0.1 with diffuse reflections.
- Median runtime for 10 advection steps.
- Number of rays: 3D 140.000.000 / 2D 3.600.000 (initial geometry: 3D 47003 disks * 3000 rays/disk, 2D 601 disks * 6000 rays/disk).
### 2D (10 iterations 3.6e6 rays / iteration)

| Method | Time | Million Rays / second | Speedup vs CPU |
|--------|------:|----------------------:|----------------:|
| CPU    | 7.566954s | 4.757528 | – |
| Disks  | 0.296120s | 121.589206 | 25.55× |
| Lvl    | 0.431793s | 83.373323 | 17.53× |
| Lines  | 0.281382s | 128.022267 | 26.89× |

### 3D (10 iterations 1.4e8 rays / iteration)

| Method | Time | Million Rays / second | Speedup vs CPU |
|--------|------:|----------------------:|----------------:|
| CPU    | 419.486749s | 3.337412 | – |
| Disks  | 18.116835s | 77.283497 | 23.15× |
| Lvl    | 17.393257s | 80.479983 | 24.11× |
| Trig   | 6.138209s | 228.061039 | 68.33× |

- In 2D the Disk and Line Mesh perform very similar which was expected.
- The LvlSet Method performs considerably worse in 2D, while in 3D it is faster than the disk method. Also the 3D final surface has some bumps which are likely from errors in area calculation and normalization. One edge of the surface has a strange curvature.
- The Ray Tracing on Triangles is hardware accelerated by NVIDIA's RT cores and therefore ~3x as fast. However in the following benchmark examples the ray number is relative to the number of surface elements. In general the Triangle mesh uses around twice as many elements as the other methods.

---
## Hole etching
### 2D

| Method | Time | Iterations | Time / Iteration | Speedup vs CPU |
|--------|------:|-----------:|-----------------:|---------------:|
| CPU    | 11.918500s | 159 | 0.074959 | – |
| Disks  | 1.468070s  | 159 | 0.009233 | 8.12× |
| Lvl    | 3.460820s  | 162 | 0.021363 | 3.44× |
| Lines  | 1.671840s  | 165 | 0.010132 | 7.13× |

### 3D

| Method | Time | Iterations | Time / Iteration | Speedup vs CPU |
|--------|------:|-----------:|-----------------:|---------------:|
| CPU    | 179.869000s | 150 | 1.199127 | – |
| Disks  | 19.827200s  | 149 | 0.133068 | 9.07× |
| Lvl    | 11.239300s  |  81 | 0.138757 | 16.00× *(bad surface)* |
| Trig   | 20.331500s  | 161 | 0.126283 | 8.85× |

- In 2D again Disks and Lines similar while the LevelSet is slower, also the LevelSet surface is very rough. The final surface from the Lines method has some unexpected dents.
- In 3D the LevelSet surface gets really bad. Trig and Disk very similar but Trig etches a bit further down due to less shadowing.

---
## Bosch Process
### 2D

| Method | Time | Iterations | Time / Iteration | Speedup vs CPU |
|--------|------:|-----------:|-----------------:|---------------:|
| CPU    | 34.7621s | 502 | 0.069255 | – |
| Disks  | 3.35755s | 504 | 0.006662 | 10.35× |
| Lvl    | 4.63136s | 474 | 0.009772 | 7.09× *(bad surface)* |
| Lines  | 3.96902s | 509 | 0.007796 | 8.89× |

### 3D

| Method | Time | Iterations | Time / Iteration | Speedup vs CPU |
|--------|------:|-----------:|-----------------:|---------------:|
| CPU    | 8609.11s | 507 | 16.982254 | – |
| Disks  | 351.46s | 512 | 0.686445 | 24.50× |
| Lvl    | 598.056s | 464 | 1.288060 | 14.39× *(bad surface)* |
| Trig   | 337.08s | 527 | 0.639504 | 25.54× |

- Again the LeveSet method produces very different final surfaces than the CPU reference.
- This example highlights the similar behavior from Lines and Triangles where both methods etch slightly more into the sidewalls compared to CPU/GPU Disks.
- Also the effect of Rays/Surface Element gets clear as the Triangles are not 3x faster than the Disks as the first benchmark might have suggested.

---
## DRAM Wiggling
### 3D (Two Process Steps)

| Method | Time | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|------:|------------:|-----------------:|---------------:|
| CPU    | 1354.94s | 24 | 56.455833 | – |
| Disks  | 45.2345s | 24 | 1.884771 | 29.96× |
| Lvl    | 157.507s | 43 | 3.662953 | 8.60× |
| Trig   | 45.6137s | 29 | 1.573576 | 35.77× |

- Surprisingly the surface of the LevelSet method matches the reference solution quite well.