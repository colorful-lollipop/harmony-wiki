# 在 OpenHarmony 中的使用

## 4.1 使用方式概述

### 4.1.1 两种使用模式

OpenCL-Headers 在 OpenHarmony 中支持两种使用模式：

| 模式 | 头文件 | 链接 | 适用场景 |
|------|--------|------|----------|
| 标准模式 | `<CL/opencl.h>` | `libcl` | 标准 OpenCL 应用 |
| 包装器模式 | `<opencl_wrapper.h>` | `libcl` | 需要动态加载控制的应用 |

### 4.1.2 推荐使用方式

**对于大多数应用**：推荐使用标准模式，直接使用 `<CL/opencl.h>` 头文件即可。

**标准模式的优势**：

- API 接口与上游完全一致
- 无需了解底层动态加载细节
- 自动初始化 OpenCL 环境

**包装器模式**：适用于需要精细控制 OpenCL 库加载的场景。

## 4.2 依赖配置

### 4.2.1 添加依赖

在模块的 BUILD.gn 文件中添加依赖：

```gn
ohos_executable("my_opencl_app") {
  sources = [
    "main.cpp",
  ]

  deps = [
    "//third_party/opencl-headers:libcl",
  ]
}
```

### 4.2.2 完整依赖配置示例

```gn
import("//build/ohos.gni")

# 创建一个使用 OpenCL 的可执行程序
ohos_executable("opencl_example") {
  # 源文件
  sources = [
    "opencl_example.cpp",
    "ocl_utils.cpp",
  ]

  # 依赖配置
  deps = [
    "//third_party/opencl-headers:libcl",
  ]

  # 编译器选项
  cflags = [
    "-Wall",
    "-Wextra",
    "-O2",
  ]

  # 包含路径（通常不需要，会自动从 deps 传递）
  include_dirs = [
    "//third_party/opencl-headers/opencl-headers-CL",
  ]
}
```

### 4.2.3 头文件导出配置

opencl-headers 通过 `public_configs` 导出公共头文件路径：

```gn
config("cl_public_config") {
  include_dirs = [
    "opencl-headers-CL",
    "include",
  ]
}

ohos_shared_library("libcl") {
  # ...
  public_configs = [ ":cl_public_config" ]
}
```

这意味着当模块依赖 `libcl` 时，头文件路径会自动添加到编译选项中。

## 4.3 代码集成

### 4.3.1 基础使用

**基本结构**：

```cpp
#include <CL/opencl.h>
#include <iostream>

int main() {
    // 1. 获取平台信息
    cl_uint platformCount = 0;
    clGetPlatformIDs(0, nullptr, &platformCount);

    if (platformCount == 0) {
        std::cerr << "No OpenCL platform found" << std::endl;
        return -1;
    }

    std::cout << "Found " << platformCount << " platform(s)" << std::endl;

    // 2. 获取设备信息
    cl_platform_id platform = nullptr;
    clGetPlatformIDs(1, &platform, nullptr);

    cl_uint deviceCount = 0;
    clGetDeviceIDs(platform, CL_DEVICE_TYPE_ALL, 0, nullptr, &deviceCount);

    std::cout << "Found " << deviceCount << " device(s)" << std::endl;

    return 0;
}
```

### 4.3.2 完整 OpenCL 示例

**完整的 OpenCL 应用程序示例**：

```cpp
#include <CL/opencl.h>
#include <iostream>
#include <fstream>
#include <vector>
#include <string>

#define CHECK_ERROR(err, msg) \
    if (err != CL_SUCCESS) { \
        std::cerr << "OpenCL Error: " << msg << " (code: " << err << ")" << std::endl; \
        return -1; \
    }

// 简单的向量加法内核
const char* kernelSource = R"(
__kernel void vector_add(__global const int* a,
                         __global const int* b,
                         __global int* c,
                         const int n) {
    int gid = get_global_id(0);
    if (gid < n) {
        c[gid] = a[gid] + b[gid];
    }
}
)";

int main() {
    cl_int err;

    // 1. 获取平台
    cl_uint platformCount = 0;
    err = clGetPlatformIDs(0, nullptr, &platformCount);
    CHECK_ERROR(err, "clGetPlatformIDs");

    if (platformCount == 0) {
        std::cerr << "No OpenCL platform found" << std::endl;
        return -1;
    }

    std::cout << "Found " << platformCount << " platform(s)" << std::endl;

    cl_platform_id platform = nullptr;
    err = clGetPlatformIDs(1, &platform, nullptr);
    CHECK_ERROR(err, "clGetPlatformIDs");

    // 2. 获取设备
    cl_uint deviceCount = 0;
    err = clGetDeviceIDs(platform, CL_DEVICE_TYPE_ALL, 0, nullptr, &deviceCount);
    CHECK_ERROR(err, "clGetDeviceIDs");

    if (deviceCount == 0) {
        std::cerr << "No OpenCL device found" << std::endl;
        return -1;
    }

    std::cout << "Found " << deviceCount << " device(s)" << std::endl;

    cl_device_id device = nullptr;
    err = clGetDeviceIDs(platform, CL_DEVICE_TYPE_ALL, 1, &device, nullptr);
    CHECK_ERROR(err, "clGetDeviceIDs");

    // 3. 创建上下文
    cl_context context = clCreateContext(nullptr, 1, &device, nullptr, nullptr, &err);
    CHECK_ERROR(err, "clCreateContext");

    // 4. 创建命令队列
    cl_command_queue queue = clCreateCommandQueue(context, device, 0, &err);
    CHECK_ERROR(err, "clCreateCommandQueue");

    // 5. 创建程序
    cl_program program = clCreateProgramWithSource(context, 1, &kernelSource, nullptr, &err);
    CHECK_ERROR(err, "clCreateProgramWithSource");

    // 6. 构建程序
    err = clBuildProgram(program, 1, &device, nullptr, nullptr, nullptr);
    CHECK_ERROR(err, "clBuildProgram");

    // 7. 创建内核
    cl_kernel kernel = clCreateKernel(program, "vector_add", &err);
    CHECK_ERROR(err, "clCreateKernel");

    // 8. 准备数据
    const int size = 1024;
    std::vector<int> a(size), b(size), c(size);
    for (int i = 0; i < size; i++) {
        a[i] = i;
        b[i] = size - i;
    }

    // 创建缓冲区
    cl_mem bufA = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                                 sizeof(cl_int) * size, a.data(), &err);
    CHECK_ERROR(err, "clCreateBuffer (A)");

    cl_mem bufB = clCreateBuffer(context, CL_MEM_READ_ONLY | CL_MEM_COPY_HOST_PTR,
                                 sizeof(cl_int) * size, b.data(), &err);
    CHECK_ERROR(err, "clCreateBuffer (B)");

    cl_mem bufC = clCreateBuffer(context, CL_MEM_WRITE_ONLY,
                                 sizeof(cl_int) * size, nullptr, &err);
    CHECK_ERROR(err, "clCreateBuffer (C)");

    // 9. 设置内核参数
    err = clSetKernelArg(kernel, 0, sizeof(cl_mem), &bufA);
    CHECK_ERROR(err, "clSetKernelArg (0)");
    err = clSetKernelArg(kernel, 1, sizeof(cl_mem), &bufB);
    CHECK_ERROR(err, "clSetKernelArg (1)");
    err = clSetKernelArg(kernel, 2, sizeof(cl_mem), &bufC);
    CHECK_ERROR(err, "clSetKernelArg (2)");
    err = clSetKernelArg(kernel, 3, sizeof(cl_int), &size);
    CHECK_ERROR(err, "clSetKernelArg (3)");

    // 10. 执行内核
    size_t globalSize = size;
    err = clEnqueueNDRangeKernel(queue, kernel, 1, nullptr, &globalSize, nullptr, 0, nullptr, nullptr);
    CHECK_ERROR(err, "clEnqueueNDRangeKernel");

    // 11. 读取结果
    err = clEnqueueReadBuffer(queue, bufC, CL_TRUE, 0, sizeof(cl_int) * size, c.data(), 0, nullptr, nullptr);
    CHECK_ERROR(err, "clEnqueueReadBuffer");

    // 验证结果
    bool success = true;
    for (int i = 0; i < size; i++) {
        if (c[i] != a[i] + b[i]) {
            std::cerr << "Error at index " << i << ": " << c[i] << " != " << a[i] + b[i] << std::endl;
            success = false;
            break;
        }
    }

    if (success) {
        std::cout << "Vector addition successful!" << std::endl;
    }

    // 12. 清理资源
    clReleaseMemObject(bufA);
    clReleaseMemObject(bufB);
    clReleaseMemObject(bufC);
    clReleaseKernel(kernel);
    clReleaseProgram(program);
    clReleaseCommandQueue(queue);
    clReleaseContext(context);

    return success ? 0 : -1;
}
```

### 4.3.3 错误处理

**错误码检查**：

```cpp
#include <CL/cl.h>

void checkError(cl_int err, const char* file, int line) {
    if (err != CL_SUCCESS) {
        std::cerr << "OpenCL error at " << file << ":" << line
                  << " (code: " << err << ")" << std::endl;
        // 根据错误码进行处理
        switch (err) {
            case CL_DEVICE_NOT_FOUND:
                std::cerr << "Error: Device not found" << std::endl;
                break;
            case CL_OUT_OF_RESOURCES:
                std::cerr << "Error: Out of resources" << std::endl;
                break;
            case CL_OUT_OF_HOST_MEMORY:
                std::cerr << "Error: Out of host memory" << std::endl;
                break;
            default:
                std::cerr << "Error: Unknown error" << std::endl;
                break;
        }
    }
}

#define CHECK_ERROR(err) checkError(err, __FILE__, __LINE__)
```

## 4.4 OH 特有功能使用

### 4.4.1 使用包装器模式

**启用包装器模式**：

```cpp
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>
```

**包装器模式提供的能力**：

1. **手动初始化**：

```cpp
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>

int main() {
    // 手动初始化 OpenCL
    if (!OHOS::InitOpenCL()) {
        std::cerr << "Failed to initialize OpenCL" << std::endl;
        return -1;
    }

    // 使用标准 OpenCL API
    cl_platform_id platform = nullptr;
    clGetPlatformIDs(1, &platform, nullptr);

    return 0;
}
```

2. **外部库加载**：

```cpp
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>

int main() {
    void* handle = nullptr;

    // 尝试加载特定路径的 OpenCL 库
    if (!OHOS::InitOpenCLExtern(&handle)) {
        std::cerr << "Failed to load OpenCL library" << std::endl;
        return -1;
    }

    // 使用 OpenCL API...

    // 完成后卸载
    if (!OHOS::UnLoadCLExtern(handle)) {
        std::cerr << "Warning: Failed to unload OpenCL library" << std::endl;
    }

    return 0;
}
```

### 4.4.2 获取函数指针

**获取扩展函数指针**：

```cpp
#define USE_OPENCL_WRAPPER
#include <opencl_wrapper.h>

// 获取扩展函数指针
clGetExtensionFunctionAddressFunc extFunc = OHOS::clGetExtensionFunctionAddress;
if (extFunc) {
    // 使用扩展函数
}
```

## 4.5 使用场景示例

### 4.5.1 图像处理

**图像模糊滤镜示例**：

```cpp
#include <CL/opencl.h>
#include <vector>
#include <iostream>

// 高斯模糊内核
const char* blurKernel = R"(
__kernel void gaussian_blur(__global const uchar4* input,
                            __global uchar4* output,
                            const int width,
                            const int height,
                            __constant float* kernel,
                            const int kernelSize) {
    int x = get_global_id(0);
    int y = get_global_id(1);

    if (x >= width || y >= height) return;

    float4 sum = (float4)(0, 0, 0, 0);
    float weightSum = 0.0f;

    int halfSize = kernelSize / 2;
    for (int ky = -halfSize; ky <= halfSize; ky++) {
        for (int kx = -halfSize; kx <= halfSize; kx++) {
            int px = clamp(x + kx, 0, width - 1);
            int py = clamp(y + ky, 0, height - 1);

            float weight = kernel[(ky + halfSize) * kernelSize + (kx + halfSize)];
            sum += convert_float4(input[py * width + px]) * weight;
            weightSum += weight;
        }
    }

    output[y * width + x] = convert_uchar4(sum / weightSum);
}
)";

int gaussianBlur(cl_uchar4* input, cl_uchar4* output, int width, int height) {
    // ... 创建上下文、队列、程序、内核的代码省略

    // 设置参数并执行
    // ...

    return 0;
}
```

### 4.5.2 并行计算

**矩阵乘法示例**：

```cpp
#include <CL/opencl.h>

const char* matmulKernel = R"(
__kernel void matmul(__global const float* A,
                     __global const float* B,
                     __global float* C,
                     const int N) {
    int row = get_global_id(0);
    int col = get_global_id(1);

    if (row < N && col < N) {
        float sum = 0.0f;
        for (int k = 0; k < N; k++) {
            sum += A[row * N + k] * B[k * N + col];
        }
        C[row * N + col] = sum;
    }
}
)";

int matmul(float* A, float* B, float* C, int N) {
    // ... 实现代码
    return 0;
}
```

## 4.6 常见问题处理

### 4.6.1 平台或设备不可用

**问题**：找不到 OpenCL 平台或设备

**解决方案**：

```cpp
cl_uint platformCount = 0;
clGetPlatformIDs(0, nullptr, &platformCount);

if (platformCount == 0) {
    std::cerr << "No OpenCL platform found" << std::endl;
    std::cerr << "Possible reasons:" << std::endl;
    std::cerr << "1. No OpenCL driver installed" << std::endl;
    std::cerr << "2. OpenCL driver not compatible" << std::endl;
    std::cerr << "3. Device does not support OpenCL" << std::endl;
    return -1;
}
```

### 4.6.2 程序构建失败

**问题**：`clBuildProgram` 返回错误

**解决方案**：

```cpp
cl_program program = clCreateProgramWithSource(context, 1, &source, nullptr, &err);
if (err != CL_SUCCESS) {
    std::cerr << "Failed to create program" << std::endl;
    return -1;
}

err = clBuildProgram(program, 1, &device, nullptr, nullptr, nullptr);
if (err != CL_SUCCESS) {
    // 获取构建日志
    size_t logSize;
    clGetProgramBuildInfo(program, device, CL_PROGRAM_BUILD_LOG, 0, nullptr, &logSize);

    std::vector<char> buildLog(logSize);
    clGetProgramBuildInfo(program, device, CL_PROGRAM_BUILD_LOG,
                          logSize, buildLog.data(), nullptr);

    std::cerr << "Build error:" << std::endl;
    std::cerr << buildLog.data() << std::endl;
    return -1;
}
```

### 4.6.3 内核执行错误

**问题**：内核执行失败

**解决方案**：

```cpp
size_t globalSize[] = {width, height};
err = clEnqueueNDRangeKernel(queue, kernel, 2, nullptr, globalSize, nullptr, 0, nullptr, nullptr);

if (err != CL_SUCCESS) {
    std::cerr << "Kernel execution failed with error: " << err << std::endl;
    return -1;
}

// 等待执行完成
err = clFinish(queue);
if (err != CL_SUCCESS) {
    std::cerr << "clFinish failed: " << err << std::endl;
    return -1;
}
```

## 4.7 资源管理

### 4.7.1 清理顺序

**正确的清理顺序**：

```cpp
// 1. 释放内存对象
clReleaseMemObject(bufferA);
clReleaseMemObject(bufferB);
clReleaseMemObject(bufferC);

// 2. 释放内核
clReleaseKernel(kernel);

// 3. 释放程序
clReleaseProgram(program);

// 4. 释放命令队列
clReleaseCommandQueue(queue);

// 5. 释放上下文
clReleaseContext(context);
```

### 4.7.2 使用 RAII 封装

**推荐：使用 RAII 管理资源**：

```cpp
class OpenCLContext {
public:
    OpenCLContext() : context_(nullptr), queue_(nullptr) {}

    bool init() {
        cl_int err;

        // 获取平台
        cl_uint platformCount = 0;
        err = clGetPlatformIDs(0, nullptr, &platformCount);
        if (platformCount == 0) return false;

        cl_platform_id platform = nullptr;
        err = clGetPlatformIDs(1, &platform, nullptr);

        // 获取设备
        cl_device_id device = nullptr;
        err = clGetDeviceIDs(platform, CL_DEVICE_TYPE_ALL, 1, &device, nullptr);

        // 创建上下文
        context_ = clCreateContext(nullptr, 1, &device, nullptr, nullptr, &err);
        if (err != CL_SUCCESS) return false;

        // 创建队列
        queue_ = clCreateCommandQueue(context_, device, 0, &err);
        if (err != CL_SUCCESS) {
            clReleaseContext(context_);
            return false;
        }

        return true;
    }

    ~OpenCLContext() {
        if (queue_) clReleaseCommandQueue(queue_);
        if (context_) clReleaseContext(context_);
    }

    cl_context get() { return context_; }
    cl_command_queue getQueue() { return queue_; }

private:
    cl_context context_;
    cl_command_queue queue_;
};

int main() {
    OpenCLContext ctx;
    if (!ctx.init()) {
        std::cerr << "Failed to init OpenCL" << std::endl;
        return -1;
    }

    // 使用 ctx 进行 OpenCL 操作...

    return 0;
}  // 自动清理
```

## 4.8 性能优化建议

### 4.8.1 内存传输优化

**使用零拷贝**：

```cpp
// 创建缓冲区时使用 CL_MEM_USE_HOST_PTR
cl_mem buffer = clCreateBuffer(context,
                                CL_MEM_READ_ONLY | CL_MEM_USE_HOST_PTR,
                                size, hostPtr, &err);

// 或者使用 CL_MEM_ALLOC_HOST_PTR + clEnqueueMapBuffer
cl_mem buffer = clCreateBuffer(context,
                                CL_MEM_ALLOC_HOST_PTR | CL_MEM_READ_ONLY,
                                size, nullptr, &err);

void* mappedPtr = clEnqueueMapBuffer(queue, buffer, CL_TRUE,
                                      CL_MAP_READ, 0, size,
                                      0, nullptr, nullptr, &err);
```

### 4.8.2 异步执行

**使用事件进行异步操作**：

```cpp
cl_event writeEvent, kernelEvent, readEvent;

// 异步写入
clEnqueueWriteBuffer(queue, buffer, CL_FALSE, 0, size, data,
                     0, nullptr, &writeEvent);

// 异步执行（等待写入完成）
clEnqueueNDRangeKernel(queue, kernel, 1, nullptr,
                        &globalSize, nullptr,
                        1, &writeEvent, &kernelEvent);

// 异步读取（等待内核完成）
clEnqueueReadBuffer(queue, resultBuffer, CL_FALSE, 0, size, result,
                    1, &kernelEvent, &readEvent);

// 等待最终结果
clWaitForEvents(1, &readEvent);

// 释放事件
clReleaseEvent(writeEvent);
clReleaseEvent(kernelEvent);
clReleaseEvent(readEvent);
```

---

*本章节详细说明了 OpenCL-Headers 在 OpenHarmony 中的使用方法，包括依赖配置、代码集成示例以及常见问题的处理方法。*
