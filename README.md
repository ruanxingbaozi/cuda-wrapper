# CUDA wrapper

## Goal

Many deep-learning researchers are not aware of how many resources they are consuming very well. But all will go wrong when there's no free GPU memory. It's necessary to limit GPU memory usage per container to share the GPU across the containers.

## Approach

This work wraps the NVIDIA driver API by `dlsym` and captures every alloc and free usage by record the size. It will reads ENV to set the quota dynamically. When the app hits the quota, we will return a `CUDA_ERROR_OUT_OF_MEMORY` error to the last alloc function. Although each deep-learning framework handles the error in different ways, it works on tensorflow-based deep-learning jobs.

We record every alloc function like `cuMemAlloc` with both their pointer, the `CUdeviceptr` and size. When there's a `cuMemfree`, we use the given pointer to find out how much memory it has alloced. We use `pthread_mutex_lock` to avoid race condition.

## Usage

### Compile

```bash

gcc -I /usr/local/cuda/include/ cuda-wrapper2.c -fPIC -shared -ldl -lcuda -o ./release/libcuda2.so

```

### Use

```bash

LD_PRELOAD=/path/to/libcuda2.so python test.py

```


```bash

LD_PRELOAD=/path/to/libcuda2.so WRAPPER_MAX_MEMORY=4217928960 python test.py

```