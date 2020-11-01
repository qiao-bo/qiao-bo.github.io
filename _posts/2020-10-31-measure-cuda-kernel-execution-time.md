---
title: "Measure CUDA kernel execution time"
classes: wide
excerpt: "How to measure time in Hipacc."
tags: 
  - CUDA 
  - Profiling 
---

More costly kernels:

Bilateral filter
1 output of nvprof: 19.3862s for 301 calls, which is 64.44ms per kernel execution.
2 using 300 + 1 runtime event measurement: 64.86ms per kernel execution.
3 single run: 68.7332ms

For cheaper kernels:
Color Conversion:
1 nvprof: 202.77ms for 301 calls, 0.673ms
2 event: 0.620ms
3 single run: 0.984ms

Windowsing:
1 nvprof: 727.03us
2 300+1:  680.64ms
3 single run: 1.03ms

cuda-samples

matrix-mul
300 iterations + warm-up: 0.126ms
nvprof time 0.124ms

concurrent-kernels
nvprof: 11.3 ms
event: 11.0ms

some throughts:
GPU is running at basically max clock rate or not?
warm up is one of those benchmarking best practices
the cudaEvent method, according to my testing, 
is significantly closer to the profiler measurement if you do the warm up run 

The profilers do change a few behaviors:

disable some power management when capturing PM counters
increase the GPU timer frequency from 1 MHz to 31.25 MHz
measure kernel execution time more precisely than is possible with CUDA events
increase CPU overhead
flush work to GPU faster (using a Windows, not Linux difference)

It’s generally recommended to use the profiler reported times. cudaEvent can give unexpected results in multi-streamed settings and/or on windows WDDM, or on linux if the GPU is driving a display.

CUDA event timer is preferrable than cpu host timer. throughful observers have pointted it out.

references:
https://stackoverflow.com/questions/59567736/different-timing-indicated-from-two-kind-of-timers
https://forums.developer.nvidia.com/t/why-would-code-run-1-7x-faster-when-run-with-nvprof-than-without/56406/6
https://stackoverflow.com/questions/5828816/cuda-difference-between-cpu-timer-and-cuda-timer-event/5846331#5846331


