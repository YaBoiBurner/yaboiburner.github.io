---
layout: post
title: Compression comparison
date: 2020-12-30
author: Jaden Pleasants
tags: compression
---

I thought it might be fun to take a look at some compression tools and see how
they stack up against each other.

I'll primarily be testing performance, compression ratios, and some capabilities.

## Tools tested

- [bzip2](http://www.bzip.org/)
- [gzip](https://www.gzip.org/)
- [pbzip2](https://launchpad.net/pbzip2) (Parallel `bzip2` implementation)
- [pigz](https://www.zlib.net/pigz/) (Parallel `gzip` implementation)
- [xz](https://tukaani.org/xz/)
- [zstd](https://github.com/facebook/zstd)

Benchmarking done using [hyperfine](https://github.com/sharkdp/hyperfine).

## Test files

For repetitive data:

- ~753MB of `/dev/zero`
- ~3.1GB of `/dev/zero`

Realistic tests:

- A `.tar` archive of the [Homebrew/homebrew-core](https://github.com/Homebrew/homebrew-core) repo (~406MB)
- The `ffmpeg-all.1` manual page (~1.5MB)
- The `gcc.1` manual page (~1.3MB)
- The `gcc.info` info document (~3.1MB)

## Compression ratios

It's worth mentioning here that `pbzip2` and `bzip2` should be functionally the
same for compression ratios.
This also partially applies to `pigz` and `gzip`, although `pigz -11` uses a
different compression algorithm than `gzip`.

| File                | Size    | `bzip2`         | `gzip`          | `pigz -11`      | `xz`            | `zstd`          |
| ------------------- | ------- | --------------- | --------------- | --------------- | --------------- | --------------- |
| `ffmpeg-all.1`      | 1.51MB  | 300.3KB (19.8%) | 385.5KB (25.5%) | 368.3KB (24.4%) | 295.0KB (19.5%) | 375.5KB (24.9%) |
| `gcc.1`             | 1.33MB  | 271.5KB (20.1%) | 351.1KB (26.3%) | 333.4KB (25.1%) | 273.4KB (20.5%) | 351.2KB (26.3%) |
| `gcc.info`          | 3.05MB  | 571.1KB (18.7%) | 725.0KB (23.8%) | 681.5KB (22.3%) | 552.3KB (18.1%) | 730.0KB (23.9%) |
| `homebrew-core.tar` | 407.8MB | 375.0MB (92.2%) | 380.2MB (93.5%) | 379.5MB (93.3%) | 350.8MB (86.2%) | 367.8MB (90.4%) |
| `zero@753MB`        | 753.4MB | 562B (0%)       | 731.2KB (0.1%)  | 822.0KB (0.1%)  | 109.7KB (0%)    | 23.6KB (0%)     |
| `zero@3.1GB`        | 3.06GB  | 2.2KB (0%)      | 3.0MB (0.1%)    | 3.3MB (0.1%)    | 445.7KB (0%)    | 96.1KB (0%)     |

I'm not too surprised that `xz` managed to do better than everything else.
More notable, often `gzip` was one of, if not the worst tools.
In a grand total of no tests, `gzip` had the best compression ratios.

## Performance

To start, I'll test the performance of similar tools.

### `bzip2` & `pbzip2`

#### Compression

`ffmpeg-all.1`

| Command                  |   Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :----------------------- | ----------: | -------: | -------: | ----------: |
| `bzip2 ffmpeg-all.1 -c`  | 103.8 ± 2.6 |    100.1 |    113.0 | 1.53 ± 0.08 |
| `pbzip2 ffmpeg-all.1 -c` |  67.8 ± 2.9 |     63.8 |     81.0 |        1.00 |

`gcc.1`

| Command           |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :---------------- | ---------: | -------: | -------: | ----------: |
| `bzip2 gcc.1 -c`  | 88.1 ± 2.8 |     84.5 |     99.3 | 1.39 ± 0.07 |
| `pbzip2 gcc.1 -c` | 63.4 ± 2.4 |     61.1 |     74.3 |        1.00 |

`gcc.info`

| Command              |   Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :------------------- | ----------: | -------: | -------: | ----------: |
| `bzip2 gcc.info -c`  | 191.3 ± 7.6 |    184.6 |    242.3 | 2.63 ± 0.15 |
| `pbzip2 gcc.info -c` |  72.6 ± 2.8 |     66.8 |     82.8 |        1.00 |

`homebrew-core.tar`

| Command                       |       Mean [s] | Min [s] | Max [s] |    Relative |
| :---------------------------- | -------------: | ------: | ------: | ----------: |
| `bzip2 homebrew-core.tar -c`  | 38.936 ± 0.197 |  38.681 |  39.190 | 2.82 ± 0.18 |
| `pbzip2 homebrew-core.tar -c` | 13.815 ± 0.876 |  11.768 |  14.946 |        1.00 |

Oh.

`zero@753MB`

| Command                |      Mean [s] | Min [s] | Max [s] |    Relative |
| :--------------------- | ------------: | ------: | ------: | ----------: |
| `bzip2 zero@753MB -c`  | 5.071 ± 0.043 |   5.009 |   5.142 | 4.29 ± 0.06 |
| `pbzip2 zero@753MB -c` | 1.183 ± 0.013 |   1.150 |   1.199 |        1.00 |

Oh dear.

`zero@3.1GB`

| Command                |       Mean [s] | Min [s] | Max [s] |    Relative |
| :--------------------- | -------------: | ------: | ------: | ----------: |
| `bzip2 zero@3.1GB -c`  | 20.382 ± 0.056 |  20.283 |  20.441 | 4.22 ± 0.08 |
| `pbzip2 zero@3.1GB -c` |  4.829 ± 0.094 |   4.636 |   4.937 |        1.00 |

#### Extraction

`ffmpeg-all.1`

| Command                       |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :---------------------------- | ---------: | -------: | -------: | ----------: |
| `bzip2 ffmpeg-all.1.bz2 -dc`  | 39.7 ± 1.3 |     37.8 |     44.2 |        1.00 |
| `pbzip2 ffmpeg-all.1.bz2 -dc` | 41.4 ± 2.5 |     38.8 |     54.1 | 1.04 ± 0.07 |

`gcc.1`

| Command                |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :--------------------- | ---------: | -------: | -------: | ----------: |
| `bzip2 gcc.1.bz2 -dc`  | 34.8 ± 1.1 |     32.8 |     38.7 |        1.00 |
| `pbzip2 gcc.1.bz2 -dc` | 36.0 ± 0.8 |     34.7 |     38.0 | 1.04 ± 0.04 |

`gcc.info`

| Command                   |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :------------------------ | ---------: | -------: | -------: | ----------: |
| `bzip2 gcc.info.bz2 -dc`  | 71.4 ± 2.4 |     68.3 |     78.7 | 1.00 ± 0.04 |
| `pbzip2 gcc.info.bz2 -dc` | 71.3 ± 1.5 |     68.9 |     76.9 |        1.00 |

`homebrew-core.tar`
zero@753MB.bz2
| Command | Mean [s] | Min [s] | Max [s] | Relative |
|:---|---:|---:|---:|---:|
| `bzip2 homebrew-core.tar.bz2 -dc` | 18.700 ± 0.215 | 18.386 | 19.187 | 3.24 ± 0.07 |
| `pbzip2 homebrew-core.tar.bz2 -dc` | 5.764 ± 0.104 | 5.610 | 5.966 | 1.00 |

`zero@753MB`

| Command                     |      Mean [s] | Min [s] | Max [s] |    Relative |
| :-------------------------- | ------------: | ------: | ------: | ----------: |
| `bzip2 zero@753MB.bz2 -dc`  | 1.976 ± 0.017 |   1.949 |   2.007 | 1.00 ± 0.01 |
| `pbzip2 zero@753MB.bz2 -dc` | 1.975 ± 0.009 |   1.960 |   1.986 |        1.00 |

`zero@3.1GB`

| Command                     |      Mean [s] | Min [s] | Max [s] |    Relative |
| :-------------------------- | ------------: | ------: | ------: | ----------: |
| `bzip2 zero@3.1GB.bz2 -dc`  | 8.019 ± 0.012 |   8.002 |   8.042 | 1.00 ± 0.01 |
| `pbzip2 zero@3.1GB.bz2 -dc` | 8.015 ± 0.041 |   7.944 |   8.075 |        1.00 |

#### Results

This is one of the more interesting things I've seen.
None of the results for compression are really surprises - `pbzip2` outperformed `bzip2` handily.
But for extraction, `pbzip2` and `bzip2` were evenly matched outside of one of the real-world tests.

I'm guessing that `pbzip2` is _capable_ of outperforming `bzip2`, but it really only kicks in at a specific file size.

Either way, `pbzip2` seems like a suitable replacement for `bzip2`, so we'll just test `pbzip2` going forward.

### `gzip` & `pigz`

Considering that `pigz -11` was negligibly smaller than `gzip` if at all, I'm not going to test it.

#### Compression

`ffmpeg-all.1`

| Command                |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :--------------------- | ---------: | -------: | -------: | ----------: |
| `gzip ffmpeg-all.1 -c` | 47.7 ± 0.8 |     45.8 |     49.9 | 3.91 ± 0.14 |
| `pigz ffmpeg-all.1 -c` | 12.2 ± 0.4 |     11.5 |     13.4 |        1.00 |

`gcc.1`

| Command         |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :-------------- | ---------: | -------: | -------: | ----------: |
| `gzip gcc.1 -c` | 46.4 ± 1.3 |     44.3 |     51.1 | 3.66 ± 0.26 |
| `pigz gcc.1 -c` | 12.7 ± 0.8 |     11.8 |     15.8 |        1.00 |

`gcc.info`

| Command            |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :----------------- | ---------: | -------: | -------: | ----------: |
| `gzip gcc.info -c` | 98.9 ± 2.2 |     95.9 |    109.8 | 4.47 ± 0.15 |
| `pigz gcc.info -c` | 22.1 ± 0.6 |     21.3 |     24.8 |        1.00 |

`homebrew-core.tar`

| Command                     |       Mean [s] | Min [s] | Max [s] |    Relative |
| :-------------------------- | -------------: | ------: | ------: | ----------: |
| `gzip homebrew-core.tar -c` | 11.028 ± 0.081 |  10.931 |  11.145 | 4.96 ± 0.13 |
| `pigz homebrew-core.tar -c` |  2.222 ± 0.057 |   2.149 |   2.318 |        1.00 |

`zero@753MB`

| Command              |      Mean [s] | Min [s] | Max [s] |    Relative |
| :------------------- | ------------: | ------: | ------: | ----------: |
| `gzip zero@753MB -c` | 3.570 ± 0.031 |   3.539 |   3.637 | 3.78 ± 0.12 |
| `pigz zero@753MB -c` | 0.944 ± 0.028 |   0.893 |   0.998 |        1.00 |

`zero@3.1GB`

| Command              |       Mean [s] | Min [s] | Max [s] |    Relative |
| :------------------- | -------------: | ------: | ------: | ----------: |
| `gzip zero@3.1GB -c` | 14.485 ± 0.039 |  14.407 |  14.538 | 3.76 ± 0.07 |
| `pigz zero@3.1GB -c` |  3.854 ± 0.071 |   3.695 |   3.966 |        1.00 |

#### Extraction

`ffmpeg-all.1`

| Command                    | Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :------------------------- | --------: | -------: | -------: | ----------: |
| `gzip ffmpeg-all.1.gz -dc` | 8.3 ± 0.3 |      7.6 |     10.2 | 1.52 ± 0.11 |
| `pigz ffmpeg-all.1.gz -dc` | 5.4 ± 0.4 |      5.1 |      7.4 |        1.00 |

`gcc.1`

| Command             | Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :------------------ | --------: | -------: | -------: | ----------: |
| `gzip gcc.1.gz -dc` | 7.4 ± 0.2 |      6.8 |      8.5 | 1.52 ± 0.06 |
| `pigz gcc.1.gz -dc` | 4.9 ± 0.1 |      4.7 |      5.6 |        1.00 |

`gcc.info`

| Command                |  Mean [ms] | Min [ms] | Max [ms] |    Relative |
| :--------------------- | ---------: | -------: | -------: | ----------: |
| `gzip gcc.info.gz -dc` | 15.8 ± 0.5 |     14.7 |     17.7 | 1.68 ± 0.06 |
| `pigz gcc.info.gz -dc` |  9.4 ± 0.1 |      9.2 |      9.8 |        1.00 |

`homebrew-core.tar`

| Command                         |      Mean [s] | Min [s] | Max [s] |    Relative |
| :------------------------------ | ------------: | ------: | ------: | ----------: |
| `gzip homebrew-core.tar.gz -dc` | 2.225 ± 0.013 |   2.197 |   2.243 | 2.05 ± 0.02 |
| `pigz homebrew-core.tar.gz -dc` | 1.087 ± 0.007 |   1.079 |   1.102 |        1.00 |

`zero@753MB`

| Command                  |      Mean [s] | Min [s] | Max [s] |    Relative |
| :----------------------- | ------------: | ------: | ------: | ----------: |
| `gzip zero@753MB.gz -dc` | 2.731 ± 0.010 |   2.713 |   2.744 | 3.30 ± 0.08 |
| `pigz zero@753MB.gz -dc` | 0.828 ± 0.021 |   0.812 |   0.880 |        1.00 |

`zero@3.1GB`

| Command                  |       Mean [s] | Min [s] | Max [s] |    Relative |
| :----------------------- | -------------: | ------: | ------: | ----------: |
| `gzip zero@3.1GB.gz -dc` | 11.099 ± 0.047 |  10.987 |  11.169 | 3.32 ± 0.03 |
| `pigz zero@3.1GB.gz -dc` |  3.345 ± 0.022 |   3.325 |   3.397 |        1.00 |

#### Results

This feels kinda unfair.

`pigz` reliably ran significantly faster than `gzip`, once again to the point where I don't see any reason to test `gzip` again.

### Cross-tool testing

#### Compression

`ffmpeg-all.1`

| Command                  |    Mean [ms] | Min [ms] | Max [ms] |     Relative |
| :----------------------- | -----------: | -------: | -------: | -----------: |
| `pbzip2 ffmpeg-all.1 -c` |   67.0 ± 2.3 |     64.4 |     80.3 |  6.28 ± 1.21 |
| `pigz ffmpeg-all.1 -c`   |   12.3 ± 0.4 |     11.7 |     13.4 |  1.15 ± 0.22 |
| `xz ffmpeg-all.1 -c`     | 417.2 ± 14.8 |    403.9 |    486.0 | 39.12 ± 7.53 |
| `zstd ffmpeg-all.1 -c`   |   10.7 ± 2.0 |      8.7 |     18.1 |         1.00 |

`gcc.1`

| Command           |    Mean [ms] | Min [ms] | Max [ms] |     Relative |
| :---------------- | -----------: | -------: | -------: | -----------: |
| `pbzip2 gcc.1 -c` |   63.2 ± 1.9 |     60.9 |     72.3 |  6.81 ± 1.07 |
| `pigz gcc.1 -c`   |   12.3 ± 0.5 |     11.7 |     14.7 |  1.32 ± 0.21 |
| `xz gcc.1 -c`     | 388.2 ± 16.3 |    373.6 |    497.4 | 41.84 ± 6.68 |
| `zstd gcc.1 -c`   |    9.3 ± 1.4 |      8.1 |     15.9 |         1.00 |

`gcc.info`

| Command              |    Mean [ms] | Min [ms] | Max [ms] |     Relative |
| :------------------- | -----------: | -------: | -------: | -----------: |
| `pbzip2 gcc.info -c` |   72.8 ± 4.0 |     66.7 |     94.4 |  4.33 ± 0.30 |
| `pigz gcc.info -c`   |   22.8 ± 1.6 |     21.2 |     29.7 |  1.35 ± 0.11 |
| `xz gcc.info -c`     | 958.7 ± 14.8 |    942.3 |   1022.8 | 57.02 ± 2.49 |
| `zstd gcc.info -c`   |   16.8 ± 0.7 |     15.8 |     20.5 |         1.00 |

`homebrew-core.tar`

| Command                       |        Mean [s] | Min [s] | Max [s] |      Relative |
| :---------------------------- | --------------: | ------: | ------: | ------------: |
| `pbzip2 homebrew-core.tar -c` |  14.118 ± 0.470 |  13.102 |  14.658 |  13.29 ± 0.51 |
| `pigz homebrew-core.tar -c`   |   2.351 ± 0.028 |   2.325 |   2.414 |   2.21 ± 0.05 |
| `xz homebrew-core.tar -c`     | 139.036 ± 0.482 | 138.280 | 140.080 | 130.87 ± 2.58 |
| `zstd homebrew-core.tar -c`   |   1.062 ± 0.021 |   1.044 |   1.106 |          1.00 |

Perhaps testing `xz` wasn't the best idea.

`zero@753MB`

| Command                |       Mean [s] | Min [s] | Max [s] |     Relative |
| :--------------------- | -------------: | ------: | ------: | -----------: |
| `pbzip2 zero@753MB -c` |  1.195 ± 0.037 |   1.128 |   1.243 |  5.29 ± 0.33 |
| `pigz zero@753MB -c`   |  0.974 ± 0.046 |   0.938 |   1.093 |  4.32 ± 0.31 |
| `xz zero@753MB -c`     | 10.198 ± 0.125 |   9.943 |  10.344 | 45.18 ± 2.48 |
| `zstd zero@753MB -c`   |  0.226 ± 0.012 |   0.218 |   0.261 |         1.00 |

`zero@3.1GB`

| Command                |       Mean [s] | Min [s] | Max [s] |     Relative |
| :--------------------- | -------------: | ------: | ------: | -----------: |
| `pbzip2 zero@3.1GB -c` |  4.909 ± 0.152 |   4.630 |   5.073 |  5.58 ± 0.18 |
| `pigz zero@3.1GB -c`   |  4.042 ± 0.111 |   3.960 |   4.302 |  4.60 ± 0.13 |
| `xz zero@3.1GB -c`     | 41.282 ± 0.206 |  40.816 |  41.570 | 46.94 ± 0.49 |
| `zstd zero@3.1GB -c`   |  0.880 ± 0.008 |   0.865 |   0.889 |         1.00 |

#### Extraction

`ffmpeg-all.1`

| Command                       |  Mean [ms] | Min [ms] | Max [ms] |     Relative |
| :---------------------------- | ---------: | -------: | -------: | -----------: |
| `pbzip2 ffmpeg-all.1.bz2 -dc` | 41.3 ± 1.6 |     39.0 |     52.0 | 17.29 ± 6.13 |
| `pigz ffmpeg-all.1.gz -dc`    |  5.3 ± 0.1 |      5.1 |      5.6 |  2.20 ± 0.78 |
| `xz ffmpeg-all.1.xz -dc`      | 17.5 ± 0.6 |     16.1 |     19.4 |  7.33 ± 2.60 |
| `zstd ffmpeg-all.1.zst -dc`   |  2.4 ± 0.8 |      1.9 |      6.3 |         1.00 |

`gcc.1`

| Command                |  Mean [ms] | Min [ms] | Max [ms] |     Relative |
| :--------------------- | ---------: | -------: | -------: | -----------: |
| `pbzip2 gcc.1.bz2 -dc` | 36.4 ± 1.5 |     34.6 |     44.9 | 18.13 ± 1.71 |
| `pigz gcc.1.gz -dc`    |  5.0 ± 0.2 |      4.8 |      5.5 |  2.48 ± 0.23 |
| `xz gcc.1.xz -dc`      | 16.0 ± 0.6 |     14.9 |     18.1 |  7.96 ± 0.74 |
| `zstd gcc.1.zst -dc`   |  2.0 ± 0.2 |      1.7 |      2.6 |         1.00 |

`gcc.info`

| Command                   |  Mean [ms] | Min [ms] | Max [ms] |     Relative |
| :------------------------ | ---------: | -------: | -------: | -----------: |
| `pbzip2 gcc.info.bz2 -dc` | 71.4 ± 1.6 |     68.8 |     78.5 | 18.40 ± 1.42 |
| `pigz gcc.info.gz -dc`    |  9.4 ± 0.1 |      9.2 |     10.1 |  2.41 ± 0.18 |
| `xz gcc.info.xz -dc`      | 31.6 ± 0.8 |     29.8 |     33.3 |  8.16 ± 0.64 |
| `zstd gcc.info.zst -dc`   |  3.9 ± 0.3 |      3.6 |      5.9 |         1.00 |

`homebrew-core.tar`

| Command                            |       Mean [s] | Min [s] | Max [s] |      Relative |
| :--------------------------------- | -------------: | ------: | ------: | ------------: |
| `pbzip2 homebrew-core.tar.bz2 -dc` |  5.730 ± 0.073 |   5.629 |   5.850 |  42.85 ± 1.20 |
| `pigz homebrew-core.tar.gz -dc`    |  1.085 ± 0.004 |   1.081 |   1.096 |   8.12 ± 0.20 |
| `xz homebrew-core.tar.xz -dc`      | 18.779 ± 0.207 |  18.574 |  19.274 | 140.43 ± 3.83 |
| `zstd homebrew-core.tar.zst -dc`   |  0.134 ± 0.003 |   0.131 |   0.146 |          1.00 |

`zero@753MB`

| Command                     |      Mean [s] | Min [s] | Max [s] |     Relative |
| :-------------------------- | ------------: | ------: | ------: | -----------: |
| `pbzip2 zero@753MB.bz2 -dc` | 1.974 ± 0.009 |   1.957 |   1.985 | 20.56 ± 0.39 |
| `pigz zero@753MB.gz -dc`    | 0.823 ± 0.010 |   0.813 |   0.841 |  8.58 ± 0.19 |
| `xz zero@753MB.xz -dc`      | 1.717 ± 0.006 |   1.710 |   1.731 | 17.89 ± 0.33 |
| `zstd zero@753MB.zst -dc`   | 0.096 ± 0.002 |   0.093 |   0.100 |         1.00 |

`zero@3.1GB`

| Command                     |      Mean [s] | Min [s] | Max [s] |     Relative |
| :-------------------------- | ------------: | ------: | ------: | -----------: |
| `pbzip2 zero@3.1GB.bz2 -dc` | 8.021 ± 0.037 |   7.958 |   8.075 | 21.14 ± 0.13 |
| `pigz zero@3.1GB.gz -dc`    | 3.337 ± 0.023 |   3.302 |   3.373 |  8.80 ± 0.07 |
| `xz zero@3.1GB.xz -dc`      | 6.982 ± 0.025 |   6.948 |   7.020 | 18.41 ± 0.10 |
| `zstd zero@3.1GB.zst -dc`   | 0.379 ± 0.001 |   0.378 |   0.382 |         1.00 |

#### Results

The results here aren't quite what I was expecting.

Compression is pretty simple. `zstd` is the fastest, then `pigz`, then `pbzip2`, and then `xz` in a distant fourth.

As for extraction, `zstd` was once again the fastest, followed up by `pigz`.
But `xz` managed to outperform `pbzip2` in all but one test - which was also the same test where `pbzip2` outperformed `bzip2`.

## Conclusion

First of all, `zstd` was far and away the fastest compression tool.
There isn't any competition for that title.

`bzip2` was generally just strange.
It might warrant some further analysis, but I can't find many `.bz2` files on my computer so I don't really feel like it's needed.

`gzip` is bad. Use `pigz` instead.

And `xz` is great at compressing large amounts of data, so long as time isn't a concern.
