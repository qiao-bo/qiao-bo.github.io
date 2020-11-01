---
title: "Measure CUDA kernel execution time"
classes: wide
excerpt: "How to measure CUDA kernel execution time."
tags: 
  - CUDA 
  - Profiling 
---

Measuring the execution time can be tricky sometimes, especially for small CUDA kernels. In general, you can use CUDA events or Nvidia profilers.

## Using Profilers 
        
```
nvprof ./kernel
```

The execution time can be seen in the profiling result. [This post](https://developer.nvidia.com/blog/cuda-pro-tip-nvprof-your-handy-universal-gpu-profiler/) has a nice explanation, and an example output is as follows:

```
==9261== Profiling application: ./tHogbomCleanHemi
==9261== Profiling result:
Time(%)      Time     Calls       Avg       Min       Max  Name
 58.73%  737.97ms      1000  737.97us  424.77us  1.1405ms  subtractPSFLoop_kernel(float const *, int, float*, int, int, int, int, int, int, int, float, float)
 38.39%  482.31ms      1001  481.83us  475.74us  492.16us  findPeakLoop_kernel(MaxCandidate*, float const *, int)
  1.87%  23.450ms         2  11.725ms  11.721ms  11.728ms  [CUDA memcpy HtoD]
  1.01%  12.715ms      1002  12.689us  2.1760us  10.502ms  [CUDA memcpy DtoH]
```

The output also contains the memory communication time such as H2D and D2H copy.

## Using CUDA Events 

If you want more flexibility, CUDA events can be used to break down a part of your application. Typically, you have something like the follows:

```
float kernel_timing_ms = 0;
cudaEvent_t start, end;
cudaEventCreate(&start);
cudaEventCreate(&end);

cudaEventRecord(start, 0);
cudaError_t err = cudaLaunchKernel(kernel, grid, block, args, 0, 0);
checkErr(err, "cudaLaunchKernel(" + kernel_name + ")");             

cudaDeviceSynchronize();                                            
err = cudaGetLastError();                                           
checkErr(err, "cudaLaunchKernel(" + kernel_name + ")");             

cudaEventRecord(end, 0);                                            
cudaEventSynchronize(end);                                          
cudaEventElapsedTime(&kernel_timing_ms, start, end);              

cudaEventDestroy(start);                                            
cudaEventDestroy(end);
```

The time of the kernel execution is reported in `kernel_timing_ms` in ms.

## Problems with CUDA Events 

The above measurement using CUDA events is commonly seen in many projects. However, it is not accurate for kernels with short running times. The table below depicts the measurement results of some applications:

| Kernel  | Nvprof  | CUDA Events| 
|:--------|:-------|:-------|
| Color Conversion   | 0.673 ms | 0.984 ms  |
| Windowing filter   | 727.03 us| 1.03 ms   |
| Bilateral filter   | 64.44 ms | 68.73 ms|
{: rules="groups"}

The CUDA kernels are generated using [Hipacc](https://hipacc-lang.org/), the benchmark is performed using a Nvidia GTX680 with CUDA 11.0 under Ubuntu 18.04 LTS. As can be seen, the time logged with CUDA events are always higher than Nvprof reported.

One way to solve this problem is to (a) perform a warm-up run before the actual measurement. (b) using a loop to execute the kernel many times and then get the average. I adapted the above measurement using CUDA event with 300 runs per kernel. The final execution time is divided by the number of executions, namely 300. The new results are shown in the table below:

| Kernel  | Nvprof  | CUDA Events| CUDA Events with Iterations |
|:--------|-------:|-------:|--------:|
| Color Conversion   | 0.673 ms | 0.984 ms  | 0.620 ms |
| Windowing filter   | 727.03 us| 1.03 ms   | 680.64 ms|
| Bilateral filter   | 64.44 ms | 68.73 ms  | 64.86 ms |
| Matrix Mul         | 0.124 ms | -  | 0.126 ms |
| Concurrent Kernels | 11.3 ms | - | 11.0 ms |
{: rules="groups"}

I also measured two additional samples in the Nvidia [CUDA samples github repository](https://github.com/NVIDIA/cuda-samples), namely matrix-mul and concurrent-kernels. The results show that the updated measurement approach is much closer to the results obtained via Nvprof.

## Observations and Further References

Whether the device is running at its boost clock rate has impacts on the logged execution time, also whether the GPU is driving a display. A warm-up run is recommended as a best practice for benchmarking. In addition, the profiler is expected to measure the kernel execution time more precisely than CUDA events. But CUDA events are much better than doing host CPU timing, since the CPU host timer is less prefereed than the CUDA event timer on the device. On Windows and linux there is also a difference in profiler behavior.
 
Some useful links for further reading:

[https://stackoverflow.com/questions/59567736/different-timing-indicated-from-two-kind-of-timers](https://stackoverflow.com/questions/59567736/different-timing-indicated-from-two-kind-of-timers)

[https://forums.developer.nvidia.com/t/why-would-code-run-1-7x-faster-when-run-with-nvprof-than-without/56406/6](https://forums.developer.nvidia.com/t/why-would-code-run-1-7x-faster-when-run-with-nvprof-than-without/56406/6)

[https://stackoverflow.com/questions/5828816/cuda-difference-between-cpu-timer-and-cuda-timer-event/5846331#5846331](https://stackoverflow.com/questions/5828816/cuda-difference-between-cpu-timer-and-cuda-timer-event/5846331#5846331)


