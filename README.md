# tiered-mem-traces

Memory access traces for tiered-memory simulation, collected from [DAMOV](https://github.com/CMU-SAFARI/DAMOV) workloads using Valgrind's lackey tool.

## Format

Each trace is a gzipped text file containing one bare hexadecimal byte address per line:

```
ff000188
ff000180
ff000178
ff000170
ff000168
```

These are raw virtual addresses captured from data loads and stores (no instruction fetches).

## Traces

| Trace | Workload | Compressed Size |
|-------|----------|-----------------|
| `bwa.AlignLite.tmsim.trace.gz` | BWA sequence alignment (AlignLite) | 4.2 MB |
| `chai.BFS.tmsim.trace.gz` | CHAi breadth-first search | 998 KB |
| `darknet.conv.tmsim.trace.gz` | Darknet convolution layer | 8.5 MB |
| `hashjoin.NPO.HashJoin.tmsim.trace.gz` | Hash join (NPO) | 29 MB |
| `hpcc.RandomAccess.tmsim.trace.gz` | HPCC RandomAccess | 24 MB |
| `phoenix.kmeans.tmsim.trace.gz` | Phoenix k-means clustering | 8.2 MB |
| `polybench.3mm.tmsim.trace.gz` | PolyBench 3mm matrix multiply | 12 MB |
| `polybench.gemm.tmsim.trace.gz` | PolyBench GEMM | 9.0 MB |
| `splash2.FFT.tmsim.trace.gz` | SPLASH-2 FFT | 1.8 MB |
| `stream.Add.tmsim.trace.gz` | STREAM Add | 7.6 MB |
| `stream.Copy.tmsim.trace.gz` | STREAM Copy | 6.6 MB |
| `stream.Scale.tmsim.trace.gz` | STREAM Scale | 6.6 MB |
| `stream.Triad.tmsim.trace.gz` | STREAM Triad | 7.8 MB |

## Usage with arcsim

```bash
zcat stream.Add.tmsim.trace.gz | arcsim --raw-trace - --capacity 1024 --model cost --behavior 2q-ac-gd-dist-lin
```

Or simulate multiple traces:

```bash
for trace in *.trace.gz; do
    echo "=== $trace ==="
    zcat "$trace" | arcsim --raw-trace - --capacity 1024 --model cost --behavior 2q-ac-gd-dist-lin
done
```

## How Traces Were Collected

Traces were collected using [Valgrind's lackey tool](https://valgrind.org/docs/manual/lk-manual.html) with `--trace-mem=yes`, which records every data load (L), store (S), and modify (M) operation. Instruction fetches are excluded. The raw Valgrind output was filtered to extract bare hex addresses of data accesses.

The workloads come from the [DAMOV benchmark suite](https://github.com/CMU-SAFARI/DAMOV) (Data Movement Analysis of Workloads), which characterizes data movement bottlenecks across a diverse set of applications.

## Attribution

The DAMOV benchmark suite is developed by [CMU-SAFARI](https://github.com/CMU-SAFARI). If you use these traces in your research, please cite:

> Geraldo F. Oliveira, Juan Gómez-Luna, Lois Orosa, Saugata Ghose, Nandita Vijaykumar, Ivan Fernandez, Mohammad Sadrosadati, Onur Mutlu. "DAMOV: A New Methodology and Benchmark Suite for Evaluating Data Movement Bottlenecks." IEEE Access, 2021.
