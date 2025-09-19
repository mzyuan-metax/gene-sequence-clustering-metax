# Build on HPCC

This is a guide to build gene-sequence-clustering on HPCC.

- HPCC: 2.33.0.x
- OS  : ubuntu22.04

## 1. Init Envs

```
export HPCC_PATH="/opt/hpcc"
export CUDA_PATH=/usr/local/cuda
export CUCC_PATH=${HPCC_PATH}/tools/cu-bridge
export PATH=${HPCC_PATH}/ompi/include:${CUDA_PATH}/bin:${HPCC_PATH}/htgpu_llvm/bin:${HPCC_PATH}/bin:${CUCC_PATH}/tools:${CUCC_PATH}/bin:${PATH}
export LD_LIBRARY_PATH=${HPCC_PATH}/lib:${HPCC_PATH}/ompi/lib:${HPCC_PATH}/htgpu_llvm/lib:${LD_LIBRARY_PATH}
```

## 2. Build and init DB

**Build** 

```
$ cd cuda/makeDB/
$ make_hpcc
```

**Init**

```
$ ./makeDB -f ../../data/gene.fasta -p ../../data/gene.packed -t 0
fasta:          ../../data/gene.fasta
packed:         ../../data/gene.packed
type:           0 (0:gene 1:protein)
. 
find 25000 sequences
longest:        1587
shortest:       1251
. 
pack 25000 sequences
total:          0.868975 seconds.
Fri Sep 19 12:21:24 2025
```

## 3. Build and run cluster

**Edit code**

- `cuda/cluster/func.cu` 140 lines 

```
// kernel_dynamicGen 动态规划
template<int32_t entropy>
__global__ void kernel_dynamic(uint32_t **reads, uint32_t *represent,
const float similarity, int32_t *jobs, const int32_t jobCount){
...
  // uint32_t line[2048];  // 每行结果
  uint32_t line[512];  // 每行结果
...
```

- `cuda/cluster/Makefile`

```
all:
	nvcc -O3 -arch=sm_80 -I/opt/hpcc/ompi/include/ -L/opt/hpcc/ompi/lib -lmpi main.cpp func.cu -o cluster
```

**Build** 

```
$ cd cuda/cluster/
$ make_hpcc
````

**Run**

```
$ mpirun -n 1 --allow-run-as-root  ./cluster -p ../../data/gene.packed -r ../../data/result.txt -s 0.95 -m 0 
packed:         ../../data/gene.packed
result:         ../../data/result.txt
similarlity:    0.95
mode:           0 (0:fast 1:precise)
reads:          25000
longest:        1587
shortest:       1251
type:           0 (0:gene 1:protein)
kLength:        14
25000
cluster:        2990
12.2869 seconds.
--------------------------------------------------------------------------
mpirun has exited due to process rank 0 with PID 0 on
node AI-141 exiting improperly. There are three reasons this could occur:
```

