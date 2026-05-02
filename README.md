# tiered-mem-traces

**60 memory access traces** for tiered-memory simulation, covering all 16
benchmark suites of the [CMU-SAFARI/DAMOV](https://github.com/CMU-SAFARI/DAMOV)
suite. Each trace preserves load/store/modify classification from
[Valgrind's lackey tool](https://valgrind.org/docs/manual/lk-manual.html).

## Format

Each trace is a gzipped text file with one memory operation per line:

```
S 1fff000198,8
S 1fff000190,8
S 1fff000188,8
L 04038e06,1
M 04038e10,4
```

- `L` = load, `S` = store, `M` = modify (read-modify-write)
- `<hex_addr>,<size>` = byte address and access size
- Instruction fetches are excluded; only data accesses are recorded

Total: 97 MB compressed (~4 GB uncompressed).

## Usage with arcsim

Extract just the addresses for `--raw-trace`:

```bash
zcat stream.Add.bytes.trace.gz | awk '{print $2}' | cut -d, -f1 | arcsim --raw-trace - --capacity 1024 --model cost --behavior 2q-ac-gd-dist-lin
```

Or simulate all traces:

```bash
for trace in *.bytes.trace.gz; do
    echo "=== $trace ==="
    zcat "$trace" | awk '{print $2}' | cut -d, -f1 | arcsim --raw-trace - --capacity 1024 --model cost --behavior 2q-ac-gd-dist-lin
done
```

## Traces (60 total, by suite)

| Trace | Description | Access Pattern | Size |
|-------|-------------|----------------|-----:|
| **bwa** (genomics) | | | |
| `bwa.AlignLite` | BWA-MEM short-read alignment via FM-index lookup | scattered reads into a hot index region | 817K |
| **chai** (heterogeneous) | | | |
| `chai.BFS` | frontier-driven graph BFS over a CSR graph | scattered adjacency reads, drifting hot frontier | 1.1M |
| `chai.BS.BEZIER_KERNEL` | 2D Bezier surface evaluation | structured streaming with small reused control set | 1.1M |
| `chai.CEDD.Gaussian` | 5x5 Gaussian-blur image convolution | stencil streaming, near-perfect spatial locality | 1.3M |
| `chai.CEDD.Sobel` | 3x3 Sobel edge-detection | stencil streaming | 1.3M |
| `chai.HSTI` | input-side histogram increment | stream-read + scattered hot-write into bins | 782K |
| `chai.OOPPAD.OOPPAD` | out-of-place 2D padding (border insertion) | streaming with reflected-border copy | 1.3M |
| `chai.SC.SC` | stream compaction = filter + prefix-sum + scatter | two streaming passes + scattered scatter | 753K |
| **darknet** (ML) | | | |
| `darknet.conv` | im2col-style 2D convolution layer | weight streaming + activation reuse | 1.1M |
| **gase** (genomics) | | | |
| `gase.Align` | banded Smith-Waterman DP cell update | streaming along the band, three-row reuse | 4.1M |
| **hardware_effects** (microbench) | | | |
| `hardware_effects.bandwidth` | page-stride memory walk | scattered, anti-prefetcher pattern | 2.5M |
| `hardware_effects.false_sharing` | single-threaded RMW alternating two cache-line slots | high temporal locality, RMW writes | 259K |
| **hashjoin** (database) | | | |
| `hashjoin.MPSM` | massively-parallel sort-merge join | two-stream merge with run-length copies | 2.8M |
| `hashjoin.NPO.HashJoin` | no-partition open-addressed hash join (build + probe) | sequential probe + hot hash-table writes | 1.1M |
| `hashjoin.NPO_st` | single-thread no-partition variant of NPO | sequential probe + scattered hash-table reads | 2.8M |
| `hashjoin.PRO` | partitioned radix-style hash join | phase change between partition-build and probe | 1.5M |
| **hpcc** (HPC) | | | |
| `hpcc.RandomAccess` | GUPS-style xor-shift updates to random table slots | uniform-random scattered RMW | 2.5M |
| **hpcg** (HPC) | | | |
| `hpcg.CG` | conjugate-gradient inner loop on a 7-pt sparse stencil | SpMV scattered reads + vector streaming reductions | 2.2M |
| `hpcg.MG` | 3-level multigrid V-cycle (smooth + restrict + prolong) | stencil sweep at multiple resolutions | 408K |
| `hpcg.SpMV` | sparse matrix-vector multiply on a 27-pt stencil | row-streaming + scattered neighbor reads | 1.2M |
| **ligra** (graph) | | | |
| `ligra.BC` | betweenness centrality via Brandes-style two-pass BFS | frontier sweep + dependency back-propagation | 2.7M |
| `ligra.BellmanFord` | single-source shortest paths via repeated edge relaxation | edge-stream sweep with dist updates per round | 3.1M |
| `ligra.BFS` | frontier-driven BFS | per-level frontier sweep, drifting hot set | 1.9M |
| `ligra.Components` | connected-components via min-label propagation | full edge-stream sweep per round, label updates | 2.4M |
| `ligra.KCore` | k-core decomposition by iterative low-degree peeling | vertex-array sweep with neighbor degree updates | 2.0M |
| `ligra.MIS` | maximal independent set via Luby-style randomized priority | neighbor-priority compare + alive removal | 1.9M |
| `ligra.PageRank` | PageRank: full-vertex sweep with adjacency-list reads | per-iter full-vertex stream + scattered neighbor writes | 2.0M |
| `ligra.Triangle` | triangle counting via per-edge neighbor-set intersection | nested adjacency-list scans, heavy reads | 520K |
| **parboil** (HPC) | | | |
| `parboil.histo` | image histogram with ~16k bins | streaming read + scattered bin RMW | 884K |
| `parboil.mri-q` | MRI Q-matrix construction (per-voxel trig accumulation) | double-loop over voxels x k-points, streaming | 1.5M |
| `parboil.sgemm` | naive single-precision dense matmul | triple-loop with strided B access | 1.1M |
| `parboil.spmv` | sparse matrix-vector multiply (random sparsity) | row-stream + irregular column reads | 815K |
| **parsec** (multimedia) | | | |
| `parsec.Streamcluster` | k-means-style center-distance scan | stream over points + sequential per-DIM | 1.1M |
| **phoenix** (mapreduce) | | | |
| `phoenix.histogram` | 256-bin histogram over a streaming byte input | streaming read + dense hot-write into 256 bins | 863K |
| `phoenix.kmeans` | k-means iterations: assign + center update | phase change between assign and reduce | 963K |
| `phoenix.lr` | linear-regression gradient descent | streaming over (x, y) pairs | 956K |
| `phoenix.matrix-mult` | MapReduce-flavored dense matmul | row-by-row mapper-style sweep | 1.4M |
| `phoenix.reverseindex` | build a doc->terms reverse index | scattered hash-style writes + linked-list traversals | 744K |
| `phoenix.WordCount` | MapReduce word count via open-addressed hash table | streaming over words + scattered hash probes | 823K |
| **polybench** (HPC) | | | |
| `polybench.3mm` | three sequential dense matmuls (E=AB; F=CD; G=EF) | tiled streaming with phase change between matmuls | 1.6M |
| `polybench.atax` | y = A^T * (A x) (transpose + matrix-vector twice) | two streaming sweeps over A with vector reads | 897K |
| `polybench.bicg` | biconjugate-gradient kernel: q=Ap; s=A^T r | single sweep over A with two simultaneous reductions | 854K |
| `polybench.doitgen` | 3D tensor reduction | structured streaming with reduction | 1.2M |
| `polybench.gemm` | naive dense matmul C = A * B + C | triple-loop with strided B access | 1.4M |
| `polybench.syr2k` | symmetric rank-2 update C = AB^T + BA^T + C | triple-loop with shared inner-K accumulation | 1.6M |
| **rodinia** (heterogeneous) | | | |
| `rodinia.bfs` | BFS on a regular 4-connected 2D grid | regular per-level frontier sweep | 1.7M |
| `rodinia.cfd` | Euler-equation flux update on an unstructured mesh | edge-stream with scatter to two endpoints | 3.9M |
| `rodinia.hotspot` | 2D thermal-stencil sweep with two-array swap | 5-pt stencil, two-array swap per iter | 1.4M |
| `rodinia.lavamd` | short-range MD pairwise force calc with cutoff | O(N^2) pairwise with cutoff rejection | 1.4M |
| `rodinia.lud` | right-looking LU decomposition without pivoting | triangular sweep with shrinking trailing block | 1.6M |
| `rodinia.srad` | speckle-reducing anisotropic-diffusion image filter | two-pass per-pixel stencil | 1.8M |
| **splash2** (HPC) | | | |
| `splash2.FFT` | 1D FFT (bit-reversal + butterfly stages) | bit-reversal scatter + log(N) butterfly passes | 2.2M |
| `splash2.FMM` | O(N^2) particle pairwise interactions | O(N^2) double-loop with sqrt per pair | 966K |
| `splash2.OCEAN` | 2D 5-pt stencil sweep with two interleaved grids | stencil streaming with three-grid update | 1.9M |
| `splash2.RADIOSITY` | hierarchical 2D iterative form-factor refinement | KxK gather kernel applied per cell | 1.3M |
| `splash2.RAYTRACE` | ray casting through a small scene of spheres | per-pixel scene scan, frequent shared-data reads | 895K |
| **stream** (microbench) | | | |
| `stream.Add` | c[i] = a[i] + b[i] | perfectly sequential streaming, no reuse | 3.5M |
| `stream.Copy` | c[i] = a[i] | perfectly sequential streaming, no reuse | 3.5M |
| `stream.Scale` | b[i] = scalar * c[i] | perfectly sequential streaming, no reuse | 3.5M |
| `stream.Triad` | a[i] = b[i] + scalar * c[i] | perfectly sequential streaming, no reuse | 3.5M |

## How Traces Were Collected

Each trace comes from a small self-contained C facsimile that reproduces the
kernel's memory access pattern at smaller-than-production scale. The pipeline:

```
DAMOV recipe -> C facsimile (gcc -O0)
             -> valgrind --tool=lackey --trace-mem=yes
             -> awk filter (L/S/M lines only)
             -> gzip -6
             -> *.bytes.trace.gz
```

These are **pre-cache** traces (Valgrind sees every load/store including stack
traffic). Working sets are small (all 60 traces have U <= 3091 4-KiB pages).
The qualitative access pattern shape (skew, reuse-distance distribution,
hot-set drift, spatial locality class) is preserved from the original DAMOV
kernels, but absolute access counts differ from production-scale runs.

13 long-running workloads were SIGKILL-capped at ~25 min wall time once the
trace had grown to 25-200M accesses.

## Suite Coverage

| Suite | DAMOV Functions | Traces Here |
|-------|----------------:|------------:|
| bwa | 2 | 1 |
| chai | 19 | 7 |
| darknet | 4 | 1 |
| gase | 2 | 1 |
| hardware_effects | 4 | 2 |
| hashjoin | 12 | 4 |
| hpcc | 2 | 1 |
| hpcg | 5 | 3 |
| ligra | 39 | 8 |
| parboil | 7 | 4 |
| parsec | 5 | 1 |
| phoenix | 5 | 6 |
| polybench | 8 | 6 |
| rodinia | 11 | 6 |
| splash2 | 14 | 5 |
| stream | 4 | 4 |
| **Total** | **143** | **60** |

## Attribution

These traces are derived from the **DAMOV** benchmark suite by Oliveira et al.
(CMU-SAFARI). The kernels were re-implemented at small scale as self-contained
C facsimiles; the intent and naming of each trace follows the original DAMOV
classification.

> Geraldo F. Oliveira, Juan Gomez-Luna, Lois Orosa, Saugata Ghose,
> Nandita Vijaykumar, Ivan Fernandez, Mohammad Sadrosadati, Onur Mutlu.
> "DAMOV: A New Methodology and Benchmark Suite for Evaluating Data
> Movement Bottlenecks." IEEE Access, 2021. arXiv:2105.03725
>
> https://github.com/CMU-SAFARI/DAMOV

DAMOV aggregates kernels from BWA, Chai, Darknet, GASE, Hardware-Effects,
HPC-Challenge, HPCG, Hashjoin, Ligra, PARSEC, Parboil, PolyBench, Phoenix,
Rodinia, SPLASH-2, and STREAM -- each licensed and credited by its upstream
authors.
