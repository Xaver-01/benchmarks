# Summary
- GPU Triangles can trace ~2.5x more Rays/Second than GPU Disks but uses ~2x the amount of surface elements for a mesh.
- GPU Disks produces identical final surfaces (up to difference of using different RNGs) as the CPU (Disk) version while achieving speedups of 15x-47x.
- GPU Levelset is a bit slower than Disks, mainly because of very poor performance when reading MaterialIDs in current implementation. Also final surfaces are not really usable:
  - holeEtching: very rough surface in 2D, completely fails in 3D.
  - boschProcess: final surface is quite far off from CPU reference.
  - DRAM Wiggling: surprisingly matches the reference solution quite well.
- GPU Lines shows similar performance as GPU Disks, while final surfaces look similar to 3D Triangle counterparts with higher etch rate due to less shadowing on the edges.

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
- The Loglevel is set to Timing with: Logger::setLogLevel(LogLevel::TIMING);
- The final surfaces in .vtp format can be found in psResults/
## CPU/GPU_Benchmark
- This benchmark compares the raw Ray Tracing performance in terms of Rays/Second with minimal influence from process specific complexity.
- MAKE_GEO Hole from BenchmarkGeometry.hpp with GRID_DELTA=0.1.
- sticking = 0.1 with diffuse reflections.
- Median runtime over 10 advection steps.
- Number of rays: 140.000.000 (initial geometry: 47003 disks * 3000 rays/disk).
### 2D:
| Method | Time   | Million Rays/Second |  Speedup vs CPU |
|--------|--------|---------------------|-----------------|
| CPU    | 63s    |   2.2               | –               |
| Disks  | 990ms  |   141.4               | 63.63×          |
| Lvl    | 1475ms |   94.9                | 42.71x          |
| Lines  | 903ms  |   155.0               | 69.77×          |
### 3D:
| Method | Time   | Million Rays/Second |  Speedup vs CPU |
|--------|--------|---------------------|-----------------|
| CPU    | 72s    |   1.9               | –               |
| Disks  | 1874ms |   74.7               | 38.42×          |
| Lvl    | 1538ms |   91.0                | 46.81x          |
| Trig   | 746ms  |   187.7               | 96.51×          |

- In 2D the Disk and Line Mesh perform very similar which was expected. The current disk implementation sacrifices 2-3% of performance for more readable and clearer code, which should bring these two even closer together.
- The LvlSet Method performs considerably worse in 2D, while in 3D it is faster than the disk method. Also the 3D final surface has some bumps which are likely from errors in area calculation and normalization.
- The Ray Tracing on Triangles is hardware accelerated by NVIDIA's RT cores and therefore more than twice as fast. However in the following benchmark examples the ray number is relative to the number of surface elements. In general the Triangle mesh uses around twice as many elements as the other methods.

---
## Hole etching
### 2D

| Method | Time  | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|-------|-------------|-------------------|----------------|
| CPU    | 35.1s | 282         | 0.12447s           | –              |
| Disks  | 2.3s  | 285         | 0.00807s           | 15.26×         |
| Lvl    | 3.5s  | 325         | 0.01077s           | 10.03×         |
| Lines  | 2.6s  | 290         | 0.00897s           | 13.50×         |

### 3D

| Method | Time   | Trace Calls | Time / Trace Call | Speedup vs CPU     |
|--------|--------|-------------|-------------------|--------------------|
| CPU    | 675s   | 279         | 2.41935s           | –                  |
| Disks  | 33.9s  | 267         | 0.12697s           | 19.91×             |
| Lvl    | 11s    | 161         | 0.06800s           | 61.36× *(bad surface)* |
| Trig   | 57.5s  | 355         | 0.16197s           | 11.74×             |

- In 2D all methods perform similar, while the LevelSet surface is very rough. Also the final surface from the Lines method has some unexpected dents.
- In 3D the LevelSet surface gets really bad, while the Triangles method etches a lot further down (notice the number of Trace Calls) which makes the Disk method wrongly look like it is a lot faster.

---
## Bosch Process
### 2D

| Method | Time  | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|-------|-------------|-------------------|----------------|
| CPU    | 96.8s | 502         | 0.19283s           | –              |
| Disks  | 3.5s  | 504         | 0.00694s           | 27.66×         |
| Lvl    | 4.6s  | 474         | 0.00970s           | 21.04×         |
| Lines  | 4.0s  | 510         | 0.00784s           | 24.20×         |

### 3D

| Method | Time   | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|--------|-------------|-------------------|----------------|
| CPU    | 305min | 507         | 36.13             | –              |
| Disks  | 372s   | 512         | 0.72656           | 49.25×         |
| Lvl    | 592s   | 466         | 1.27039           | 30.95×         |
| Trig   | 318s   | 531         | 0.59887           | 57.61×         |

- Again the LeveSet method produces very different final surfaces than the CPU reference.
- This example highlights the similar behavior from Lines and Triangles where both methods etch slightly more into the sidewalls compared to CPU/GPU Disks.
- Also the effect of Rays/Surface Element gets clear as the Triangles are not 2.5x faster than the Disks as the first benchmark might have suggested.

---
## DRAM Wiggling
### 3D (Second Iteration)

| Method | Time  | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|-------|-------------|-------------------|----------------|
| CPU    | 41min | 49          | 50.61225s          | –              |
| Disks  | 52.5s | 48          | 1.09375s           | 47.24x         |
| Lvl    | 87s   | 60          | 1.45000s           | 28.51x         |
| Trig   | 39.4s | 58          | 0.67931s           | 62.94x         |

- Surprisingly the surface of the LevelSet method matches the reference solution quite well.