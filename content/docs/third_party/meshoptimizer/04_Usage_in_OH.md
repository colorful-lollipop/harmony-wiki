# 在 OpenHarmony 中的使用

## 4.1 使用现状

### 4.1.1 当前依赖状态

**重要说明**: meshoptimizer **目前尚未被 OpenHarmony 其他模块显式依赖**。

该库目前处于"**备用就绪**"状态，已完成 OHOS 构建系统集成，但等待需要 3D 网格优化能力的模块进行引用。

### 4.1.2 预期使用场景

根据 OpenHarmony 的技术规划，meshoptimizer 预期将在以下场景中使用：

| 场景 | 用途 | 相关模块 |
|------|------|---------|
| glTF 模型加载 | 解码 EXT_meshopt_compression 扩展 | 资源加载、3D 引擎 |
| 实时渲染优化 | 顶点缓存、过度绘制优化 | ACE 引擎、图形模块 |
| LOD 管理 | 网格简化算法 | 场景管理、视口管理 |
| 资源压缩 | 减少 3D 资源大小 | 资源打包、应用包体积优化 |

---

## 4.2 依赖声明

### 4.2.1 BUILD.gn 依赖配置

要在您的 OHOS 模块中使用 meshoptimizer，请在 BUILD.gn 文件中添加依赖：

```gn
# 方式 1: 直接依赖
deps = ["//third_party/meshoptimizer:meshoptimizer"]

# 方式 2: 带可见性的依赖
ohos_executable("my_app") {
    sources = ["src/main.cpp"]
    deps = [
        "//third_party/meshoptimizer:meshoptimizer",
    ]
    # ...
}

# 方式 3: 如果您有定制的库目标
ohos_shared_library("my_graphics_lib") {
    sources = ["src/graphics.cpp"]
    deps = [
        "//third_party/meshoptimizer:meshoptimizer",
    ]
    # ...
}
```

### 4.2.2 头文件引用

添加依赖后，可以通过以下方式引用头文件：

```cpp
// 方式 1: 直接引用（推荐）
#include "meshoptimizer.h"

// 方式 2: 如果头文件搜索路径未正确配置
#include "third_party/meshoptimizer/src/meshoptimizer.h"
```

---

## 4.3 基础 API 使用

### 4.3.1 核心解码 API

meshoptimizer 在 OHOS 中的**主要用途**是解码 glTF 的 EXT_meshopt_compression 扩展。

#### 顶点缓冲区解码

```cpp
#include "meshoptimizer.h"

// 解码压缩的顶点缓冲区
int decodeVertexBuffer(
    void* destination,           // 目标缓冲区
    size_t vertex_count,         // 顶点数量
    size_t vertex_size,          // 顶点大小（字节）
    const void* source,          // 编码数据源
    size_t source_size           // 编码数据大小
);

// 使用示例
std::vector<unsigned char> vertex_data(compressed_size);
std::vector<float> vertices(vertex_count * vertex_size);

int result = meshopt_decodeVertexBuffer(
    vertices.data(),
    vertex_count,
    vertex_size,
    vertex_data.data(),
    vertex_data.size()
);

if (result != 0) {
    // 处理解码错误
    // result != 0 表示解码失败
}
```

#### 索引缓冲区解码

```cpp
#include "meshoptimizer.h"

// 解码压缩的索引缓冲区
int decodeIndexBuffer(
    void* destination,           // 目标缓冲区
    size_t index_count,          // 索引数量
    const void* source,          // 编码数据源
    size_t source_size           // 编码数据大小
);

// 使用示例
std::vector<unsigned char> index_data(compressed_size);
std::vector<unsigned int> indices(index_count);

int result = meshopt_decodeIndexBuffer(
    indices.data(),
    index_count,
    index_data.data(),
    index_data.size()
);
```

### 4.3.2 滤波解码 API

meshoptimizer 提供三种滤波编解码器，用于压缩额外的顶点属性：

#### 八面体滤波解码

```cpp
#include "meshoptimizer.h"

// 解码八面体编码的法线/切线
void decodeFilterOct(
    void* destination,           // 目标缓冲区
    size_t count,                 // 元素数量
    const void* source,          // 编码数据
    size_t size,                 // 编码数据大小
    size_t stride,               // 目标元素跨度
    size_t components            // 每个元素的组件数（通常为3）
);

// 使用示例：解码法线
std::vector<float> normals(vertex_count * 3);
meshopt_decodeFilterOct(
    normals.data(),
    vertex_count,
    encoded_data,
    encoded_size,
    sizeof(float) * 3,
    3
);
```

#### 四元数滤波解码

```cpp
#include "meshoptimizer.h"

// 解码四元数编码的旋转
void decodeFilterQuat(
    void* destination,
    size_t count,
    const void* source,
    size_t size,
    size_t stride,
    size_t components  // 通常为4
);

// 使用示例：解码切线四元数
std::vector<float> tangents(vertex_count * 4);
meshopt_decodeFilterQuat(
    tangents.data(),
    vertex_count,
    encoded_data,
    encoded_size,
    sizeof(float) * 4,
    4
);
```

#### 指数滤波解码

```cpp
#include "meshoptimizer.h"

// 解码指数编码的浮点数据
void decodeFilterExp(
    void* destination,
    size_t count,
    const void* source,
    size_t size,
    size_t stride,
    size_t components
);

// 使用示例：解码自定义浮点属性
std::vector<float> attributes(vertex_count * 4);
meshopt_decodeFilterExp(
    attributes.data(),
    vertex_count,
    encoded_data,
    encoded_size,
    sizeof(float) * 4,
    4
);
```

---

## 4.4 典型使用场景

### 4.4.1 glTF 模型加载器集成

#### 场景描述

在 OHOS 应用中加载 glTF 模型时，遇到使用 EXT_meshopt_compression 扩展的模型时，需要使用 meshoptimizer 进行解码。

#### 实现示例

```cpp
#include "meshoptimizer.h"
#include <fstream>
#include <vector>

class GLTFLoader {
public:
    bool loadMesh(const std::string& path) {
        // 1. 读取 glTF 文件
        std::ifstream file(path, std::ios::binary | std::ios::ate);
        if (!file.is_open()) {
            return false;
        }
        
        size_t file_size = file.tellg();
        std::vector<char> file_data(file_size);
        file.seekg(0);
        file.read(file_data.data(), file_size);
        file.close();
        
        // 2. 解析 glTF JSON（省略具体实现）
        // 3. 检查是否使用 meshopt 压缩
        bool use_meshopt_compression = checkMeshoptExtension(file_data);
        
        if (use_meshopt_compression) {
            // 4. 解码压缩的顶点数据
            return decodeMeshoptData(file_data);
        }
        
        return true;
    }
    
private:
    bool decodeMeshoptData(const std::vector<char>& file_data) {
        // 从 glTF buffer 中提取编码数据
        const void* vertex_data = getVertexBufferData(file_data);
        const void* index_data = getIndexBufferData(file_data);
        
        size_t vertex_count = getVertexCount(file_data);
        size_t index_count = getIndexCount(file_data);
        size_t vertex_size = getVertexSize(file_data);
        
        // 解码顶点缓冲区
        std::vector<float> decoded_vertices(vertex_count * vertex_size / sizeof(float));
        int result = meshopt_decodeVertexBuffer(
            decoded_vertices.data(),
            vertex_count,
            vertex_size,
            vertex_data,
            getVertexBufferSize(file_data)
        );
        
        if (result != 0) {
            return false;  // 解码失败
        }
        
        // 解码索引缓冲区
        std::vector<unsigned int> decoded_indices(index_count);
        result = meshopt_decodeIndexBuffer(
            decoded_indices.data(),
            index_count,
            index_data,
            getIndexBufferSize(file_data)
        );
        
        if (result != 0) {
            return false;  // 解码失败
        }
        
        // 使用解码后的数据渲染
        // ...
        
        return true;
    }
};
```

### 4.4.2 运行时网格优化

#### 场景描述

对于需要在运行时优化 3D 网格的应用，可以使用 meshoptimizer 的优化算法。

#### 实现示例

```cpp
#include "meshoptimizer.h"

void optimizeMesh(
    unsigned int* indices,           // 索引缓冲区
    float* vertices,                 // 顶点缓冲区（位置）
    size_t index_count,              // 索引数量
    size_t vertex_count,             // 顶点数量
    size_t vertex_size               // 顶点大小（字节）
) {
    // 1. 顶点缓存优化
    meshopt_optimizeVertexCache(
        indices,
        indices,
        index_count,
        vertex_count
    );
    
    // 2. 顶点提取优化
    meshopt_optimizeVertexFetch(
        vertices,
        indices,
        index_count,
        vertices,
        vertex_count,
        vertex_size
    );
    
    // 3. 量化顶点数据（可选）
    // 将浮点位置量化为 16 位整数
    for (size_t i = 0; i < vertex_count; i++) {
        float* vertex = reinterpret_cast<float*>(
            reinterpret_cast<char*>(vertices) + i * vertex_size
        );
        unsigned short quantized_position[3];
        quantized_position[0] = meshopt_quantizeHalf(vertex[0]);
        quantized_position[1] = meshopt_quantizeHalf(vertex[1]);
        quantized_position[2] = meshopt_quantizeHalf(vertex[2]);
        // ...
    }
}
```

### 4.4.3 LOD 生成

#### 场景描述

使用 meshoptimizer 的简化算法生成多级细节 (LOD)，用于视距相关的细节调整。

#### 实现示例

```cpp
#include "meshoptimizer.h"

struct LODLevel {
    std::vector<unsigned int> indices;
    float error;
};

std::vector<LODLevel> generateLODs(
    const unsigned int* indices,
    const float* vertices,
    size_t index_count,
    size_t vertex_count,
    size_t vertex_size
) {
    std::vector<LODLevel> lods;
    
    // 生成多个 LOD 级别
    float target_error = 1e-1f;  // 初始目标误差
    
    for (int level = 0; level < 4; level++) {
        size_t target_index_count = index_count / (1 << level);  // 每级减少 50%
        
        LODLevel lod;
        lod.indices.resize(target_index_count);
        
        float lod_error = 0.f;
        lod.indices.resize(meshopt_simplify(
            lod.indices.data(),
            indices,
            index_count,
            vertices,
            vertex_count,
            vertex_size,
            target_index_count,
            target_error,
            0,  // 选项
            &lod_error
        ));
        
        lod.error = lod_error;
        lods.push_back(lod);
        
        // 下一级使用更严格的目标误差
        target_error *= 0.5f;
    }
    
    return lods;
}
```

---

## 4.5 内存管理

### 4.5.1 默认内存分配器

meshoptimizer 默认使用 C++ 的 `operator new` 和 `operator delete` 进行内存分配：

```cpp
// 内部使用示例（在库代码中）
void* buffer = operator new(size);
// ...
operator delete(buffer);
```

### 4.5.2 自定义内存分配器

如果您的应用需要控制内存分配，可以使用 `meshopt_setAllocator` 设置自定义分配器：

```cpp
#include "meshoptimizer.h"

// 自定义分配函数
void* customAllocate(size_t size) {
    // 实现自定义分配逻辑
    return malloc(size);
}

// 自定义释放函数
void customFree(void* ptr) {
    // 实现自定义释放逻辑
    free(ptr);
}

// 设置自定义分配器
void setupAllocator() {
    meshopt_setAllocator(customAllocate, customFree);
}
```

### 4.5.3 零内存分配 API

meshoptimizer 提供的**解码函数不进行动态内存分配**，适合内存敏感的应用：

```cpp
// 这些函数不分配内存，直接在提供的缓冲区中工作

// ✅ 解码函数 - 零内存分配
meshopt_decodeVertexBuffer(destination, count, size, source, source_size);
meshopt_decodeIndexBuffer(destination, count, source, source_size);

// ⚠️ 编码函数 - 需要预计算缓冲区大小
size_t bound = meshopt_encodeVertexBufferBound(vertex_count, vertex_size);
std::vector<unsigned char> buffer(bound);
meshopt_encodeVertexBuffer(buffer.data(), buffer.size(), /* ... */);
```

---

## 4.6 错误处理

### 4.6.1 返回值说明

| 函数 | 返回值 | 说明 |
|------|-------|------|
| `meshopt_decodeVertexBuffer` | `int` | 0 = 成功，非 0 = 失败 |
| `meshopt_decodeIndexBuffer` | `int` | 0 = 成功，非 0 = 失败 |
| 优化函数 (如 `meshopt_simplify`) | `size_t` | 返回实际处理的元素数量 |
| 分析函数 | `meshopt_AnalyzerResult` | 返回分析结果结构 |

### 4.6.2 完整错误处理示例

```cpp
#include "meshoptimizer.h>
#include <stdexcept>

class MeshOptimizer {
public:
    void decodeMesh(const MeshData& input, MeshData& output) {
        // 1. 分配输出缓冲区
        output.vertices.resize(input.vertex_count * input.vertex_size);
        output.indices.resize(input.index_count * sizeof(unsigned int));
        
        // 2. 解码顶点缓冲区
        int result = meshopt_decodeVertexBuffer(
            output.vertices.data(),
            input.vertex_count,
            input.vertex_size,
            input.compressed_vertices.data(),
            input.compressed_vertices.size()
        );
        
        if (result != 0) {
            throw std::runtime_error(
                "顶点缓冲区解码失败，错误码: " + std::to_string(result)
            );
        }
        
        // 3. 解码索引缓冲区
        result = meshopt_decodeIndexBuffer(
            output.indices.data(),
            input.index_count,
            input.compressed_indices.data(),
            input.compressed_indices.size()
        );
        
        if (result != 0) {
            throw std::runtime_error(
                "索引缓冲区解码失败，错误码: " + std::to_string(result)
            );
        }
        
        // 4. 解码属性滤波（如果有）
        if (input.use_filter_oct) {
            decodeAttributeFilters(input, output);
        }
    }
    
private:
    void decodeAttributeFilters(const MeshData& input, MeshData& output) {
        // 八面体滤波解码
        meshopt_decodeFilterOct(
            output.normals.data(),
            output.vertex_count,
            input.encoded_normals.data(),
            input.encoded_normals.size(),
            sizeof(float) * 3,
            3
        );
        
        // 四元数滤波解码
        meshopt_decodeFilterQuat(
            output.tangents.data(),
            output.vertex_count,
            input.encoded_tangents.data(),
            input.encoded_tangents.size(),
            sizeof(float) * 4,
            4
        );
    }
};
```

---

## 4.7 线程安全性

### 4.7.1 线程安全保证

meshoptimizer 的设计**不包含任何全局状态**，因此：

| 特性 | 线程安全性 | 说明 |
|------|-----------|------|
| 解码函数 | ✅ 线程安全 | 纯函数，无共享状态 |
| 编码函数 | ✅ 线程安全 | 纯函数，无共享状态 |
| 优化函数 | ✅ 线程安全 | 纯函数，无共享状态 |
| 内存分配器 | ⚠️ 取决于实现 | 自定义分配器需自行保证 |

### 4.7.2 多线程使用示例

```cpp
#include "meshoptimizer.h"
#include <thread>
#include <vector>

// 并行解码多个网格
void decodeMeshesParallel(
    std::vector<MeshData>& meshes
) {
    std::vector<std::thread> threads;
    
    for (auto& mesh : meshes) {
        threads.emplace_back([&mesh]() {
            // 每个线程独立处理一个网格
            meshopt_decodeVertexBuffer(/* ... */);
            meshopt_decodeIndexBuffer(/* ... */);
        });
    }
    
    // 等待所有线程完成
    for (auto& t : threads) {
        t.join();
    }
}
```

---

## 4.8 性能建议

### 4.8.1 缓冲区分配

```cpp
// ✅ 推荐：预分配足够大的缓冲区
std::vector<unsigned char> vertex_buffer(
    meshopt_encodeVertexBufferBound(vertex_count, vertex_size)
);
size_t encoded_size = meshopt_encodeVertexBuffer(
    vertex_buffer.data(),
    vertex_buffer.size(),
    /* ... */
);
vertex_buffer.resize(encoded_size);  // 调整到实际大小
```

### 4.8.2 避免不必要的拷贝

```cpp
// ✅ 推荐：直接解码到目标缓冲区
meshopt_decodeVertexBuffer(
    final_vertex_buffer.data(),  // 直接写入最终缓冲区
    vertex_count,
    vertex_size,
    encoded_data.data(),
    encoded_data.size()
);

// ❌ 不推荐：先解码到临时缓冲区，再拷贝
std::vector<unsigned char> temp_buffer(...);
meshopt_decodeVertexBuffer(temp_buffer.data(), /* ... */);
std::copy(temp_buffer.begin(), temp_buffer.end(), final_vertex_buffer.begin());
```

### 4.8.3 批量处理

```cpp
// ✅ 推荐：批量解码提高缓存效率
for (const auto& mesh : mesh_batch) {
    meshopt_decodeVertexBuffer(/* ... */);
    meshopt_decodeIndexBuffer(/* ... */);
}
```

---

## 4.9 总结

### 使用要点

| 要点 | 说明 |
|------|------|
| 依赖声明 | `deps = ["//third_party/meshoptimizer:meshoptimizer"]` |
| 头文件引用 | `#include "meshoptimizer.h"` |
| 主要用途 | glTF EXT_meshopt_compression 解码 |
| 内存分配 | 支持自定义分配器 |
| 线程安全 | ✅ 完全线程安全 |
| 零依赖 | 无第三方依赖 |

### 快速参考

```cpp
// 最小使用示例
#include "meshoptimizer.h"

std::vector<float> vertices(vertex_count * vertex_size);
meshopt_decodeVertexBuffer(
    vertices.data(),
    vertex_count,
    vertex_size,
    encoded_data.data(),
    encoded_data.size()
);
```

---

*文档版本: v1.0*
*最后更新: 2026-02-07*
