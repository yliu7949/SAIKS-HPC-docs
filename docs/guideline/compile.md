---
icon: material/file-code
---

# 代码编译 {#compile}

## 编译 CUDA 代码

在 GPU 集群上开发高性能计算应用时，CUDA 是最常用的编程框架之一。本文将详细介绍如何在 GPU 集群环境中编译 `.cu` 代码文件。

### 环境准备

首先需要确保在登录节点已加载正确的 CUDA 环境：

```shell
module load cuda/13.0
nvcc --version
```

### 使用 NVCC 编译器

CUDA 代码使用 `nvcc` 编译器进行编译，基本语法为：

```shell
nvcc [选项] <输入文件> -o <输出文件>
```

对于一个向量加法示例，编译命令为：

```shell
nvcc vector_add.cu -o vector_add -O3
```

??? note "示例代码文件：vector_add.cu"

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <stdbool.h>
    #include <cuda_runtime.h>
    
    #define N 1000000
    #define THREADS_PER_BLOCK 256
    
    #define CHECK_CUDA(call) { \
        cudaError_t err = call; \
        if (err != cudaSuccess) { \
            fprintf(stderr, "CUDA error at %s:%d: %s\n", \
                    __FILE__, __LINE__, cudaGetErrorString(err)); \
            exit(EXIT_FAILURE); \
        } \
    }
    
    __global__ void vectorAdd(int *a, int *b, int *c) {
        int idx = blockIdx.x * blockDim.x + threadIdx.x;
        if (idx < N) {
            c[idx] = a[idx] + b[idx];
        }
    }
    
    int main() {
        int *a, *b, *c;
        int *d_a, *d_b, *d_c;
    
        // 获取 GPU 设备数量和信息
        int deviceCount;
        CHECK_CUDA(cudaGetDeviceCount(&deviceCount));
        printf("可用 GPU 数量: %d\n", deviceCount);
    
        int device;
        CHECK_CUDA(cudaGetDevice(&device));
        cudaDeviceProp prop;
        CHECK_CUDA(cudaGetDeviceProperties(&prop, device));
        printf("当前使用的 GPU[%d]: %s\n", device, prop.name);
        printf("SM 数量: %d, 显存: %.2f GB\n", prop.multiProcessorCount,
               prop.totalGlobalMem / (1024.0 * 1024.0 * 1024.0));
    
        // 主机端内存分配
        a = (int*)malloc(N * sizeof(int));
        b = (int*)malloc(N * sizeof(int));
        c = (int*)malloc(N * sizeof(int));
    
        for (int i = 0; i < N; i++) {
            a[i] = rand() % 100;
            b[i] = rand() % 100;
        }
    
        CHECK_CUDA(cudaMalloc((void **)&d_a, N * sizeof(int)));
        CHECK_CUDA(cudaMalloc((void **)&d_b, N * sizeof(int)));
        CHECK_CUDA(cudaMalloc((void **)&d_c, N * sizeof(int)));
    
        CHECK_CUDA(cudaMemcpy(d_a, a, N * sizeof(int), cudaMemcpyHostToDevice));
        CHECK_CUDA(cudaMemcpy(d_b, b, N * sizeof(int), cudaMemcpyHostToDevice));
    
        int blocks = (N + THREADS_PER_BLOCK - 1) / THREADS_PER_BLOCK;
    
        // CUDA Event 用于计时 kernel 执行时间
        cudaEvent_t start, stop;
        CHECK_CUDA(cudaEventCreate(&start));
        CHECK_CUDA(cudaEventCreate(&stop));
    
        CHECK_CUDA(cudaEventRecord(start, 0));
        vectorAdd<<<blocks, THREADS_PER_BLOCK>>>(d_a, d_b, d_c);
        CHECK_CUDA(cudaEventRecord(stop, 0));
    
        CHECK_CUDA(cudaGetLastError());
        CHECK_CUDA(cudaEventSynchronize(stop));
    
        float kernelTime = 0;
        CHECK_CUDA(cudaEventElapsedTime(&kernelTime, start, stop));
    
        CHECK_CUDA(cudaMemcpy(c, d_c, N * sizeof(int), cudaMemcpyDeviceToHost));
    
        bool success = true;
        for (int i = 0; i < N; i++) {
            if (c[i] != a[i] + b[i]) {
                printf("验证失败! 索引 %d: %d + %d != %d\n", i, a[i], b[i], c[i]);
                success = false;
                break;
            }
        }
    
        free(a); free(b); free(c);
        cudaFree(d_a); cudaFree(d_b); cudaFree(d_c);
    
        CHECK_CUDA(cudaEventDestroy(start));
        CHECK_CUDA(cudaEventDestroy(stop));
    
        if (success) {
            printf("Test PASSED: 成功完成 %d 元素向量加法\n", N);
            printf("Kernel 执行时间: %.3f ms\n", kernelTime);
            printf("吞吐率: %.2f Million elements/sec\n", N / (kernelTime * 1e3));
            return 0;
        } else {
            printf("Test FAILED\n");
            return 1;
        }
    }
    ```
    
也可以写成一个编译脚本：

```shell
#!/bin/bash
# 加载 CUDA 模块
module load cuda/13.0

# 编译 CUDA 代码
nvcc vector_add.cu -o vector_add -O3
```

得到 `vector_add` 二进制文件。
