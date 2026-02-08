# 05_API_Differences.md - API 差异分析

## 核心结论

**astc-encoder 在 OpenHarmony 中没有 API 修改。**

本库完全使用上游原始 API，通过标准头文件 `astcenc.h` 提供功能。OH 特有的定制仅通过条件编译宏控制行为，未改变 API 接口。

---

## 1. API 概述

### 1.1 头文件位置

```
//third_party/astc-encoder/Source/astcenc.h
```

### 1.2 API 设计原则

根据头文件注释，astcenc API 设计目标：

1. **易用性**：非专家也能轻松使用
2. **可控制性**：专家可以精细调整压缩器启发式算法
3. **内存缓冲**：所有输入输出通过内存缓冲区传递
4. **显式大小**：所有缓冲区明确指定大小，防止安全问题

### 1.3 API 版本兼容性

**重要提示**（来自头文件）：

> While the aim is that we keep this interface mostly stable, it should be viewed as a mutable interface tied to a specific source version. **We are not trying to maintain backwards compatibility across codec versions.**

这意味着：
- API 可能在版本间发生变化
- 升级时需要重新编译依赖模块
- 不保证二进制兼容性

---

## 2. 核心 API 函数

### 2.1 配置管理

#### astcenc_config_init

```cpp
astcenc_error astcenc_config_init(
    astcenc_profile profile,
    unsigned int block_x,
    unsigned int block_y,
    unsigned int block_z,
    float quality,
    unsigned int flags,
    astcenc_config* config
);
```

**功能**：初始化编解码器配置

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `profile` | `astcenc_profile` | 配置文件（LDR/HDR/Full） |
| `block_x` | `unsigned int` | 块宽度（1-12） |
| `block_y` | `unsigned int` | 块高度（1-12） |
| `block_z` | `unsigned int` | 块深度（1-12，3D 纹理） |
| `quality` | `float` | 质量预设（0.0-100.0） |
| `flags` | `unsigned int` | 功能标志 |
| `config` | `astcenc_config*` | 输出配置结构 |

**返回值**：`ASTCENC_SUCCESS` 或错误码

**OH 中使用**：标准使用，无修改

---

#### astcenc_config_deinit

```cpp
void astcenc_config_deinit(
    astcenc_config* config
);
```

**功能**：清理配置（当前为空实现，预留未来使用）

---

### 2.2 上下文管理

#### astcenc_context_alloc

```cpp
astcenc_error astcenc_context_alloc(
    const astcenc_config* config,
    unsigned int thread_count,
    astcenc_context** context
);
```

**功能**：分配编解码器上下文

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `config` | `const astcenc_config*` | 配置结构 |
| `thread_count` | `unsigned int` | 工作线程数 |
| `context` | `astcenc_context**` | 输出上下文指针 |

**说明**：
- 上下文可用于顺序压缩多个图像
- 多线程上下文可并行处理单张图像
- 解压专用上下文内存占用更小

**OH 中使用**：标准使用，无修改

---

#### astcenc_context_free

```cpp
void astcenc_context_free(
    astcenc_context* context
);
```

**功能**：释放上下文资源

---

### 2.3 图像压缩

#### astcenc_compress_image

```cpp
astcenc_error astcenc_compress_image(
    astcenc_context* context,
    astcenc_image* image,
    const astcenc_swizzle* swizzle,
    uint8_t* data_out,
    size_t data_len,
    size_t* out_len,
    unsigned int thread_index
);
```

**功能**：压缩单张图像

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `context` | `astcenc_context*` | 编解码器上下文 |
| `image` | `astcenc_image*` | 输入图像 |
| `swizzle` | `const astcenc_swizzle*` | 颜色通道重排 |
| `data_out` | `uint8_t*` | 输出缓冲区 |
| `data_len` | `size_t` | 输出缓冲区大小 |
| `out_len` | `size_t*` | 实际输出大小 |
| `thread_index` | `unsigned int` | 线程索引（多线程时使用） |

**OH 中使用**：标准使用，无修改

---

#### astcenc_compress_reset

```cpp
void astcenc_compress_reset(
    astcenc_context* context
);
```

**功能**：重置上下文，准备压缩新图像

---

### 2.4 图像解压

#### astcenc_decompress_image

```cpp
astcenc_error astcenc_decompress_image(
    astcenc_context* context,
    const uint8_t* data,
    size_t data_len,
    astcenc_image* image_out,
    const astcenc_swizzle* swizzle,
    unsigned int thread_index
);
```

**功能**：解压单张图像

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| `context` | `astcenc_context*` | 编解码器上下文 |
| `data` | `const uint8_t*` | ASTC 压缩数据 |
| `data_len` | `size_t` | 数据长度 |
| `image_out` | `astcenc_image*` | 输出图像 |
| `swizzle` | `const astcenc_swizzle*` | 颜色通道重排 |
| `thread_index` | `unsigned int` | 线程索引 |

**OH 中使用**：标准使用，无修改

---

#### astcenc_decompress_reset

```cpp
void astcenc_decompress_reset(
    astcenc_context* context
);
```

**功能**：重置上下文，准备解压新图像

---

### 2.5 实用函数

#### astcenc_get_error_string

```cpp
const char* astcenc_get_error_string(
    astcenc_error status
);
```

**功能**：将错误码转换为人类可读字符串

**返回值**：错误描述字符串

---

## 3. 核心数据结构

### 3.1 astcenc_profile（配置文件）

```cpp
enum astcenc_profile {
    ASTCENC_PRF_LDR,    // 低动态范围
    ASTCENC_PRF_HDR,    // 高动态范围
    ASTCENC_PRF_HDR_RGB_LDR_A,  // HDR RGB + LDR Alpha
    ASTCENC_PRF_FULL    // 完整功能（2D/3D + LDR/HDR）
};
```

**OH 中使用**：标准使用，无修改

---

### 3.2 astcenc_config（配置结构）

```cpp
struct astcenc_config {
    astcenc_profile profile;
    unsigned int block_x;
    unsigned int block_y;
    unsigned int block_z;
    unsigned int flags;
    // ... 其他内部字段
};
```

**说明**：
- 由 `astcenc_config_init` 初始化
- 专家用户可直接修改某些字段调整启发式算法

**OH 中使用**：标准使用，无修改

---

### 3.3 astcenc_image（图像结构）

```cpp
struct astcenc_image {
    unsigned int dim_x;      // 宽度
    unsigned int dim_y;      // 高度
    unsigned int dim_z;      // 深度（3D 纹理）
    astcenc_type data_type;  // 数据类型（U8/F16/F32）
    void** data;             // 图像数据（三维数组）
};
```

**数据布局**：
```
data[z_coord][y_coord * x_dim * 4 + x_coord * 4 + 0]  // R
data[z_coord][y_coord * x_dim * 4 + x_coord * 4 + 1]  // G
data[z_coord][y_coord * x_dim * 4 + x_coord * 4 + 2]  // B
data[z_coord][y_coord * x_dim * 4 + x_coord * 4 + 3]  // A
```

**OH 中使用**：标准使用，无修改

---

### 3.4 astcenc_swizzle（通道重排）

```cpp
struct astcenc_swizzle {
    uint8_t r;  // 输出 R 通道来源（0-3，或 4 表示 1.0）
    uint8_t g;  // 输出 G 通道来源
    uint8_t b;  // 输出 B 通道来源
    uint8_t a;  // 输出 A 通道来源
};
```

**预定义常量**：
```cpp
#define ASTCENC_SWZ_R 0
#define ASTCENC_SWZ_G 1
#define ASTCENC_SWZ_B 2
#define ASTCENC_SWZ_A 3
#define ASTCENC_SWZ_0 4  // 常量 0.0
#define ASTCENC_SWZ_1 5  // 常量 1.0
```

**常见用法**：
```cpp
// RGBA（默认）
astcenc_swizzle swizzle{ ASTCENC_SWZ_R, ASTCENC_SWZ_G, ASTCENC_SWZ_B, ASTCENC_SWZ_A };

// BGRA（交换 R 和 B）
astcenc_swizzle swizzle{ ASTCENC_SWZ_B, ASTCENC_SWZ_G, ASTCENC_SWZ_R, ASTCENC_SWZ_A };

// 灰度（仅 R 通道，G 和 B 复制 R）
astcenc_swizzle swizzle{ ASTCENC_SWZ_R, ASTCENC_SWZ_R, ASTCENC_SWZ_R, ASTCENC_SWZ_1 };

// 预定义：默认 swizzle
astcenc_swizzle swizzle = { ASTCENC_SWZ_R, ASTCENC_SWZ_G, ASTCENC_SWZ_B, ASTCENC_SWZ_A };
```

**OH 中使用**：标准使用，无修改

---

### 3.5 astcenc_error（错误码）

```cpp
enum astcenc_error {
    ASTCENC_SUCCESS = 0,                    // 成功
    ASTCENC_ERR_OUT_OF_MEM,                 // 内存不足
    ASTCENC_ERR_BAD_CPU_FLOAT,              // CPU 浮点支持不足
    ASTCENC_ERR_BAD_PARAM,                  // 参数错误
    ASTCENC_ERR_BAD_BLOCK_SIZE,             // 块大小不支持
    ASTCENC_ERR_BAD_PROFILE,                // 配置文件不支持
    ASTCENC_ERR_BAD_QUALITY,                // 质量设置不支持
    ASTCENC_ERR_BAD_FLAGS,                  // 标志错误
    ASTCENC_ERR_BAD_CONTEXT,                // 上下文无效
    ASTCENC_ERR_NOT_INIT,                   // 上下文未初始化
    ASTCENC_ERR_BAD_DECOMPRESS_ONLY,        // 解压专用上下文错误使用
    ASTCENC_ERR_BAD_COMPRESS_ONLY,          // 压缩专用上下文错误使用
};
```

**OH 中使用**：标准使用，无修改

---

## 4. 功能标志（Flags）

### 4.1 标志常量

```cpp
// 无特殊标志
#define ASTCENC_FLG_NONE 0x0000

// 解压专用模式（减小内存占用）
#define ASTCENC_FLG_DECOMPRESS_ONLY 0x0001

// 使用 decode_unorm8 舍入规则（4.7.0+）
#define ASTCENC_FLG_USE_DECODE_UNORM8 0x0002

// 自解压专用模式（压缩和解压使用相同质量预设）
#define ASTCENC_FLG_SELF_DECOMPRESS_ONLY 0x0004
```

### 4.2 OH 特有标志

**当前状态**：无 OH 特有标志

**条件编译宏**（BUILD.gn 中定义，非 API 标志）：
- `ASTC_CUSTOMIZED_ENABLE`
- `SUT_PATH_X64`
- `BUILD_HMOS_SDK`

这些宏控制编译行为，不影响 API 接口。

---

## 5. 使用模式对比

### 5.1 标准使用流程

```cpp
// 1. 初始化配置
astcenc_config config;
astcenc_config_init(ASTCENC_PRF_LDR, 6, 6, 1, ASTCENC_PRE_MEDIUM, ASTCENC_FLG_NONE, 1, &config);

// 2. 分配上下文
astcenc_context* context;
astcenc_context_alloc(&config, 1, &context);

// 3. 压缩/解压图像
// ...

// 4. 释放上下文
astcenc_context_free(context);
```

### 5.2 OH 中的使用

**使用方式完全相同**，OH 不修改 API 调用方式。

唯一的区别是 BUILD.gn 可能根据产品配置定义宏：
```cpp
#ifdef ASTC_CUSTOMIZED_ENABLE
    // 可能启用某些功能
#endif
```

**注意**：目前未在源码中发现 `ASTC_CUSTOMIZED_ENABLE` 等宏的实际使用，可能是预留或已移除的功能。

---

## 6. 与上游 API 的对比

### 6.1 API 函数对比

| 函数 | 上游 4.7.0 | OpenHarmony | 差异 |
|------|-----------|-------------|------|
| `astcenc_config_init` | ✅ | ✅ | 无 |
| `astcenc_config_deinit` | ✅ | ✅ | 无 |
| `astcenc_context_alloc` | ✅ | ✅ | 无 |
| `astcenc_context_free` | ✅ | ✅ | 无 |
| `astcenc_compress_image` | ✅ | ✅ | 无 |
| `astcenc_compress_reset` | ✅ | ✅ | 无 |
| `astcenc_decompress_image` | ✅ | ✅ | 无 |
| `astcenc_decompress_reset` | ✅ | ✅ | 无 |
| `astcenc_get_error_string` | ✅ | ✅ | 无 |

### 6.2 数据结构对比

| 结构 | 上游 4.7.0 | OpenHarmony | 差异 |
|------|-----------|-------------|------|
| `astcenc_config` | ✅ | ✅ | 无 |
| `astcenc_image` | ✅ | ✅ | 无 |
| `astcenc_swizzle` | ✅ | ✅ | 无 |
| `astcenc_error` | ✅ | ✅ | 无 |
| `astcenc_profile` | ✅ | ✅ | 无 |

### 6.3 标志对比

| 标志 | 上游 4.7.0 | OpenHarmony | 差异 |
|------|-----------|-------------|------|
| `ASTCENC_FLG_NONE` | ✅ | ✅ | 无 |
| `ASTCENC_FLG_DECOMPRESS_ONLY` | ✅ | ✅ | 无 |
| `ASTCENC_FLG_USE_DECODE_UNORM8` | ✅ | ✅ | 无 |
| `ASTCENC_FLG_SELF_DECOMPRESS_ONLY` | ✅ | ✅ | 无 |

---

## 7. 新增 API 可能性分析

### 7.1 当前状态

**无 OH 特有新增 API**

### 7.2 未来可能的新增 API

如果未来需要 OH 特有功能，可能考虑：

| 潜在功能 | 可能的 API | 备注 |
|---------|-----------|------|
| OH 特定内存管理 | `astcenc_oh_alloc_image` | 使用 OH 内存分配器 |
| OH 特定性能调优 | `astcenc_oh_set_performance_hint` | 针对 OH 设备优化 |
| HMOS SDK 功能 | `astcenc_oh_sdk_init` | SDK 特定初始化 |

**当前建议**：无需新增 API，使用标准 API 即可满足需求。

---

## 8. 调用示例

### 8.1 完整压缩示例

```cpp
#include "astcenc.h"
#include <cstdio>
#include <cstdlib>

int main() {
    // 配置参数
    const unsigned int block_x = 6;
    const unsigned int block_y = 6;
    const float quality = ASTCENC_PRE_MEDIUM;
    const unsigned int thread_count = 4;
    
    // 1. 初始化配置
    astcenc_config config;
    astcenc_error status = astcenc_config_init(
        ASTCENC_PRF_LDR,
        block_x, block_y, 1,
        quality,
        ASTCENC_FLG_NONE,
        &config
    );
    
    if (status != ASTCENC_SUCCESS) {
        printf("Config init failed: %s\n", astcenc_get_error_string(status));
        return 1;
    }
    
    // 2. 分配上下文
    astcenc_context* context;
    status = astcenc_context_alloc(&config, thread_count, &context);
    if (status != ASTCENC_SUCCESS) {
        printf("Context alloc failed: %s\n", astcenc_get_error_string(status));
        return 1;
    }
    
    // 3. 准备输入图像（256x256 RGBA）
    const unsigned int width = 256;
    const unsigned int height = 256;
    astcenc_image input_image;
    input_image.dim_x = width;
    input_image.dim_y = height;
    input_image.dim_z = 1;
    input_image.data_type = ASTCENC_TYPE_U8;
    
    // 分配图像数据
    uint8_t* image_data = (uint8_t*)malloc(width * height * 4);
    input_image.data = (void**)&image_data;
    
    // 填充图像数据（示例：渐变）
    for (unsigned int y = 0; y < height; y++) {
        for (unsigned int x = 0; x < width; x++) {
            image_data[(y * width + x) * 4 + 0] = (uint8_t)(x % 256);
            image_data[(y * width + x) * 4 + 1] = (uint8_t)(y % 256);
            image_data[(y * width + x) * 4 + 2] = 128;
            image_data[(y * width + x) * 4 + 3] = 255;
        }
    }
    
    // 4. 计算输出缓冲区大小
    const unsigned int block_count_x = (width + block_x - 1) / block_x;
    const unsigned int block_count_y = (height + block_y - 1) / block_y;
    const size_t output_buffer_size = block_count_x * block_count_y * 16;  // 每个 ASTC 块 16 字节
    
    uint8_t* output_buffer = (uint8_t*)malloc(output_buffer_size);
    size_t compressed_size = 0;
    
    // 5. 压缩图像
    astcenc_swizzle swizzle = { ASTCENC_SWZ_R, ASTCENC_SWZ_G, ASTCENC_SWZ_B, ASTCENC_SWZ_A };
    status = astcenc_compress_image(
        context,
        &input_image,
        &swizzle,
        output_buffer,
        output_buffer_size,
        &compressed_size,
        0  // 线程索引（单线程示例）
    );
    
    if (status != ASTCENC_SUCCESS) {
        printf("Compress failed: %s\n", astcenc_get_error_string(status));
        free(image_data);
        free(output_buffer);
        astcenc_context_free(context);
        return 1;
    }
    
    printf("Compressed %dx%d image to %zu bytes\n", width, height, compressed_size);
    
    // 6. 清理
    free(image_data);
    free(output_buffer);
    astcenc_context_free(context);
    
    return 0;
}
```

### 8.2 BUILD.gn 配置

```gn
ohos_executable("astc_example") {
  sources = [ "astc_example.cpp" ]
  external_deps = [
    "astc-encoder:astc_encoder_shared",
  ]
}
```

---

## 9. 总结

### 9.1 API 状态

| 项目 | 状态 |
|------|------|
| API 修改 | 无 |
| 新增 API | 无 |
| 废弃 API | 无 |
| 行为变更 | 无 |

### 9.2 使用建议

1. **直接使用标准 API**：无需关注 OH 特有逻辑
2. **参考上游文档**：官方 API 文档完全适用
3. **关注版本更新**：API 可能随上游版本变化
4. **完整错误处理**：始终检查返回值

### 9.3 参考资源

- **头文件**：`Source/astcenc.h`（详细注释）
- **上游文档**：https://github.com/ARM-software/astc-encoder/blob/main/Docs/
- **示例代码**：`Utils/Example/`
- **单元测试**：`Source/UnitTest/`
