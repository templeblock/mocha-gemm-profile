## Profiling GEMM on Android

# The Gap

Caffe relies heavily on GEMM. For example, ~90% of the computation in AlexNet is convolution. And Caffe implements convolution by turning it into GEMM through lowering. Therefore, it is important that GEMM runs fast.

To run GEMM on phone’s GPU, we need to implement it in OpenCL. After searching on the web, I found two popular OpenCL based BLAS implementations:

* [ViennaCL](http://viennacl.sourceforge.net). Supports OpenCL >= 1.1.
* [clBLAS] (clBLAS). Requires OpenCL >= 1.2.


Unfortunately, our Android phone only supports OpenCL 1.1. However, there are [a few benchmark results](http://viennacl.sourceforge.net/viennacl-benchmarks.html) showing ViennaCL with better performance than clBLAS.

![There is a big gap between GPU and CPU performance](images/throughput-cpu-gpu_opencl.png)
<center>**There is a big gap between GPU and CPU performance**</center>


The gap between GPU and its peak performance is even bigger. Using open-source clpeak benchmark tool, we get the following peak data:
Here, float**x** and double**x** are the SIMD instruction used. **x** stands for x numbers at a time.

# Better GEMM Kernels


![](images/register-constraints.png)

where I is the number of work items in parallel, and R is the number of registers used in kernel. **This implies it would be hard to implement kernels with complex caching and blocking.**


## Implementation #1: just vectorize
```
__kernel void gemm (

## Implementation #2: 2x2 Blocking 

```
__kernel void gemm (


# Benchmark

## Case 1: Square matrix a = b = c =M

![](images/square-time-arch.png)

![](images/square-throughput-arch.png)

![](images/square-energy-arch.png)


Note: 


```
Device perf:  30.6515 GFlops
```

![](images/square-throughput-kernel.png)

![](images/square-energy-kernel.png)

## Case 2: Fix a=c=1024, change b

![](images/change-b-time-arch.png)

![](images/change-b-energy-arch.png)

# Precision

![](images/precision-throughput.png)
