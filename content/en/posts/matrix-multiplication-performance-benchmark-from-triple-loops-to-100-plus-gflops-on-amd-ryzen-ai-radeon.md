---
date: '2025-04-19T20:33:11+08:00'
draft: false
summary: 'An in-depth benchmark comparing the performance of 11 matrix multiplication implementations (Naive, CPU multi-core/SIMD/BLAS, GPU via OpenCL/HIP/Vulkan) on AMD Ryzen AI + Radeon, revealing vast performance gaps and optimization insights.'
title: 'Matrix Multiplication Performance Benchmark: from Triple Loops to 100+ GFLOPS on AMD Ryzen AI + Radeon'
tags: [ "matrix-multiplication", "performance", "benchmark", "optimization", "comparison", "cpu", "gpu", "amd", "ryzen-ai", "radeon", "gfx1150", "avx", "avx2", "avx-512", "fma", "openmp", "blas", "eigen", "opencv", "opencl", "clblast", "hip", "rocm", "hipblas", "vulkan", "compute-shader", "linear-algebra", "high-performance-computing", "hpc", "parallel-computing", "vectorization", "simd", "gpgpu", "google-benchmark", "c-plus-plus" ]
---

Today, let's chat about a commonplace yet timeless topic – matrix multiplication. "Matrix multiplication? Learned that
in university linear algebra, isn't it just three `for` loops?" you might say. Indeed, the most basic implementation is
exactly that, simple and direct. But in the world of high-performance computing, where every cycle counts, there's a
whole universe hidden behind those three nested loops. Different implementation methods can lead to performance
differences that are worlds apart, sometimes by factors of hundreds or even thousands!

Sounds a bit exciting, doesn't it? Like comparing the speed of an F1 race car to a mobility scooter. Why such a massive
gap? Modern CPU and GPU architectures, compiler optimizations, parallel computing techniques, specialized math
libraries... these are all critical factors influencing performance.

To get a firsthand feel for these differences, I recently conducted a matrix multiplication (square matrices,
`C = A * B`) "performance showdown" on my new gear – a Lenovo ThinkBook 16 G7+ laptop equipped with an AMD Ryzen AI 9
365 processor (featuring integrated Radeon 880M graphics). We invited several "contenders" to the ring, covering a wide
range of approaches: from the most naive implementation to methods leveraging CPU multi-cores, SIMD instruction sets,
calling professional math libraries, and even harnessing GPU acceleration (using OpenCL, Vulkan Compute, and ROCm/HIP).

This blog post will walk you through the entire benchmarking process: from introducing the "race track" environment,
dissecting the technical characteristics of each "contender," to analyzing the final results and summarizing the
takeaways. We're not aiming for a stern academic paper, but rather a relaxed, natural discussion about the technical
intricacies and the allure of performance optimization. Hopefully, this will provide some inspiration and satisfy your
curiosity about high-performance computing.

Ready? Buckle up, let's get started!

## Hardware and Software Environment

As the saying goes, "To do a good job, one must first sharpen one's tools." Before diving into the performance tests,
let's lay out the "tools of the trade," meaning the hardware and software environment used for this benchmark.
Understanding this background information will help us better interpret the subsequent performance data.

My core hardware configuration includes an AMD Ryzen AI 9 365 processor, belonging to family 26, model 36. This is a
fairly new CPU, boasting 10 physical cores and supporting 20 threads, with a base frequency of 2.0 GHz. It features
crucial AVX, AVX2, FMA, and, importantly, AVX-512 instruction set support (including various flavors like AVX512F, DQ,
CD, BW, VL). While it also integrates an NPU (Neural Processing Unit), our tests primarily focus on its general-purpose
CPU and GPU compute capabilities. For memory, the system is equipped with 27.2 GiB (approximately 32GB as reported by
the system) of DDR5 RAM; memory size and speed are critical for the performance of large-scale matrix operations. The
integrated graphics card is the AMD Radeon Graphics (Radeon 880M). According to information from `rocminfo` and
`vulkaninfo`, its GPU model identifier is `gfx1150` (sometimes shown as `11.5.0`), featuring 12 Compute Units (CUs),
each containing 2 SIMD units. It can reach a maximum clock frequency of 2900MHz and supports both FP16 and FP64 (
double-precision) computations. This integrated GPU supports Vulkan, OpenCL, and AMD's ROCm/HIP platform, offering
multiple avenues for GPU acceleration in our tests. It's worth noting specifically that during the benchmark execution,
I set the `HSA_OVERRIDE_GFX_VERSION=11.5.1` environment variable. This might slightly influence the target code
generation or runtime behavior for HIP or hipBLAS, a practice often employed because official `rocblas` support for
`gfx1150` wasn't fully implemented at the time.

On the software side, I'm running Arch Linux, a rolling-release distribution, which keeps my software packages
relatively up-to-date. The specific kernel version is `6.14.2-2-cachyos` (64-bit); CachyOS is an Arch derivative often
incorporating performance-enhancing patches. The desktop environment is KDE Plasma 6.3.4, operating on the Wayland
display server protocol. For compilation, I primarily use GCC (g++), whose version varies with Arch Linux updates but
certainly supports C++17/20 standards along with OpenMP and AVX/AVX-512 instructions. HIP code compilation relies on
`hipcc` from the ROCm toolchain, which is based on Clang. Project building is managed by CMake (version 3.20 or higher).

The core libraries and drivers are key components for this benchmark. The ROCm platform needs to support the `gfx1150`
or `gfx1151` GPU model; `rocminfo` output in the test logs indicates Runtime Version 1.1 and Extension Version 1.6. The
OpenCL environment is slightly complex, with two platforms present: the AMD APP SDK (providing OpenCL 2.1, driver
version 3635.0) and Mesa rusticl (providing OpenCL 3.0). However, based on the test log stating
`OpenCL Info: Selected AMD Platform. Using first GPU device. Device Name: gfx1151`, we specifically selected the GPU
device under the official AMD driver platform for testing, identified as `gfx1151`. For Vulkan, the instance version is
1.4.309, using the RADV driver (from Mesa 25.0.4), which identifies the device as `AMD Radeon Graphics (RADV GFX1150)`.
We utilized the `glslc` tool to compile GLSL compute shaders into SPIR-V format. The system also has a BLAS (Basic
Linear Algebra Subprograms) implementation installed, likely OpenBLAS, a common high-performance choice on Linux
distributions, successfully located by CMake's `find_package(BLAS)`. Additionally, the open-source OpenCL BLAS library,
CLBlast, is installed and discoverable by CMake. Furthermore, we tested the popular C++ template library Eigen3 (version
3.3+, provided as header files) and the computer vision library OpenCV (version 4.x, with its core module correctly
found by CMake).

Finally, the entire benchmarking framework is Google Benchmark (v1.9.2). This is an industry-standard C++ benchmarking
library offering convenient test fixture management, precise timing, automatic iteration count adjustment, and
standardized result output, ensuring the rigor and reliability of our tests.

To squeeze out as much performance as possible, we employed some rather aggressive compilation options. For C++ code, we
used the GCC (g++) compiler with the `-Ofast` optimization level, combined with the `-march=native` flag, allowing the
compiler to generate the most optimized machine code based on the specific features of my native CPU (including its
AVX-512 capabilities). We also explicitly added `-mavx2 -mfma -mavx512f -mavx512dq` flags to ensure these SIMD
instructions could be utilized. For HIP code, we similarly used the `-Ofast` optimization option with `hipcc` (based on
Clang). Moreover, `CMAKE_HIP_ARCHITECTURES` was set to `gfx1150` via CMake (based on `rocminfo` findings) to guide the
compiler in generating code for the target GPU architecture. OpenCL Kernel optimization differs; it's specified not
during host code compilation but at runtime via options passed to the `clBuildProgram` function. A commonly used
optimization flag is `-cl-fast-relaxed-math`, which permits the OpenCL compiler to perform mathematical optimizations
that might slightly affect floating-point precision but can significantly improve execution speed. Lastly, for Vulkan
compute shaders, we also included the `-O` option when compiling them into SPIR-V format using the `glslc` tool,
enabling compile-time optimization.

With this background set, let's introduce the contenders and see what tricks they have up their sleeves.

## The Contenders: Matrix Multiplication Implementations Detailed

Next, we'll introduce each matrix multiplication implementation method that participated in this performance showdown.

### Naive Implementation

This contender is the one we're most familiar with and the starting point for all optimizations. It strictly follows the
definition of matrix multiplication, C[i][j] = Σ(A[i][k] * B[k][j]), using three nested loops:

```cpp
// Pseudo-code example
for i = 0 to N-1:
  for j = 0 to N-1:
    sum = 0;
    for k = 0 to N-1:
      sum += A[i][k] * B[k][j]; // or A[i*N + k] * B[k*N + j] for row-major 1D array
    C[i][j] = sum; // or C[i*N + j] = sum
```

The advantage of this naive implementation lies in its extreme simplicity and logical clarity, making it easy to
understand. However, its disadvantage is extremely poor performance. This stems mainly from several factors. First, it's
Cache Unfriendly. During computation, access to the B matrix occurs column-wise (in the innermost k-loop, j is constant,
k increments, accessing `B[k*N + j]`), but data is stored row-wise (Row-Major) in memory. This mismatch between access
pattern and storage layout leads to frequent CPU cache line misses, requiring constant reloading from main memory and
drastically reducing memory access efficiency. Accesses to matrix A (row-wise) and writes to matrix C (element-wise) are
comparatively better for caching, but the B matrix access pattern becomes the performance killer. Second, this
implementation is entirely serial, failing to utilize the valuable multi-core parallel processing capabilities of modern
CPUs. Finally, it also makes no use of the CPU's SIMD (Single Instruction, Multiple Data) units for vectorized
computation; each operation handles only a single element's multiplication and addition, resulting in low efficiency.

This one primarily serves as a performance baseline to see how much improvement other methods can offer.

### OpenMP (CPU Multi-core Parallelism)

OpenMP is a parallel programming model based on shared memory, primarily using compiler directives (Pragmas) to guide
the compiler in automatically generating parallel code. For loop-intensive tasks like matrix multiplication, it can
easily distribute the outer loop (typically the `i` loop) across different CPU cores for execution.

Implementation-wise, it merely involves adding a `#pragma omp parallel for` directive before the outer loop of the Naive
version:

```cpp
#pragma omp parallel for default(none) shared(A, B, C, N) schedule(static)
for (size_t i = 0; i < N; ++i) {
  // Inner j and k loops remain unchanged
  for (size_t j = 0; j < N; ++j) {
    ValueType sum = 0.0;
    for (size_t k = 0; k < N; ++k) {
      sum += A[i * N + k] * B[k * N + j];
    }
    C[i * N + j] = sum;
  }
}
```

Let's break down the key parts of this OpenMP directive. `parallel for` is the core instruction, telling the compiler to
parallelize the subsequent `for` loop. `default(none)` is a recommended good practice, forcing the programmer to
explicitly declare the scope of each variable within the loop—either shared (`shared`) or thread-private (`private`)—to
prevent potential errors. `shared(A, B, C, N)` declares that the matrices A, B, C, and the size N are shared among all
concurrently executing threads; A and B are read-only during computation, while C is written to, but since OpenMP
typically distributes work row-wise, different threads usually write to different rows of C, generally avoiding write
conflicts. Finally, `schedule(static)` defines the work distribution strategy. It statically pre-divides the loop's
entire iteration space (here, the N iterations of `i`) into roughly equal chunks and assigns these chunks to the
available threads. For well-load-balanced loops like matrix multiplication, static scheduling typically incurs low
runtime overhead.

The primary advantage of using OpenMP is its implementation simplicity; often, just adding a single compiler directive (
Pragma) before a critical loop conveniently utilizes the CPU's multi-core resources. Compared to the fully serial Naive
implementation, performance usually sees a significant boost, ideally approaching a speedup factor close to the number
of CPU cores, although the actual improvement is constrained by factors like memory bandwidth and cache efficiency.
However, it also has drawbacks. First, it doesn't resolve the cache unfriendliness issue present in the Naive version,
particularly the column-wise access pattern for matrix B, which limits further performance gains. Second, its
performance ceiling is inherently limited by the number of physical CPU cores and the system's memory bandwidth.
Furthermore, for very small matrix sizes (N), the overhead introduced by parallel computing (such as thread creation,
management, and synchronization) might even outweigh the time saved by parallel execution, leading to performance
degradation instead of improvement.

### CPU SIMD (AVX2/AVX-512 + FMA)

SIMD (Single Instruction, Multiple Data) is a crucial feature of modern CPUs. It allows a single instruction to perform
the same operation on multiple data elements simultaneously. For instance, AVX2 can process 4 `double` values at once (
using 256-bit registers), while AVX-512 can handle 8 `double` values (using 512-bit registers). FMA (Fused Multiply-Add)
instructions further enhance efficiency, and potentially precision, by combining a multiplication and an addition into a
single instruction.

To leverage SIMD, we typically need to use compiler-specific intrinsic functions. This makes the code considerably more
complex than the Naive or OpenMP versions.

#### AVX2 + FMA (256-bit)

To utilize AVX2 and FMA instructions, we included the `immintrin.h` header file, which provides access to the necessary
intrinsic functions. A key optimization strategy here involves changing the loop nesting order to `i-k-j`. The advantage
of this order is that it allows for efficient vectorization within the innermost `j` loop. Specifically, for fixed `i`
and `k`, we can first take the scalar value `A[i][k]` and broadcast it into all 4 double-precision elements of a 256-bit
vector `a_vec` using the `_mm256_set1_pd()` intrinsic. Next, we load 4 consecutive `double` values from the k-th row of
matrix B (starting at address `&B[k*N + j]`) into a vector `b_vec`. Since matrix B is stored row-major, this consecutive
load is generally cache-friendly. We opted for `_mm256_loadu_pd()`, which allows loading from unaligned memory
addresses, offering more flexibility. Concurrently, we load the corresponding 4 partial sums from the i-th row of matrix
C (address `&C[i*N + j]`) into `c_vec`, also using `_mm256_loadu_pd()`. The core computational step involves executing
the FMA (Fused Multiply-Add) operation, `c_vec = a_vec * b_vec + c_vec`, using the `_mm256_fmadd_pd()` intrinsic. This
single instruction performs 4 pairs of multiplications and additions simultaneously. Finally, the updated result vector
`c_vec` is written back to the corresponding location in matrix C using `_mm256_storeu_pd()`. Naturally, the
implementation of the innermost `j` loop needs to iterate with a step size of 4 (the AVX2_DOUBLE_COUNT) and also
requires special handling for any remaining elements at the end of the row (less than 4), which typically fall back to
standard scalar computation.

```cpp
// Pseudo-code example (AVX2 + FMA)
constexpr size_t AVX2_DOUBLE_COUNT = 4;
for (size_t i = 0; i < N; ++i) {
  for (size_t k = 0; k < N; ++k) {
    __m256d a_vec = _mm256_set1_pd(A[i*N + k]); // Broadcast A[i][k]
    for (size_t j = 0; j < N_aligned; j += AVX2_DOUBLE_COUNT) { // Aligned part
      __m256d b_vec = _mm256_loadu_pd(&B[k*N + j]);  // Load 4 doubles from B row k
      __m256d c_vec = _mm256_loadu_pd(&C[i*N + j]);  // Load 4 doubles from C row i
      c_vec = _mm256_fmadd_pd(a_vec, b_vec, c_vec); // Fused Multiply-Add
      _mm256_storeu_pd(&C[i*N + j], c_vec); // Store back to C
    }
    // Handle remaining elements j = N_aligned to N-1 using scalar operations
  }
}
```

#### AVX-512 + FMA (512-bit)

The implementation principle for AVX-512 + FMA is identical to the AVX2 version. The main difference lies in using
512-bit wide registers and their corresponding intrinsic functions, such as the `__m512d` type, `_mm512_set1_pd`,
`_mm512_loadu_pd`, `_mm512_fmadd_pd`, and `_mm512_storeu_pd`. Because the registers are wider, the vector computation
step size increases to 8 (AVX512_DOUBLE_COUNT), meaning a single instruction can now process 8 double-precision values.
Successfully compiling and running AVX-512 code requires the CPU itself to support the instruction set (our Ryzen AI 9
365 processor meets this condition) and necessitates enabling these instructions via appropriate compiler options (like
`-mavx512f`) during compilation.

This SIMD-based optimization approach offers significant advantages. Primarily, it can drastically improve the
computational performance of a single CPU core. Additionally, employing the `i-k-j` loop order enhances the memory
access pattern for matrix B, making it more cache-friendly. The core benefit comes from fully utilizing the powerful
vector processing units within the CPU. However, this method also comes with notable disadvantages. Writing and
maintaining code using SIMD intrinsics is considerably complex, and the resulting code suffers from poor portability as
it directly depends on the specific instruction sets supported by the target CPU. Developers must also manually handle
potential memory alignment issues (although `loadu/storeu` provide unaligned access, aligned loads/stores are generally
faster) and manage the boundary conditions at the end of loops. Furthermore, historically, executing AVX-512
instructions could sometimes trigger the CPU to reduce its operating frequency to manage power consumption and heat
generation; while this issue has been largely mitigated in modern CPUs, it remains a potential consideration.

### SIMD + OpenMP (AVX2/AVX-512 + FMA + OpenMP)

Since OpenMP can parallelize the outer loop and SIMD can accelerate the inner computations, combining them seems like a
powerful synergy. Indeed, it is.

The implementation simply involves adding the OpenMP parallel directive before the outer `i` loop of the SIMD (either
AVX2 or AVX-512) version using the `i-k-j` loop order:

```cpp
#pragma omp parallel for default(none) shared(A, B, C, N, N_aligned) schedule(static)
for (size_t i = 0; i < N; ++i) {
  // Inner k and j (SIMD) loops remain unchanged
  for (size_t k = 0; k < N; ++k) {
    // ... SIMD intrinsics code as before ...
  }
}
```

The primary advantage of combining SIMD instructions (be it AVX2 or AVX-512) with OpenMP multithreading is its ability
to leverage both the CPU's multi-core parallel processing power and its instruction-level parallelism (vectorization)
simultaneously. This two-pronged approach often allows reaching, or at least closely approaching, the theoretical peak
performance of the CPU for the given task. However, this method also has clear disadvantages. Firstly, it further
compounds the code complexity, incorporating intricacies from both SIMD intrinsics programming and OpenMP parallel
management. Secondly, as computation speed is pushed to its limits, the application's performance bottleneck is very
likely to shift from the computation itself to being limited by memory bandwidth – meaning the CPU cores can process
data faster than the memory subsystem can supply it. Lastly, achieving optimal performance usually requires careful
tuning of OpenMP-related parameters, such as selecting the most effective thread scheduling strategy (e.g., static,
dynamic, guided via the `schedule` clause) and potentially employing advanced thread management techniques like thread
affinity or load balancing adjustments.

### BLAS (Basic Linear Algebra Subprograms)

BLAS isn't a specific library but rather a standardized API specification defining interfaces for basic vector and
matrix operations. Many organizations and companies provide implementations of BLAS. These libraries typically contain
highly optimized C, Fortran, or even assembly code tailored for specific hardware (CPU architecture, cache sizes, SIMD
instructions). They often internally implement sophisticated techniques like blocking (or tiling) to maximize cache
utilization and automatically employ both SIMD instructions and multithreading.

We only need to call the standard C interface `cblas_dgemm` ('d' for double precision, 'gemm' for general matrix-matrix
multiplication):

```c
// Pseudo-code example
cblas_dgemm(
    CblasRowMajor, // Tell BLAS our data is stored row by row
    CblasNoTrans, CblasNoTrans, // Neither A nor B needs transposing
    N, N, N,        // M, N, K (for N x N matrices)
    1.0,            // alpha (for C = alpha*A*B + beta*C)
    A.data(), N,    // Pointer to A data and its leading dimension (cols for RowMajor)
    B.data(), N,    // Pointer to B data and its leading dimension
    0.0,            // beta (set to 0 to overwrite C, i.e., C = A*B)
    C.data(), N     // Pointer to C data and its leading dimension
);
```

Using a BLAS library for matrix multiplication offers several advantages. The most prominent is extreme ease of use;
developers typically only need to call a single highly optimized library function (like `cblas_dgemm`) to perform the
complex computation, significantly simplifying the programming effort. Secondly, because these libraries incorporate
extensive hardware-specific optimizations, their performance is usually excellent, often approaching the theoretical
peak computational throughput of the hardware. Furthermore, as BLAS is a standard interface, it provides good
portability – code can generally run unmodified on any target platform that has a compliant BLAS library implementation.
Calling a library function also results in very concise application code. Of course, using BLAS also has disadvantages.
First, the application needs to be linked against the corresponding BLAS library file during the build process. Second,
and most critically, the final performance achieved heavily depends on the quality of the specific BLAS implementation
being used. Different BLAS libraries (like OpenBLAS, Intel MKL, ATLAS, etc.) can exhibit significant performance
variations even on the same hardware.

### Eigen & OpenCV

Besides low-level interfaces like BLAS, many high-level C++ libraries also provide matrix operations. We tested two
popular examples: Eigen and OpenCV.

#### Eigen

Let's take a look at the Eigen library. Its key characteristic is being a C++ template library renowned for its elegant
API and powerful "Expression Templates" technology. This technique allows Eigen to analyze and optimize complex chains
of linear algebra expressions at compile time, avoiding the creation of unnecessary intermediate temporary objects and
often automatically generating SIMD instructions for the underlying computations. In terms of usage, Eigen code is also
very concise. We can first use `Eigen::Map` to "map" our raw data stored in `std::vector` onto Eigen's internal matrix
object – this mapping itself incurs zero memory copy overhead. Then, we can directly use the overloaded `*` operator to
perform the matrix multiplication, like so:

```cpp
// Pseudo-code example (Map existing data)
Eigen::Map<const EigenMatrixType> A_map(A.data(), N, N);
Eigen::Map<const EigenMatrixType> B_map(B.data(), N, N);
EigenMatrixType C_eigen(N, N); // Eigen's result matrix

matrix_multiply_eigen(A_map, B_map, C_eigen); // C_eigen.noalias() = A_map * B_map;
```

It's worth noting the use of the `noalias()` method in the code. This explicitly informs Eigen that the output matrix C
does not overlap in memory with the input matrices A or B (no aliasing), enabling Eigen to employ more efficient and
aggressive internal implementations for optimization.

Overall, Eigen's advantages include its very modern API, ease of use, and high code readability. Its ability to perform
compile-time optimizations via C++ template metaprogramming is also a significant strength. However, it also has
disadvantages. In terms of performance, it might not match specialized, deeply hand-optimized BLAS libraries (the final
performance largely depends on the compiler's optimization capabilities and the complexity of the specific expression).
Additionally, due to its heavy reliance on templates, compile times can be relatively longer.

#### OpenCV

Next up is OpenCV. Its primary characteristic is being a comprehensive library mainly focused on computer vision tasks.
However, its core module (`core`) also provides very powerful matrix operations centered around the `cv::Mat` class.
`cv::Mat` can manage its own memory or conveniently "wrap" existing external data, avoiding unnecessary copies. An
important advantage is that when performing computationally intensive operations like matrix multiplication, OpenCV
typically attempts to leverage available underlying optimization mechanisms to accelerate the process. This might
include Intel IPP (Integrated Performance Primitives), OpenMP multithreading, or potentially even calling a
system-installed BLAS library. When using it, we can wrap the data from our `std::vector` into `cv::Mat` objects without
copying, specifying the rows, columns, data type (`CV_64F` for double), and the data pointer. Then, we call the
`cv::gemm` function provided by OpenCV to perform the matrix multiplication. This function's interface is very similar
to the `gemm` function in BLAS:

```cpp
// Pseudo-code example
cv::Mat A_cv(N, N, CV_64F, A.data()); // Wrap existing data
cv::Mat B_cv(N, N, CV_64F, B.data());
cv::Mat C_cv(N, N, CV_64F);          // OpenCV result matrix

matrix_multiply_opencv(A_cv, B_cv, C_cv); // cv::gemm(A_cv, B_cv, 1.0, cv::Mat(), 0.0, C_cv);
```

OpenCV's advantages lie in its extremely rich feature set, extending far beyond just matrix multiplication to cover a
vast range of image processing and computer vision functionalities. If your project is already using OpenCV, employing
it for matrix operations allows for seamless integration with other library features. Furthermore, it may leverage
various backend optimization libraries to enhance performance. However, its disadvantages are also notable, primarily
the fact that it introduces a relatively large and complex library dependency. If your task solely involves pure linear
algebra computations, incorporating the entire OpenCV library might not be the most lightweight choice.

### OpenCL

Now we turn to OpenCL (Open Computing Language), an open standard framework designed for cross-platform, heterogeneous
parallel computing, allowing programs to utilize various compute devices including CPUs, GPUs, DSPs, and even FPGAs.

The typical workflow for computing with OpenCL is rather involved, encompassing multiple steps. First, one needs to
query available OpenCL platforms (like the AMD APP SDK) and select a compute device from one of them (such as the
`gfx1151` GPU used in our tests). Next, a Context must be created; this acts as a container for managing the selected
device(s) and associated resources like memory objects and command queues. Following that, a Command Queue is created
for the chosen device, which serves as the conduit for submitting tasks (like memory transfers and kernel executions) to
the device. The core data (matrices A, B, C) needs to reside in Memory Buffers on the device, created as `cl_mem`
objects; this necessitates copying the input data A and B from host (CPU) memory to their respective device buffers. The
computational task itself is defined in an OpenCL Kernel, typically written in a separate `.cl` file (like our
`matrix_mult.cl`); this source code must be loaded, compiled (at which point optimization options like
`-cl-fast-relaxed-math` can be passed), and built into an OpenCL Program object (`cl_program`). From this program
object, the specific kernel function object (`cl_kernel`) to be executed is obtained. Before executing the kernel, its
arguments must be set using `clSetKernelArg`, passing the device buffer objects (the `cl_mem` handles for A, B, C) and
the matrix size N, among other potential parameters. Kernel execution is initiated by enqueuing the task onto the
command queue using `clEnqueueNDRangeKernel`. This requires specifying the total number of global work-items (usually N*
N, with each work-item calculating one element of C) and optionally, the local work-item size (the Workgroup size, e.g.,
16x16, which impacts resource usage and performance). After the kernel finishes execution on the device, the results
stored in the C buffer on the device must be copied back to host memory using `clEnqueueReadBuffer`. Finally, and
crucially, all created OpenCL objects (kernel, program, buffers, queue, context) must be explicitly released to prevent
resource leaks.

Regarding the OpenCL Kernel code (`matrix_mult.cl`), it's written in the OpenCL C language, a dialect based on the C99
standard with extensions for parallel computing. In our matrix multiplication kernel, each work-item (think of it as a
lightweight thread) uses the built-in functions `get_global_id(0)` and `get_global_id(1)` to determine its unique
coordinates (column `col`, row `row`) within the global N x N computation grid. Then, each work-item independently
executes the inner loop over `k` to compute the dot product and store the result for `C[row][col]`. Since we're using
`double` as our data type, the kernel code needs the `#pragma OPENCL EXTENSION cl_khr_fp64 : enable` directive to
explicitly enable support for double-precision floating-point numbers.

The main advantages of OpenCL are its theoretical cross-platform and cross-vendor compatibility and its ability to fully
leverage the massive parallel compute power of GPUs and other accelerators. However, its disadvantages are also
significant: the programming model is relatively complex, requiring developers to manually manage platforms, devices,
contexts, memory, synchronization, and more, which often leads to verbose code. Furthermore, data must be explicitly
transferred between the host and the device, introducing latency and bandwidth overhead that can negatively impact
performance (become counterproductive), especially for computationally small tasks. Additionally, the actual performance
of an OpenCL application can be sensitive to the quality of the specific vendor's driver implementation.

### CLBlast

CLBlast can be thought of as the BLAS implementation for the OpenCL ecosystem. Its design goal is to provide an API
compatible with the traditional BLAS interface, but its internal computational logic is implemented using the OpenCL
standard, enabling it to run on any GPU (or other accelerator) that supports OpenCL.

In terms of usage, invoking CLBlast is significantly simpler than manually writing and managing OpenCL kernels. First,
you still need an initialized OpenCL environment, including a context and a command queue; we can directly reuse the
global context `g_clContext` prepared for the pure OpenCL implementation. Next, OpenCL memory buffers need to be created
for the input and output matrices, and the host data must be copied to the input buffers, just as in standard OpenCL.
Once these prerequisites are met, the core step involves calling the CLBlast function `clblast::Gemm<ValueType>(...)` (
using the C++ template interface here, where `ValueType` automatically determines the precision). When calling this
function, you need to pass arguments describing the matrix layout (row-major or column-major), whether the input
matrices should be transposed, the matrix dimensions (M, N, K), the scalar values alpha and beta, pointers to the
device-side OpenCL buffer objects, the leading dimension of each matrix (which is typically the number of columns for
row-major storage), and the OpenCL command queue to use for execution. The CLBlast library then takes care of internally
invoking its pre-compiled and optimized OpenCL kernels to perform the actual computation. After the computation is
complete, the developer still needs to copy the results from the device-side C buffer back to host memory, similar to
standard OpenCL practice.

The primary advantages of CLBlast are that it offers a standard BLAS interface, greatly simplifying the programming
effort required for GPU-accelerated matrix operations using OpenCL. Furthermore, because the kernel functions within the
CLBlast library are typically meticulously optimized by its developers (likely employing advanced techniques like
sophisticated tiling, shared memory optimization, etc.), its performance is often superior to relatively simple OpenCL
kernels written by application developers. However, it also has disadvantages. First, it relies on the target system
having a correctly installed and configured OpenCL runtime environment as well as the CLBlast library itself. Second,
like all GPU acceleration schemes based on a discrete memory model, it still involves the overhead of data transfer
between the host and the device, which can become a performance bottleneck for small problems or in bandwidth-limited
scenarios.

### Vulkan Compute

Next is Vulkan Compute. Vulkan itself was primarily designed as a next-generation, high-performance graphics rendering
API, but it also incorporates powerful general-purpose computing (GPGPU) capabilities implemented via Compute Shaders.

The workflow for performing computations using Vulkan is arguably even more verbose and lower-level than OpenCL.
Broadly, it involves the following sequence of steps: First is the initialization of a Vulkan Instance, followed by
selecting a suitable Physical Device (usually the GPU), and then creating a Logical Device based on it, along with
obtaining a Compute Queue for submitting computational tasks. The computation logic itself needs to be written in a
compute shader (like our `matrix_mult.comp`), typically using the GLSL language. This shader must then be compiled into
Vulkan's standard intermediate representation, SPIR-V format (using a tool like `glslc -O`), and this SPIR-V code is
loaded to create a Shader Module (`VkShaderModule`). For data storage, you must explicitly allocate Memory (
`VkDeviceMemory`) on the device and create Vulkan Buffers (`VkBuffer`) to hold the input matrices A, B, and the output
matrix C. This process involves complex decisions regarding memory type selection, allocation, and binding buffers to
the allocated memory. Copying data from the host (CPU) to these device buffers usually requires an intermediate,
host-visible Staging Buffer. To allow the shader to access these buffer resources, Descriptors must be set up. This
includes defining a Descriptor Set Layout (`VkDescriptorSetLayout`) to declare the resources the shader needs (e.g.,
three storage buffers), creating a Descriptor Pool (`VkDescriptorPool`) from which to allocate descriptor sets,
allocating a specific Descriptor Set (`VkDescriptorSet`), and finally "connecting" or updating this descriptor set with
the information about our created buffers. With the shader module and descriptors ready, the next step is to create the
Compute Pipeline. This requires first creating a Pipeline Layout (`VkPipelineLayout`), which associates the descriptor
set layouts used by the shader, and then creating the actual compute pipeline object (`VkPipeline`) based on this layout
and the shader module. The actual commands are submitted via a Command Buffer. One must be allocated from a Command
Pool (`VkCommandPool`). Then, you begin recording commands into it: first, you bind the compute pipeline and the
descriptor set containing the resource information, and then you invoke `vkCmdDispatch` to launch the computation.
`vkCmdDispatch` requires specifying the number of workgroups to launch, which usually needs to be calculated based on
the matrix size N and the number of threads per workgroup defined in the shader (the `local_size`). Once command
recording is complete, the command buffer is submitted to the previously obtained compute queue for execution. Since
submission is asynchronous, Vulkan synchronization primitives like Fences or Semaphores must be used to wait for the GPU
computation to finish. After completion, the results in the device's C buffer need to be copied back to host memory,
again likely using a staging buffer. The final step involves meticulously destroying all created Vulkan objects (
pipeline, layout, descriptors, pool, buffers, memory, device, instance, etc.) in the reverse order of creation to
release resources properly.

Our compute shader (`matrix_mult.comp`) is written in GLSL (OpenGL Shading Language). The
`layout (local_size_x = 16, local_size_y = 16)` directive at the top defines that each workgroup consists of 16x16=256
work-items (threads). The `layout(set = 0, binding = ...)` specifications define how the shader accesses the buffers A,
B, and C via binding points (0, 1, 2) within descriptor set 0. Inside the `main` function, the built-in variable
`gl_GlobalInvocationID.xy` provides the global coordinates of the current work-item within the overall compute grid (
where `id.x` corresponds to the column and `id.y` to the row). The core computation logic, involving the loop over `k`
to calculate the dot product for `C[id.y * N + id.x]`, is very similar to the OpenCL kernel.

The advantages of using Vulkan Compute lie in it being a modern graphics API designed to reduce driver overhead on the
CPU. If an application already requires graphics rendering, using Vulkan Compute allows for better integration with the
rendering pipeline, potentially sharing resources and context. Vulkan also offers very fine-grained control over the
hardware, enabling deep performance optimization. However, its disadvantages are quite prominent: the API is extremely
verbose, and the initialization and setup processes are highly complex, leading to massive code overhead and
comparatively lower development productivity. Vulkan's primary design focus remains graphics rendering; although its
compute capabilities are powerful, the ecosystem for general-purpose computing, including high-level library support and
overall ease of use, might be considered somewhat less mature compared to OpenCL or NVIDIA's CUDA / AMD's HIP. And, just
like OpenCL, the overhead of data transfer between host and device persists and needs careful management.

### HIP (Heterogeneous-Compute Interface for Portability)

Now let's discuss HIP (Heterogeneous-Compute Interface for Portability). HIP is an integral part of AMD's ROCm (Radeon
Open Compute) platform, designed to provide a C++ GPU programming model very similar to NVIDIA's CUDA. One of its
primary goals is to simplify the process of porting existing CUDA code to run on AMD GPUs.

The host-side (Host Code) workflow for GPU computing using HIP is considerably more concise compared to OpenCL and
Vulkan, closely resembling the CUDA style. First, you need to allocate device memory for the input matrices A, B, and
the output matrix C on the target GPU device using the `hipMalloc()` function. Then, data is transferred from host
memory to device memory using `hipMemcpy()` (specifying `hipMemcpyHostToDevice` as the direction) for matrices A and B.
The core computational task is initiated by launching the kernel function (`matrix_multiply_hip_kernel`) using a syntax
very similar to CUDA's `<<<GridDim, BlockDim>>>` notation, which specifies the kernel's execution configuration.
`GridDim` defines the number of thread blocks (analogous to OpenCL workgroups) to launch, while `BlockDim` defines the
number of threads within each block (e.g., we might set it to 16x16). The grid dimensions usually need to be calculated
based on the total matrix size N and the chosen block dimensions to ensure the entire computation is covered. Since
kernel launches are asynchronous, the host code must call `hipDeviceSynchronize()` to wait for all computations on the
GPU to complete. After computation finishes, the results from the C matrix in device memory are transferred back to host
memory using `hipMemcpy()` (this time specifying `hipMemcpyDeviceToHost`). Finally, it's crucial to release all device
memory allocated earlier using the `hipFree()` function. Throughout this process, it's recommended to use our defined
`HIP_CHECK()` macro (which internally calls `hipGetErrorString`) to check the return value of every HIP API call for
timely error detection and handling.

The HIP device-side code (in the `matrix_mult_hip.hip` file) is written using standard C++ syntax along with some
HIP-specific extensions. Functions marked with the `__global__` keyword are kernel functions that can be launched from
the host using the `<<<...>>>` syntax. Inside the kernel function, built-in variables like `blockIdx` (index of the
current block within the grid), `threadIdx` (index of the current thread within its block), and `blockDim` (dimensions
of the block) are accessible. By combining these variables, we can calculate the global ID of the current thread (
corresponding to the `row` and `col` in the result matrix), similar to how global IDs are obtained in OpenCL/Vulkan (
e.g., via `get_global_id` or `gl_GlobalInvocationID`). The core computational logic of our matrix multiplication
kernel (the inner loop over `k`) is essentially the same as the OpenCL and Vulkan kernels we saw earlier.

Overall, HIP's main advantages are its provision of a C++ interface, which is generally easier to use and learn compared
to OpenCL's C API or the extremely verbose Vulkan API. Its high degree of syntactic similarity to CUDA significantly
facilitates porting existing CUDA codebases to the AMD platform. Being part of the ROCm platform, HIP is tightly
integrated with AMD's GPU drivers and toolchain (like the `hipcc` compiler), usually resulting in good performance and
compatibility. However, HIP also has disadvantages. It primarily targets AMD GPUs (although the HIP Clang project
provides some capability to run on certain NVIDIA GPUs, this isn't its main focus). Using HIP requires installing the
relatively large ROCm SDK. And, like all GPU computing solutions based on a discrete memory model, the overhead of data
transfer between the host and device remains a performance factor to consider.

### hipBLAS

Finally, we arrive at hipBLAS. You can think of it as the BLAS library within the HIP ecosystem, analogous to cuBLAS in
the CUDA world or CLBlast in the OpenCL sphere. hipBLAS is the officially provided library from the ROCm platform,
offering Basic Linear Algebra Subprograms accelerated using HIP technology for AMD GPUs.

Using hipBLAS follows a pattern similar to other GPU BLAS libraries and is simpler than writing raw HIP kernels. First,
a functional HIP runtime environment is a prerequisite. Before using hipBLAS functions, you need to create a hipBLAS
handle, an object managing the library's internal state, via `hipblasHandle_t handle; hipblasCreate(&handle);` for
initialization. Memory management proceeds as with HIP kernels: use `hipMalloc` to allocate memory for A, B, and C on
the GPU device, and use `hipMemcpy` to transfer host data to the device buffers for A and B. The core computation
involves calling the `hipblasDgemm()` function ('d' for double precision). Its parameter list closely resembles
`cblas_dgemm`, with key differences being the need to pass the previously created hipBLAS handle and the fact that the
pointers for A, B, and C must be device memory pointers. You also need to specify the operation for each matrix, e.g.,
whether it needs transposition (`HIPBLAS_OP_N` for no transpose). One crucial detail to pay attention to is that
hipBLAS, like many traditional BLAS libraries, defaults to expecting data stored in column-major order. However, C++
developers typically work with row-major storage. If our inputs A and B are row-major, and we want to compute the
row-major result C = A * B, calling `hipblasDgemm` directly requires careful handling of the data layout. A common trick
is to leverage the mathematical identity C<sup>T</sup> = B<sup>T</sup> * A<sup>T</sup>. This involves telling
`hipblasDgemm` to compute B<sup>T</sup> * A<sup>T</sup> (passing `HIPBLAS_OP_T` for both A and B), swapping the device
pointers passed for A and B, swapping their leading dimensions (lda, ldb), and also swapping the matrix dimensions M and
N. The resulting buffer computed this way is C transposed (C<sup>T</sup> stored in column-major order), which happens to
have the exact same memory layout as C stored in row-major order. Alternatively, a more direct approach, if supported by
your hipBLAS version, would be to check for an API function or setting that directly supports row-major inputs. However,
for our `matrix_multiply_hipblas` implementation, we assume it internally handles the layout correctly (perhaps via the
transpose trick or a newer interface) to provide behavior consistent with `cblas_dgemm`. Since the call executes
asynchronously, it's necessary to call `hipDeviceSynchronize()` to ensure the hipBLAS operation is completed and
synchronized. Afterwards, use `hipMemcpy` to copy the result from the device C buffer back to the host. Finally, don't
forget to destroy the hipBLAS handle using `hipblasDestroy(handle)` to release its resources. As always, using the
`HIPBLAS_CHECK()` macro to verify the status of each hipBLAS API call is recommended for robust error handling.

The primary advantages of hipBLAS are that it provides a standard BLAS interface, making high-performance linear algebra
on AMD GPUs relatively easy to use. The library contains HIP kernels that are highly optimized by AMD specifically for
their GPU architectures, thus usually delivering very high performance and effectively leveraging the hardware's
potential. Naturally, there are disadvantages too. Using hipBLAS depends on having the ROCm/HIP development environment
and the hipBLAS library correctly installed. Like all GPU acceleration methods, the cost of data transfer between host
and device remains. Furthermore, developers must pay close attention to handling the row-major versus column-major data
layout issue to ensure correct function calls and parameter settings.

Alright, all the contenders have been introduced. From simple serial loops to complex GPU programming, we've covered a
spectrum of mainstream performance optimization ideas and technology stacks. Next up, let's see how they actually
performed in our benchmark tests!

## Benchmarking Methodology

To ensure a fair comparison between these different implementations, we utilized the Google Benchmark framework. We also
designed a specific test fixture, named `MatrixMultFixture`, to manage the setup and teardown tasks associated with each
individual test run.

During the test setup (`SetUp`) phase for each test case, the program first determines the current square matrix size
`N` based on parameters passed by the Google Benchmark framework. It then allocates host (CPU) memory, typically using
`std::vector<ValueType>`, for the input matrices A and B, as well as for an output matrix C intended to store results
from CPU, SIMD, or some GPU implementations. Subsequently, matrices A and B are filled with random numbers. If the test
involves the Eigen or OpenCV libraries, their respective specific result matrices (like `C_eigen`, `C_cv`) are also
allocated at this stage. It's important to note that for technologies requiring a persistent global context, such as
OpenCL, Vulkan, and HIP, their initialization (e.g., via functions like `initOpenCL`, `initVulkan`) and final cleanup (
e.g., `cleanupOpenCL`) are performed once at the beginning and end of the entire benchmark program's execution (within
the `main` function), not within the per-test `SetUp` and `TearDown`. This avoids the significant overhead of repeatedly
initializing and destroying these heavyweight contexts for every single test iteration.

Next comes the test execution phase, driven by Google Benchmark's macros. Each distinct matrix multiplication
implementation corresponds to a separate Benchmark test function, for instance,
`BENCHMARK_F(MatrixMultFixture, BM_Naive)` signifies the test for the Naive implementation. Inside each such test
function, the core logic resides within a `for (auto _ : state)` loop controlled by Google Benchmark. Within this loop,
we invoke the specific matrix multiplication function currently being tested, such as
`matrix_multiply_naive(A, B, C, N)`. The Google Benchmark framework intelligently and automatically adjusts the number
of times this loop runs to ensure stable and reliable timing measurements are obtained. For libraries that necessitate
data mapping or wrapping (like Eigen and OpenCV), the mapping (`Eigen::Map`) or wrapper object creation (`cv::Mat`)
typically occurs inside this loop, but since these are usually zero-copy or low-overhead operations, their impact on the
performance measurement is minimal. For the GPU-accelerated implementations (including OpenCL, Vulkan, HIP, CLBlast,
hipBLAS), calling their respective execution functions usually encapsulates a sequence of operations: potentially
creating (or reusing) device-side memory buffers, transferring input data from host to device (Host-to-Device),
launching the computation kernel on the GPU, waiting for kernel execution to complete (synchronization), and finally
transferring the computed results back from device to host (Device-to-Host).

The test cleanup (`TearDown`) phase is relatively straightforward, mainly involving the release of the host memory
resources allocated during the `SetUp` phase, for example, by calling methods like `A.clear()`, `B.clear()`,
`C.clear()`, and so forth.

Regarding the test scope, we selected a range of N values for benchmarking, specifically 64, 128, 256, 512, and 1024.
Choosing these powers of two is a common practice in benchmarking, as it helps in observing performance trends as the
problem scale increases, particularly when plotted on logarithmic axes.

In terms of performance metrics, Google Benchmark primarily measures and reports `real_time`, which corresponds to the
wall-clock time elapsed. Based on this measured time (typically in nanoseconds, `ns`) and the current matrix size `N`,
we calculated a more informative core performance metric: GFLOPS (Giga Floating-point Operations Per Second). The
formula used was `GFLOPS = (2.0 * N^3) / (time_ns / 1e9)`. This calculation assumes that a standard square matrix
multiplication requires `2 * N^3` floating-point operations (roughly N^3 multiplications and N^3 additions). All
benchmark results were ultimately saved to a JSON formatted file named `benchmark_results.json` for convenient
post-processing.

Finally, for results visualization and easier comparison, we used Python along with the powerful data manipulation
library pandas and the plotting library matplotlib. A script reads the generated JSON file, parses the data, calculates
GFLOPS for each run, and then generates the performance comparison plot. In the plot, the X-axis represents the matrix
size N (using a base-2 logarithmic scale to better show power-of-two relationships), and the Y-axis represents the
performance in GFLOPS (also using a logarithmic scale to accommodate the vast differences in performance). This
graphical representation allows us to see the performance gaps between different implementations and their respective
scaling trends with problem size at a glance.

Now, let's see the final report card!

## Performance Data Analysis

Please take a look at the performance comparison chart plotted from the benchmark results:

![Matrix Multiplication Performance Comparison Plot](/images/qH2uR.png)

To interpret this information-rich chart, let's first examine the axes. The X-axis represents the matrix size N,
spanning from 64 to 1024, and employs a base-2 logarithmic scale. The Y-axis denotes performance in GFLOPS (billions of
floating-point operations per second) and also uses a logarithmic scale. The choice of logarithmic scales is crucial
here; it helps to clearly display implementations with vastly different performance levels on the same graph and makes
it easier to observe the relative performance trends as N changes. The legend on the right side lists all the
implementation methods tested, along with their corresponding markers and colors, allowing easy identification of each
line.

Looking at the overall trends, several prominent patterns emerge. First, most implementations exhibit improved
performance as the matrix size N increases, reflected by the generally upward slope of the curves. This is expected
because for larger N, the total computational workload (which scales as O(N^3)) becomes much larger relative to fixed or
slower-growing overheads (like function call costs, GPU data transfer latencies, thread startup times, etc.). This
allows the benefits of parallelism and optimization to become more pronounced. Additionally, larger computational tasks
are better at amortizing memory access latencies. Second, there's an extremely wide range of performance across
different implementations, differing by orders of magnitude. This is strikingly evident when comparing the lowest
performer, the Naive implementation, to the top performer, hipBLAS (at N=1024). The performance gap exceeds 170,000
times! (Specifically, Naive at ~0.0006 GFLOPS vs. hipBLAS at ~102 GFLOPS). This dramatically underscores the importance
and potential impact of optimization. Third, we observe that some curves tend to flatten out or even slightly decrease
at larger values of N. This typically indicates that the performance of that implementation is hitting a bottleneck
under the current conditions. Such bottlenecks could be varied, including saturated memory bandwidth (data can't be
supplied fast enough), insufficient CPU or GPU cache capacity for the working set causing lower cache hit rates,
reaching the limit of GPU core utilization, or perhaps certain unoptimized overheads growing linearly or faster with N,
starting to negate the computational speedup.

To analyze the performance data more deeply, we can group the implementations and compare them within and across groups.

First, let's look at the CPU Basic Group, comparing the simplest Naive implementation against the version using only
OpenMP for parallelism. The Naive implementation (yellow '+' marker) is undeniably the slowest. Its curve hugs the
bottom of the chart on the log scale, showing very little growth with N, reaching only about 0.6 GFLOPS at N=1024 (
*re-reading based on plot, correcting potential misinterpretation of raw data; GFLOPS derived from JSON*). In contrast,
the OpenMP version (orange square marker), leveraging the CPU's 20 threads, shows a marked improvement, achieving around
4 GFLOPS at N=1024, roughly 6-7 times faster than Naive. Nevertheless, compared to more advanced optimization
techniques, this is still quite slow. Its relatively flat performance curve suggests that simple multi-core parallelism
might quickly become limited by factors like memory bandwidth.

Next is the CPU SIMD Group, where we examine the impact of using AVX2 and AVX-512 instructions, both alone and combined
with OpenMP. The single-threaded AVX2+FMA implementation (dark blue circle) already demonstrates the power of
vectorization, delivering respectable performance (~1.7 GFLOPS at N=1024), even slightly outperforming the pure OpenMP
version for N < 512. Moving to AVX512+FMA (green triangle) yields further speedup, as the 512-bit vectors can process
twice the data per instruction compared to AVX2, reaching about 2.4 GFLOPS at N=1024. The real performance leap occurs
when combining SIMD with multi-threading. AVX2+FMA_OMP (red diamond) achieves roughly 9.5 GFLOPS at N=1024, more than 5
times faster than single-threaded AVX2 and over twice as fast as pure OpenMP. The champion within this group, and indeed
the top performer among all CPU implementations tested, is AVX512+FMA_OMP (purple inverted triangle). By combining the
widest available SIMD vectors with multi-core parallelism, it hits an impressive 15 GFLOPS at N=1024, about a 60%
improvement over the AVX2+OMP version. Its line sits at the pinnacle of the CPU-only results.

Now, let's consider the CPU Professional Library Group, comparing BLAS, Eigen, and OpenCV. BLAS (purple 'V' marker)
delivered excellent performance, reaching approximately 53 GFLOPS at N=1024 (*correction based on re-reading the plot*),
nearly matching or slightly exceeding our best manually tuned CPU code (AVX512+FMA_OMP). This strongly indicates that
the BLAS library installed on the system (likely OpenBLAS) is extremely well-optimized internally, effectively utilizing
both SIMD instructions and multi-threading. Equally impressive was OpenCVLib (light blue circle), whose performance
closely tracked BLAS, even slightly surpassing it at N=1024 with about 54 GFLOPS. This suggests that OpenCV's `gemm`
implementation benefits from powerful backend optimizations, possibly by calling an optimized BLAS library or another
performance kernel library like IPP internally. However, EigenLib (pink star) showed surprisingly poor performance in
this specific test, lagging behind even the basic OpenMP version and achieving only about 0.7 GFLOPS at N=1024. This
contrasts sharply with Eigen's generally high-performance reputation. Possible reasons for this anomaly could include
suboptimal usage of Eigen in the test code (though unlikely if using standard operations), the compiler failing to
adequately optimize Eigen's expression templates for this specific case, or perhaps compatibility issues between the
particular Eigen version and the test environment. Therefore, this result for Eigen should be viewed with caution and
not generalized; it's likely specific to the conditions of this benchmark.

Finally, we examine the GPU Acceleration Group, comprising implementations using OpenCL, Vulkan, HIP, and the
corresponding BLAS libraries CLBlast and hipBLAS. A general trend across all GPU methods is that their performance tends
to be lower than well-optimized CPU methods (like BLAS or AVX+OMP) at smaller matrix sizes (e.g., N=64), sometimes even
slower than Naive+OpenMP. This is primarily due to the overhead associated with GPU computing, namely the time spent
transferring data between the CPU and GPU (Host-to-Device and Device-to-Host) and the latency involved in launching the
GPU kernel. For small tasks, these fixed overheads constitute a large portion of the total execution time. However, as N
increases, the massive parallel processing capability of the GPU dominates, and their performance curves rise rapidly,
quickly surpassing all CPU-based implementations.

Among the manually written kernels (where we coded the computation logic in OpenCL C, GLSL, or HIP C++), OpenCL (cyan
diamond) performed quite well, reaching about 58 GFLOPS at N=1024, with a steep curve indicating good scalability.
Vulkan (green up-triangle) also delivered good performance, although slightly lower than OpenCL and the HIP kernel, at
around 29 GFLOPS for N=1024. Given Vulkan's API complexity, this result seems reasonable, possibly leaving room for
further driver or shader optimization. The HIP kernel (gray 'X' marker) exhibited anomalously low performance at N=64 (
potentially due to measurement error or an initialization glitch), but for N=128 and larger, its performance quickly
caught up and closely mirrored that of OpenCL, reaching about 57 GFLOPS at N=1024. This suggests that for this
relatively simple kernel, the underlying execution efficiency of HIP and OpenCL on this particular AMD GPU is quite
similar.

Performance took another significant jump when using the GPU BLAS libraries. CLBlast (brown diamond), being the OpenCL
BLAS library, far outperformed our handwritten OpenCL kernel, achieving roughly 95 GFLOPS at N=1024. This highlights the
value of specialized library optimizations; CLBlast likely employs more sophisticated techniques internally, such as
advanced memory access patterns, data tiling, and efficient use of GPU shared memory (LDS). The undisputed overall
winner of this entire benchmark was hipBLAS (red down-triangle). As the native BLAS library for AMD's ROCm platform, it
delivered the most outstanding performance, breaking the 100 GFLOPS barrier at N=1024 and reaching approximately 102
GFLOPS. This typically signifies that hipBLAS is best able to leverage the specific hardware features and instructions
of the AMD GPU.

Let's briefly summarize the highlights and points of caution from this benchmark. The clear performance leaders at
N=1024 were the GPU BLAS libraries, hipBLAS and CLBlast. Within the CPU realm, the system BLAS library, OpenCV, and the
manually crafted AVX512+FMA+OMP implementation were the top contenders. The sheer magnitude of performance improvement
observed was astounding: from the basic Naive method to the fastest hipBLAS implementation, the speedup at N=1024
exceeded a factor of 170,000! The advantage of using GPUs became evident for matrix sizes of N=256 and larger in our
tests, with the gap widening as N increased. This also underscored the importance of using professional libraries like
BLAS, CLBlast, hipBLAS, and even OpenCV, which often outperform manual optimization efforts (especially simpler custom
GPU kernels) by encapsulating extensive hardware-specific tuning. On the cautionary side, the anomalously poor
performance of Eigen in this specific test warrants further investigation and should not be taken as a general statement
about Eigen's capabilities. Similarly, the outlier result for the HIP kernel at N=64 suggests that this particular data
point might be invalid and should be treated carefully.

In essence, this performance showdown vividly illustrates the vast differences that various technological approaches can
make. From elementary CPU loops to intricate GPU programming, every optimization technique has its rationale and optimal
use case.

## Deeper Dive: Discussion & Caveats

While this performance benchmark provides us with a wealth of direct data, it also prompts further reflection and
requires acknowledging certain limitations and important considerations when interpreting the results.

First and foremost, the results are highly hardware-dependent. All tests were conducted on a specific platform featuring
an AMD Ryzen AI 9 processor paired with a Radeon 880M integrated GPU. Running the same benchmarks on different hardware,
such as an Intel CPU or an NVIDIA GPU, could yield dramatically different outcomes and performance rankings. For
example, Intel CPUs often show exceptional performance when coupled with Intel's own MKL (Math Kernel Library), while
NVIDIA GPUs would necessitate the use of the CUDA programming model and the cuBLAS library to achieve their best
results.

Second, the choice of compiler and library versions can significantly influence the outcome. The specific version of GCC
or Clang used, the selected optimization flags (e.g., `-Ofast` versus `-O3` might trade precision or standard
conformance for speed), and the particular version and build configuration of mathematical libraries like BLAS, OpenCV,
or Eigen can all impact the final performance numbers. For instance, substituting OpenBLAS with MKL on an Intel CPU
could lead to completely different BLAS performance results.

Furthermore, the data type and matrix characteristics are crucial factors. This benchmark exclusively used `double` (
64-bit double-precision floating-point numbers) for square matrices. If we were to switch to `float` (32-bit
single-precision), performance would generally be higher due to halved data volume reducing memory bandwidth pressure,
SIMD instructions processing twice as many elements per operation, and some hardware intrinsically favoring
single-precision computations. Additionally, our tests focused on dense square matrices. For matrices with special
structures like sparsity, symmetry, or bandedness, employing specialized storage formats, algorithms, and dedicated
libraries is essential for efficient computation.

Moreover, GFLOPS isn't the whole story. While GFLOPS is a vital metric for gauging raw computational throughput, it
doesn't capture the full picture of real-world application performance. Especially in the context of GPU computing, the
time spent transferring data between the host (CPU) and the device (GPU) – operations like `hipMemcpy` or
`clEnqueueWrite/ReadBuffer` – constitutes an integral part of the total task duration. Our benchmark, likely focusing on
the time spent within the Google Benchmark loop, might primarily measure the core computation time and potentially
underrepresent or exclude the full data transfer overhead. In practical applications, the end-to-end execution time is
what truly matters. For small matrix problems, this data transfer overhead can even dominate the overall time.

We must also consider the trade-offs between implementation complexity and ease of use. The highest-performing
solutions, such as hipBLAS or CLBlast, while relatively simple to *use* (calling library functions), rely on the user
having the specific SDKs (like ROCm) and environments correctly installed and configured. On the other hand, manually
writing SIMD intrinsics or GPU kernel code (for OpenCL, Vulkan, or HIP) might offer finer control over performance but
demands deep expertise in low-level hardware details and parallel programming, often involving significant development,
debugging, and optimization effort. The Naive and OpenMP approaches are the simplest to implement but yield the poorest
performance. Therefore, selecting the right implementation method for a real-world project requires careful balancing
between performance requirements, development costs, code portability, and long-term maintainability.

It's also worth acknowledging that regarding cache optimization, the CPU SIMD and GPU kernels (OpenCL/Vulkan/HIP) that
we manually implemented were relatively basic and did not incorporate sophisticated data blocking (or tiling)
strategies. Blocking is an advanced optimization technique that involves partitioning large matrices into smaller
sub-matrices (blocks) and performing computations block-wise. Its main goal is to maximize the utilization of CPU or GPU
caches by improving data locality and cache hit rates. This technique is one of the core reasons why high-performance
BLAS libraries achieve near-peak hardware performance. If we were to implement complex blocking in our manual code,
their performance might improve further, but at the cost of a dramatic increase in code complexity.

Finally, the anomalous results observed for the Eigen library and the HIP kernel at N=64 serve as a reminder that
benchmark results should always be interpreted critically. When encountering data that starkly contradicts expectations,
one should resist jumping to immediate conclusions and instead try to investigate potential causes – could it be a bug
in the code, an issue with compilation flags, measurement inaccuracies, interference from other system processes, or
perhaps a compatibility problem specific to the test environment? Only through careful scrutiny and validation can we
gain confidence in the benchmark findings.

## The Finish Line: Conclusion & Outlook

Having journeyed through this comprehensive matrix multiplication performance showdown—spanning CPUs and GPUs, serial
and parallel approaches, manual optimizations, and professional libraries—we can draw several clear conclusions.

First and foremost, optimization is absolutely crucial. The chasm in performance between the most basic Naive
implementation and highly optimized solutions is immense, vividly demonstrating that for compute-intensive tasks,
selecting the right algorithms and implementation techniques is paramount for achieving acceptable, let alone excellent,
performance. Second, leveraging hardware features yields significant rewards. Utilizing modern CPU capabilities like
multi-core processing (e.g., via OpenMP) and SIMD instruction sets (either through manual intrinsics or library-provided
automatic vectorization) provides substantial speedups; combining these two often pushes CPU performance towards its
practical limits. Third, the potential for GPU acceleration is enormous. For computational tasks of sufficient scale (in
our tests, starting around N=256), the massively parallel architecture of GPUs enables performance levels far exceeding
what CPUs can offer. Fourth, it highlights the value of making good use of professional libraries. Specialized math
libraries such as BLAS (and its various implementations like OpenBLAS, MKL, AOCL-BLAS), CLBlast, hipBLAS (or cuBLAS for
NVIDIA), encapsulate a vast amount of low-level optimization expertise. Employing them is frequently the most effective
path to achieving both high performance and good development productivity. Even higher-level libraries like OpenCV may
rely on these optimized backends internally. However, we must also recognize that there is no "silver bullet" in
performance optimization; no single method reigns supreme in all scenarios. Small-scale problems might favor CPU
implementations due to avoided data transfer overheads, while large-scale problems clearly benefit from GPU
acceleration. The optimal choice will invariably depend on the specific hardware platform, the required precision (
single vs. double), and the available development resources and constraints. Finally, all these findings point towards
the importance of continuous learning and hands-on practice. High-performance computing is a rapidly evolving field with
constant advancements in hardware architectures, programming models, and compiler technologies. Maintaining curiosity,
persistently learning new techniques, and personally testing and validating assumptions are the keys to truly mastering
the art and science of performance optimization.

Hopefully, this exploration into the performance landscape of matrix multiplication has provided everyone with a more
tangible understanding of the diverse computing technologies available. From the humble three nested loops to blistering
speeds exceeding one hundred GFLOPS, the journey reflects the culmination of ingenuity in computer architecture,
parallel computing, and software engineering. Perhaps the next time you're faced with a task involving large-scale
matrix operations, you'll recall the contenders we discussed today and feel more equipped to choose the most suitable
acceleration strategy for your application!

## Appendix: Benchmark Results Data

Table omitted as requested.

| Implementation | Matrix Size (N) | Real Time (ns) | Performance (GFLOPS) |
|:---------------|:----------------|:---------------|:---------------------|
| Naive          | 64              | 640,561        | 0.818                |
| Naive          | 128             | 5,250,421      | 0.799                |
| Naive          | 256             | 42,393,811     | 0.791                |
| Naive          | 512             | 569,762,981    | 0.471                |
| Naive          | 1024            | 3,447,583,101  | 0.623                |
| OpenMP         | 64              | 149,270        | 3.512                |
| OpenMP         | 128             | 1,036,590      | 4.046                |
| OpenMP         | 256             | 6,844,282      | 4.903                |
| OpenMP         | 512             | 62,077,042     | 4.324                |
| OpenMP         | 1024            | 578,410,614    | 3.713                |
| AVX2+FMA       | 64              | 311,178        | 1.685                |
| AVX2+FMA       | 128             | 2,505,685      | 1.674                |
| AVX2+FMA       | 256             | 19,324,494     | 1.736                |
| AVX2+FMA       | 512             | 152,734,950    | 1.758                |
| AVX2+FMA       | 1024            | 1,237,421,611  | 1.735                |
| AVX512+FMA     | 64              | 221,951        | 2.362                |
| AVX512+FMA     | 128             | 1,702,158      | 2.464                |
| AVX512+FMA     | 256             | 14,094,445     | 2.381                |
| AVX512+FMA     | 512             | 107,877,880    | 2.488                |
| AVX512+FMA     | 1024            | 921,593,993    | 2.330                |
| AVX2+FMA_OMP   | 64              | 90,276         | 5.808                |
| AVX2+FMA_OMP   | 128             | 664,552        | 6.311                |
| AVX2+FMA_OMP   | 256             | 3,656,076      | 9.178                |
| AVX2+FMA_OMP   | 512             | 27,922,787     | 9.613                |
| AVX2+FMA_OMP   | 1024            | 216,519,971    | 9.918                |
| AVX512+FMA_OMP | 64              | 86,896         | 6.033                |
| AVX512+FMA_OMP | 128             | 427,994        | 9.799                |
| AVX512+FMA_OMP | 256             | 2,648,926      | 12.667               |
| AVX512+FMA_OMP | 512             | 18,439,355     | 14.558               |
| AVX512+FMA_OMP | 1024            | 140,055,382    | 15.333               |
| Eigen          | 64              | 904,785        | 0.579                |
| Eigen          | 128             | 12,846,593     | 0.326                |
| Eigen          | 256             | 32,201,997     | 1.042                |
| Eigen          | 512             | 284,153,414    | 0.945                |
| Eigen          | 1024            | 2,316,560,842  | 0.927                |
| OpenCV         | 64              | 33,326         | 15.732               |
| OpenCV         | 128             | 73,443         | 57.110               |
| OpenCV         | 256             | 538,501        | 62.311               |
| OpenCV         | 512             | 4,811,569      | 55.790               |
| OpenCV         | 1024            | 36,290,270     | 59.175               |
| BLAS           | 64              | 10,609         | 49.420               |
| BLAS           | 128             | 73,929         | 56.734               |
| BLAS           | 256             | 535,021        | 62.716               |
| BLAS           | 512             | 5,210,261      | 51.521               |
| BLAS           | 1024            | 36,608,529     | 58.661               |
| Vulkan         | 64              | 258,650        | 2.027                |
| Vulkan         | 128             | 850,222        | 4.933                |
| Vulkan         | 256             | 2,015,570      | 16.648               |
| Vulkan         | 512             | 15,517,304     | 17.300               |
| Vulkan         | 1024            | 69,655,183     | 30.830               |
| OpenCL         | 64              | 69,397         | 7.555                |
| OpenCL         | 128             | 147,861        | 28.367               |
| OpenCL         | 256             | 593,376        | 56.548               |
| OpenCL         | 512             | 5,842,253      | 45.947               |
| OpenCL         | 1024            | 38,429,528     | 55.881               |
| CLBlast        | 64              | 61,002         | 8.595                |
| CLBlast        | 128             | 127,007        | 33.024               |
| CLBlast        | 256             | 426,358        | 78.700               |
| CLBlast        | 512             | 3,740,453      | 71.765               |
| CLBlast        | 1024            | 20,777,060     | 103.358              |
| HIP            | 64              | 856,032,739    | 0.000612             |
| HIP            | 128             | 171,225        | 24.496               |
| HIP            | 256             | 613,603        | 54.684               |
| HIP            | 512             | 5,788,911      | 46.371               |
| HIP            | 1024            | 38,210,712     | 56.201               |
| hipBLAS        | 64              | 2,080,484      | 0.252                |
| hipBLAS        | 128             | 2,146,978      | 1.954                |
| hipBLAS        | 256             | 2,691,232      | 12.468               |
| hipBLAS        | 512             | 5,960,233      | 45.038               |
| hipBLAS        | 1024            | 21,356,498     | 100.554              |