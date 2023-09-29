# gxhash-rust
![CI](https://github.com/ogxd/gxhash-rust/actions/workflows/rust.yml/badge.svg)

The fastest non-cryptographic hashing algorithm

## Publication

> I'm committed to the open dissemination of scientific knowledge. In an era where access to information is more democratized than ever, I believe that science should be freely available to all – both for consumption and contribution. Traditional scientific journals often involve significant financial costs, which can introduce biases and can shift the focus from purely scientific endeavors to what is currently trendy. 
>
> To counter this trend and to uphold the true spirit of research, I have chosen to share my work on "gxhash" directly on GitHub, ensuring that it's openly accessible to anyone interested. Additionally, the use of a free Zenodo DOI ensures that this research is citable and can be referenced in other works, just as traditional publications are. 
>
> I strongly believe in a world where science is not behind paywalls, and I encourage other researchers to join this movement for a more inclusive, unbiased, and open scientific community.
>
> _— Olivier Giniaux_

Cite this article / algorithm:  
[![DOI](https://zenodo.org/badge/690754256.svg)](https://zenodo.org/badge/latestdoi/690754256)

## Prerequisites

- Compatible CPU
    - x86-64 bit OR
    - ARM 64-bit
- Rust with nightly enabled `rustup default nightly`
- Environment variable `RUSTFLAGS="-C target-cpu=native"` (windows powershell `$env:RUSTFLAGS = "-C target-cpu=native"`). Required for binary to be compiled for current CPU, marking use of instrinsics. Hopefully simpler in the future thanks to [portable-simd](https://github.com/rust-lang/portable-simd) initiative.

## Benchmarks

Displayed numbers is throughput in Gibibytes of data hashed per second. Higher is better.  
To run the benchmarks: `cargo bench --bench throughput` (don't forget the env flag)

### GCP n2-standard-2 (Intel Ice Lake)

| Method           |      4 |     16 |     64 |    256 |   1024 |   4096 |  16384 |
| ---------------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| t1ha-0 AVX2      |  0.921 | 3.5989 | 11.858 | 32.581 | 52.552 | 60.746 | 65.863 |
| gxhash-0 AVX2    | 11.638 | 46.292 | 31.353 | 81.483 | 103.91 | 101.29 | 99.666 |
| gxhash-0 AVX-512 | 10.927 | 43.136 | 27.944 | 68.942 | 136.63 | 112.16 | 105.32 |

I suspect memory bandwidth to be very high on this GCP server, which is one less bottleneck for gxhash to achieve even greater throughput for small input sizes.

### AMD Ryzen 5 5625U

| Method           | 4      | 16     | 64     | 256    | 1024   | 4096   | 16384  |
|------------------|--------|--------|--------|--------|--------|--------|--------|
| fnv1a            | 2.5703 | 2.2739 | 1.4179 |   1.03 |   1.01 | 1.0068 | 1.0077 |
| xxhash           |  0.439 |  1.003 | 3.0224 | 6.1791 | 8.3379 | 9.1554 | 9.3548 |
| highwayhash      |  0.131 |   0.52 | 2.9722 | 7.4698 | 11.422 | 12.954 |  15.69 |
| t1ha-0 AVX2      |  0.857 | 3.1684 | 11.146 | 34.312 | 71.562 | 87.984 | 85.248 |
| gxhash-0 AVX2    | 1.7197 | 6.9709 |  44.02 | 98.562 | 149.36 | 187.21 | 211.54 |

## Debugging

Algorithm is mostly inlined, making most profilers fail at providing useful intrinsics. The best I could achieve is profiling at assembly level. [cargo-asm](https://github.com/gnzlbg/cargo-asm) is an easy way to view the actual generated assembly code (`cargo asm gxhash::gxhash::gxhash`). [AMD μProf](https://www.amd.com/en/developer/uprof.html) gives some useful insights on time spent per instruction.