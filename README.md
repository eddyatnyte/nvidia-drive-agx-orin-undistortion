# Fisheye Undistortion with CUDA on Drive AGX Orin

The first thing you should do, is to get familiar with the properties of the GPU. The properties of the Nvidia Orin are:

```
Total Global Memory: 28953 MB
Multiprocessors: 16
Compute Capability: 8.7
CUDA Cores: 2048
Max Threads per Block: 1024
Max Blocks per Multiprocessor: 16
Max Threads per Multiprocessor: 1536
Max Grid Size: (2147483647, 65535, 65535)
```
It is important to first determine the block size. You need to ensure that the maximum number of threads per SM can be utilized. The dimensions of the block size should typically be powers of two. Additionally, a multiple of the block size should match the maximum number of threads per SM (here, 1536), but it should not exceed 16 blocks, as an SM is limited to 16 blocks.  
In this example, a block size of 32x16 was chosen. This results in 512 threads per block, which can run 3 times on an SM, perfectly utilizing all 1536 threads per SM.

The block size must be normalized to ensure that all pixels of the image are accounted for. It determines how many blocks in total need to be launched. While the earlier code section outlines the limits for these values, they can often be disregarded in this context since the required number of blocks will not approach the grid size limit. The grid size is calculated based on the image width and height (see [this Stack Overflow discussion](https://stackoverflow.com/questions/14739207/what-are-the-right-grid-and-block-dimensions-for-2d-triangular-smooth-in-cuda)).

In undistortion processes, it is common to use trigonometric functions, such as $\arctan(x)$, which are computationally expensive and can slow down the process. CUDA provides optimized math functions, such as ```atanf```, to mitigate this issue (see [CUDA Math API](https://docs.nvidia.com/cuda/cuda-math-api/cuda_math_api/group__CUDA__MATH__SINGLE.html?highlight=powf#_CPPv44powfff)).
