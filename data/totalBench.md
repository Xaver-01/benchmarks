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
