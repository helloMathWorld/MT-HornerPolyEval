# <p align="center">MT-HornerPolyEval</p>

<p align="center">
  <img src="https://img.shields.io/badge/language-C-blue.svg">
  <img src="https://img.shields.io/badge/threading-pthreads-green.svg">
  <img src="https://img.shields.io/badge/license-MIT-lightgrey.svg">
  <img src="https://img.shields.io/badge/type-microbenchmark-orange.svg">
</p>

Multi-Threaded Horner Polynomial Evaluation Benchmark (C / pthreads)

A lightweight and portable microbenchmark designed to evaluate CPU floating-point
throughput and thread scheduling overhead using multi-threaded polynomial
evaluation based on Horner’s method.

The implementation follows a performance-engineering–oriented style inspired by
projects such as `unum-cloud/usearch`, emphasizing explicit work partitioning,
lock-free reduction, and deterministic verification.

---

## Overview

This benchmark evaluates a high-degree polynomial over a large number of sample
points in parallel.

Key characteristics:

- Compute-bound workload
- Minimal memory traffic
- Deterministic execution and checksum
- Suitable for cross-machine and cross-compiler comparison

Typical use cases include:

- Floating-point throughput evaluation
- pthreads scalability analysis
- Compiler optimization comparison (`-O2`, `-O3`, `-march`, `-ffast-math`)

---

## Features

- **Multi-threaded execution**
  - Implemented with pthreads
  - Explicit static work partitioning
  - No locks or atomics in the hot path

- **Horner polynomial kernel**
  - Default: 1400 coefficients
  - Deep dependency chain
  - Floating-point intensive

- **Large deterministic sampling**
  - Uniform sampling over `[-1, 1]`
  - Default: 200,000 samples

- **Deterministic checksum**
  - Per-thread 64-bit mixing
  - XOR reduction in main thread
  - Stable and reproducible across runs

- **Compile-time configuration**
  - Threads, samples, and coefficients configured via macros
  - No runtime arguments (benchmark-friendly)

---

## Build

### Requirements

- Standard C compiler
- pthreads (available by default on Windows, Linux and macOS)

### GCC / Clang

```shell
gcc -O2 -march=native -pthread mt_horner_poly_benchmark.c -o mt_horner_bench
# or
clang -O2 -march=native -pthread mt_horner_poly_benchmark.c -o mt_horner_bench
# run
./mt_horner_bench
# Example output:
[pure-math] threads=4 samples=200000 coef=1400 checksum=0xdccb8034e650e7ef
```

## Field description:

| Field    | Description                       |
| -------- | --------------------------------- |
| threads  | Number of worker threads          |
| samples  | Total number of sample points     |
| coef     | Number of polynomial coefficients |
| checksum | Deterministic verification value  |

---

## Compile-time Configuration

Default parameters can be overridden using compile-time macros.

| Macro     | Default | Description                       |
| --------- | ------- | --------------------------------- |
| N_THREADS | 4       | Number of threads                 |
| N_SAMPLES | 200000  | Total number of sample points     |
| N_COEF    | 1400    | Number of polynomial coefficients |

Example: 8 threads, 1,000,000 samples, 1400 coefficients

```shell
gcc -O2 -march=native -pthread 
 -DN_THREADS=8 -DN_SAMPLES=1000000 -DN_COEF=1400 
 mt_horner_poly_benchmark.c -o mt_horner_bench
```

---

## Implementation Notes

### Polynomial Kernel

For each sample point `x`, the polynomial is evaluated using Horner’s method:

$acc = (...((c[n-1] * x + c[n-2]) * x + ...) * x + c[0])$

This form provides:

- Reduced number of multiplications
- Sequential dependency chain
- High floating-point utilization

---

### Work Partitioning

The total number of samples is statically divided among threads:

- `base = N_SAMPLES / N_THREADS`
- The first `N_SAMPLES % N_THREADS` threads process one extra sample

This ensures balanced workload and zero synchronization during computation.

---

### Checksum Reduction

- Each thread computes a local 64-bit checksum
- Floating-point results are mixed at bit level
- Final checksum is produced by XOR reduction in the main thread

No mutexes or atomic operations are used.

---

## Benchmarking Notes

- Pin threads or fix CPU frequency to reduce noise
- Use physical core count as thread count
- Increase sample size for improved statistical stability
- Compare different compiler flags and toolchains

---

## Inspiration

Design patterns in this benchmark are inspired by:

- `unum-cloud/usearch`

Including:

- Static work slicing
- Lock-free reduction
- Hash-style result verification

---

## Contributing

Contributions are welcome, including but not limited to:

- Wall-time or perf-counter instrumentation
- OpenMP comparison implementation
- SIMD / FMA optimizations
- Alternative input distributions
