## CPU/GPU_Benchmark
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

---
## Hole etching
### 2D

| Method | Time  | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|-------|-------------|-------------------|----------------|
| CPU    | 35.1s | 282         | 0.12447           | –              |
| Disks  | 2.3s  | 285         | 0.00807           | 15.26×         |
| Lvl    | 3.5s  | 325         | 0.01077           | 10.03×         |
| Lines  | 2.6s  | 290         | 0.00897           | 13.50×         |

### 3D

| Method | Time   | Trace Calls | Time / Trace Call | Speedup vs CPU     |
|--------|--------|-------------|-------------------|--------------------|
| CPU    | 675s   | 279         | 2.41935           | –                  |
| Disks  | 33.9s  | 267         | 0.12697           | 19.91×             |
| Lvl    | 11s    | 161         | 0.06800           | 61.36× *(bad surface)* |
| Trig   | 57.5s  | 355         | 0.16197           | 11.74×             |

---
## Bosch Process
### 2D

| Method | Time  | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|-------|-------------|-------------------|----------------|
| CPU    | 96.8s | 502         | 0.19283           | –              |
| Disks  | 3.5s  | 504         | 0.00694           | 27.66×         |
| Lvl    | 4.6s  | 474         | 0.00970           | 21.04×         |
| Lines  | 4.0s  | 510         | 0.00784           | 24.20×         |

### 3D

| Method | Time   | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|--------|-------------|-------------------|----------------|
| CPU    | 270min*| 523*        | 30.98s             | –              |
| Disks  | 372s   | 512         | 0.72656s           | 43.55×         |
| Lvl    | 592s   | 466         | 1.27039s           | 27.36×         |
| Trig   | 318s   | 531         | 0.59887s           | 50.94×         |

*estimated based on 7/10 cycles.

---
## DRAM Wiggling
### 3D (Second Iteration)

| Method | Time  | Trace Calls | Time / Trace Call | Speedup vs CPU |
|--------|-------|-------------|-------------------|----------------|
| CPU    | 41min | 49          | 50.61225          | –              |
| Disks  | 52.5s | 48          | 1.09375           | 47.24x         |
| Lvl    | 87s   | 60          | 1.45000           | 28.51x         |
| Trig   | 39.4s | 58          | 0.67931           | 62.94x         |
