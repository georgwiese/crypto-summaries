Sources:
- [CUDA Programming Guide](https://docs.nvidia.com/cuda/cuda-programming-guide/index.html)
- [Stanford CS149 I Parallel Computing I 2023 I Lecture 7 - GPU architecture and CUDA Programming](https://www.youtube.com/watch?v=qQTDF0CBoxE)
- [CUDA Teaching Center (Josh Holloway)](https://www.youtube.com/playlist?list=PLC6u37oFvF40BAm7gwVP7uDdzmW83yHPe)

What is CUDA?
- **CUDA**: Compute Unified Device Architecture (introduced 2006)
- Programmed using **C++**, or higher-level DSLs, such as [Nvidia Warp](https://developer.nvidia.com/warp-python) or [OpenAI Triton](https://openai.com/index/triton/)
## Programming Model

### Threads, Blocks and Grids
- A programmer specifies a **kernel**, which is a function that is executed several times in different **threads**
- Threads are organized into **blocks**, which are organized into **grids**
- Grids and blocks are 1, 2, or 3 dimensional

**TLDR**: Thread $\in$ Block $\in$ Grid
### Syntax basics
- Kernels are C functions which return `void` and are annotated with the `__global__` specifier
- A common way to launch a kernel is via the **triple chevron notation**, which lets the program pass parameters such as the grid and block dimensions

**Example:**
```cpp
__global__ void matAdd(float* A, float* B, float* C, int width, int height)
{
    // 2D thread indices
    int col = threadIdx.x + blockDim.x * blockIdx.x;
    int row = threadIdx.y + blockDim.y * blockIdx.y;

    // Convert 2D index to 1D index
    int index = row * width + col;

    // Bounds check
    if (row < height && col < width) {
        // Actual computation
        C[index] = A[index] + B[index];
    }
}

int main()
{
    ...
    int width = 32;
    int height = 32;

    // Total elements = 1024 (32x32)
    
    // Block dimension: 16 x 16 (256 threads per block)
    dim3 threadsPerBlock(16, 16);
    // Grid dimension: 2 x 2 (4 blocks in the grid)
    dim3 numBlocks(width / 16, height / 16);

    matAdd<<<numBlocks, threadsPerBlock>>>(A, B, C, width, height);
    ...
}
```
### Execution in hardware
In hardware, the GPU consists of a collection of **Streaming Multiprocessors (SM)**: A collection of cores, with on-chip **registers** and **shared memory**
![[gpu_hardware.png]]

- All **thread blocks** are executed on a **single SM**
	- -> Threads of a block can use the fast **shared memory** and can easily **synchronize**
	- But, a SM might run multiple blocks at the same time, if resources suffice
- Different thread blocks might be executed in **any order**, or in parallel
- Similarly, thread blocks can be grouped into **clusters**. Clusters are executed on a **single GPC**. This gives additional opportunity to communicate and synchronize between threads.

The **occupancy** of a CUDA kernel is the ratio of the number of active warps to the number of active warps supported by the SM. It might go below 100% because of various hardware constraints, like the maximum number of blocks per SM, the shared memory size, etc.
### Warps and SIMT
Zooming in, each SM consists of several **sub-cores**. **32 threads** are grouped into **one warp**. All execution contexts (= register values) are stored on chips, with one register being the PC. Execution happens as follows:
- In each cycle, the sub-core selects a warp to advance
- All threads with the same PC are executed in a SIMD fashion
	- Nvidia calls this **Single-instruction multiple-threads** instead of SIMD, because which threads are run together is decided at runtime (=> those with the same PC)
	- If threads of a warp have different PCs, this is called **warp divergence** and leads to lower GPU utilization
- Any given instruction executes in **several cycles**, but different instructions (e.g. integer vs floating point instructions) can overlap

![[v100_sm.png]]

**Example: V100**
- 80 SMs
- 64 warps / SM
- => 5,120 warps and 163,840 threads on the device at once
- => 4 warps x 80 SMs x 32 threads / warp = 10,240 threads can be advanced per cycle
- 1.245 GHz clock speed
### Memories
Memory types by scope:
- **Thread-level**:
	- **Registers**: Very fast and on-chip. This is the default for local variables in kernels.
	- **Local memory**: For register spilling. Slow, because it is located on device DRAM.
- **Block-level**:
	- **Shared memory**: Accessible by all threads in the block. Fast on-chip memory. For variables declared as `__shared__`. It is also possible to dynamically allocate shared memory.
- **Program-level**:
	- **Global memory**: On device DRAM, with a connection to Host. For variables declared as `__device__`. Also managed via `cudaMalloc()`, `cudaMemset()`, `cudaMemCopy()`, or `cudaFree()`.
	- **Constant memory**: Similar to global memory, but aggressively cached and therefore fast. For variables declared as `__constant__`.
	- **Unified memory**: Similar to global memory, but accessible by the host directly. Declared as `__managed__`.

![[GPU_layout.png]]
![[memory_latencies.png]]
## Synchronization
### Between threads in a block
The `__syncthreads()` barrier in a kernel function achieves synchronization between threads in a block: All threads in a block (=> on the same SM) will execute until that point before any thread continues from there. For example, the implementation can ensure that the shared memory has been initialized properly before it is read.
### Between device and host
Kernel launches are asynchronous, i.e., the host function returns immediately and does not wait for the kernels to be executed completely.
The `cudaDeviceSynchronize()` blocks until all scheduled kernels have completed.
### CUDA streams
![[cuda_streams.png]]
A **CUDA stream** acts a **work queue** into which programs can add operations, such as memory copies or kernel launches, to be executed in order. If there are multiple streams, they are executed concurrently. Programmers can assign a priority to streams.

Once instantiated, a stream can be passed when launching a kernel using triple chevron notation:
`kernel<<<grid_dim, block_dim, shared_mem_size, stream>>>(...)`

**Synchronization**:
- The `cudaStreamSynchronize()` function blocks the host until all the work in the stream has been completed.
- **Events**: More fine-granular control
	- Events are **markers** that can be added to the stream, using `cudaEventRecord(event, stream)`
	- The host can block on events using `cudaEventSynchronize(event)`
## Memory Performance
### Coalesced global memory access
Global memory is accessed via **32-byte memory transactions**. If warps access **consecutive** memory regions, that is, accesses from many threads can be handled by fewer memory transactions.

For example, consider this matrix transpose kernel:
![[matrix_transpose_naive.png]]
The reads are perfectly coalesced, the writes are not.

This can be fixed by using shared memory:
![[matrix_transpose_shared_mem.png]]
### Shared memory bank conflicts
Shared memory has 32 banks that are organized such that successive 32-bit words map to successive banks. Each bank has a bandwidth of 32 bits per cycle.
When multiple threads in the same warp attempt to access different elements in the same bank, a **bank conflict** occurs.

This is the case in the example above:
- A warp has 32 threads
- When writing to `smemArray`, all threads in the same warp have the same value for `threadIdx.y`, accessing `smemArray` with a stride of 32 words
- => All accesses are in the same bank
- Note that when reading from `smemArray`, there are no bank conflicts

A common fix to this (according to the programming guide) is to allocate the shared memory one *larger* than needed:
`__shared__ float smemArray[THREADS_PER_BLOCK_X][THREADS_PER_BLOCK_Y + 1]`

## Practical Tips
- Set `CUDA_LOG_FILE` env variable to debug CUDA errors
- Kernel resource usage (e.g. the size of the shared memory) can be determined by passing `--resource-usage` to `nvcc`
- Profiling tools: [NSight Compute](https://developer.nvidia.com/nsight-compute) and [NSight Systems](https://developer.nvidia.com/nsight-systems)