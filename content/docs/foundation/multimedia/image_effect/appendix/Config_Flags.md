# 配置宏与 Feature Flags

## 概述

本文档记录 ImageEffect 模块中关键的编译配置宏、Feature Flags 和运行时参数，便于开发者理解和调整模块行为。

## 编译时配置

### 项目全局配置

**配置文件**：`config.gni`

| 配置项 | 默认值 | 用途 | 证据 |
|--------|--------|------|------|
| `image_effect_root_dir` | "//foundation/multimedia/image_effect" | 项目根路径 | config.gni |
| `image_effect_colorspace_convertor_enable` | false | 是否启用色彩空间转换器 | config.gni |
| `image_effect_sanitize.integer_overflow` | true | 整数溢出检测（编译时） | config.gni |
| `image_effect_sanitize.ubsan` | true | 未定义行为检测 | config.gni |
| `image_effect_sanitize.boundary_sanitize` | true | 边界检查 | config.gni |
| `image_effect_sanitize.cfi` | true | 控制流完整性 | config.gni |
| `image_effect_sanitize.cfi_cross_dso` | true | 跨 DSO 控制流完整性 | config.gni |
| `image_effect_sanitize.cfi_vcall_icall_only` | true | 仅虚函数/间接调用检查 | config.gni |
| `image_effect_sanitize.debug` | false | 调试模式 | config.gni |

### 模块级配置

**配置文件**：`frameworks/native/BUILD.gn`

#### image_effect_impl 宏定义

| 宏 | 定义 | 用途 | 证据 |
|----|------|------|------|
| `HST_ANY_WITH_NO_RTTI` | 定义 | 禁用 RTTI | BUILD.gn |
| `IMAGE_COLORSPACE_FLAG` | 定义 | 启用色彩空间标记 | BUILD.gn |
| `VPE_ENABLE` | 条件定义 | 启用视频处理引擎 | BUILD.gn |

#### 公共头文件搜索路径

```gn
include_dirs = [
  "interfaces/inner_api/native/*",
  "frameworks/native/effect/pipeline/include/*",
  "frameworks/native/effect/base",
  "frameworks/native/render_environment/*",
]
```

## 运行时常量

### 内存限制

| 常量 | 值 | 用途 | 证据 |
|------|-----|------|------|
| `MAX_RAM_SIZE` | 256 * 1024 * 1024 (256MB) | 单次内存分配上限 | effect_memory.cpp:52 |
| `MAX_EFILTER_NUMS` | 100 | 最大滤镜数量 | image_effect.cpp:65 |
| `MAX_CHAR_LEN` | 1024 | 字符串最大长度 | image_effect_filter.cpp:89 |

### 缓冲区配置

| 常量 | 用途 | 证据 |
|------|------|------|
| `DEFAULT_BUFFER_ALIGNMENT` | 默认缓冲区对齐 | effect_buffer.h |

## 滤镜名称宏

**定义文件**：`frameworks/native/capi/image_effect_filter.cpp`

| 宏 | 值 | 滤镜类型 | 证据 |
|----|------|----------|------|
| `OH_EFFECT_BRIGHTNESS_FILTER` | "brightness" | 亮度滤镜 | image_effect_filter.cpp:63 |
| `OH_EFFECT_CONTRAST_FILTER` | "contrast" | 对比度滤镜 | image_effect_filter.cpp |
| `OH_EFFECT_CROP_FILTER` | "crop" | 裁剪滤镜 | image_effect_filter.cpp |
| `OH_EFFECT_FILTER_INTENSITY_KEY` | "intensity" | 滤镜强度参数 | image_effect_filter.cpp:91 |

## 数据类型枚举

### ImageEffect_DataType

**定义文件**：`interfaces/kits/native/image_effect_filter.h:108`

| 枚举值 | 说明 | 字节大小 |
|--------|------|----------|
| `EFFECT_DATA_TYPE_INT32` | 32 位整数 | 4 |
| `EFFECT_DATA_TYPE_FLOAT` | 单精度浮点 | 4 |
| `EFFECT_DATA_TYPE_DOUBLE` | 双精度浮点 | 8 |
| `EFFECT_DATA_TYPE_BOOL` | 布尔值 | 1 |
| `EFFECT_DATA_TYPE_POINTER` | 指针 | 8 |

### ImageEffect_ErrorCode

**定义文件**：`interfaces/kits/native/image_effect_errors.h`

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERR_OK` | 0 | 成功 |
| `ERR_INPUT_NULL` | 1 | 输入参数为空 |
| `ERR_PARAM_INVALID` | 2 | 参数无效 |
| `ERR_MEMCPY_FAIL` | 3 | 内存拷贝失败 |
| `ERR_ALLOC_MEMORY_FAIL` | 4 | 内存分配失败 |
| `ERR_FILTER_NOT_FOUND` | 5 | 滤镜不存在 |
| `ERR_CREATE_FAILED` | 6 | 创建失败 |
| `ERR_START_FAILED` | 7 | 启动失败 |
| `ERR_STOP_FAILED` | 8 | 停止失败 |
| `ERR_PERMISSION_DENIED` | 9 | 权限拒绝 |

### ConfigType

**定义文件**：`interfaces/inner_api/native/base/effect_type.h`

| 枚举值 | 说明 |
|--------|------|
| `CONFIG_TYPE_DEFAULT` | 默认配置 |
| `CONFIG_TYPE_RENDER` | 渲染配置 |
| `CONFIG_TYPE_CUSTOM` | 自定义配置 |

### BufferType

**定义文件**：`interfaces/inner_api/native/base/effect_type.h`

| 枚举值 | 说明 |
|--------|------|
| `BUFFER_TYPE_DEFAULT` | 默认缓冲区 |
| `BUFFER_TYPE_CPU` | CPU 缓冲区 |
| `BUFFER_TYPE_GPU` | GPU 缓冲区 |
| `BUFFER_TYPE_SHARED` | 共享缓冲区 |

## 错误处理宏

**定义文件**：`interfaces/inner_api/native/common/error_code.h`

| 宏 | 签名 | 功能 | 证据 |
|----|------|------|------|
| `CHECK_AND_RETURN_RET_LOG` | `(cond, ret, msg)` | 条件检查，失败时记录日志并返回 | error_code.h:131 |
| `FALSE_RETURN_E` | `(cond, ret)` | 条件为假时返回错误码 | error_code.h |
| `FALSE_RETURN_MSG_E` | `(cond, ret, msg)` | 条件为假时返回错误码并记录消息 | error_code.h |
| `FAIL_RETURN` | `(err)` | 错误传播 | error_code.h:174 |

### 宏使用示例

```cpp
// 空值检查
CHECK_AND_RETURN_RET_LOG(effect != nullptr && uri != nullptr, 
                         ERR_INPUT_NULL, "uri is null");

// 参数范围检查
CHECK_AND_RETURN_RET_LOG(size > 0 && size <= MAX_RAM_SIZE,
                         ERR_PARAM_INVALID, "size invalid");

// 滤镜数量检查
CHECK_AND_RETURN_RET_LOG(filterChain.size() < MAX_EFILTER_NUMS,
                         ERR_TOO_MANY_FILTERS, "too many filters");
```

## 导出宏

**定义文件**：`frameworks/native/capi/native_common_utils.h:29`

| 宏 | 定义 | 用途 |
|----|------|------|
| `EFFECT_EXPORT` | `__attribute__((visibility("default")))` | 导出 C API 符号 |
| `IMAGE_EFFECT_EXPORT` | - | 内部 API 导出宏 |

**内部 API 导出**：`frameworks/native/utils/dfx/image_effect_marco_define.h:23`

## 滤镜注册宏

**定义文件**：`interfaces/inner_api/native/efilter/efilter_factory.h`

| 宏 | 功能 | 证据 |
|----|------|------|
| `REGISTER_EFILTER_FACTORY(FilterClass)` | 注册滤镜工厂 | efilter_factory.h |
| `AutoRegisterEFilter` | 自动注册辅助类 | efilter_factory.h |
| `EFilterFunction` | 滤镜创建函数指针类型 | efilter_factory.h |

### 使用示例

```cpp
// 在 brightness_efilter.cpp 中
REGISTER_EFILTER_FACTORY(BrightnessEFilter)
```

## EGL 状态枚举

**定义文件**：`render_environment/render_environment.h`

| 枚举值 | 说明 |
|--------|------|
| `EGL_STATUS_UNINITIALIZED` | 未初始化 |
| `EGL_STATUS_INITIALIZED` | 已初始化 |
| `EGL_STATUS_ERROR` | 错误状态 |

## 渲染策略配置

**定义文件**：`efilter/base/render_strategy.h`

| 配置项 | 用途 |
|--------|------|
| `RenderStrategy::Init()` | 初始化渲染策略 |
| `RenderStrategy::GetStrategy()` | 获取渲染策略 |
| `RenderStrategy::SetQuality()` | 设置渲染质量 |

## 调试日志宏

| 宏 | 用途 | 证据 |
|----|------|------|
| `IMG_LOGI` | 信息日志 | - |
| `IMG_LOGE` | 错误日志 | external_loader.cpp:39 |
| `IMG_LOGW` | 警告日志 | - |
| `IMG_LOGD` | 调试日志 | - |

## 功能开关

### 条件编译

| 开关 | 用途 | 启用条件 |
|------|------|----------|
| `VPE_ENABLE` | 视频处理引擎支持 | 定义在 BUILD.gn |
| `HST_ANY_WITH_NO_RTTI` | 禁用 RTTI | 始终启用 |
| `IMAGE_COLORSPACE_FLAG` | 色彩空间标记 | 始终启用 |

### 运行时特性

| 特性 | 描述 | 控制方式 |
|------|------|----------|
| GPU 渲染 | OpenGL 加速 | RenderEnvironment 初始化 |
| 滤镜缓存 | EFilter 缓存 | EFilterCacheNegotiate |
| 内存复用 | Buffer 复用 | EffectMemoryManager |

## 相关文档

| 文档 | 说明 |
|------|------|
| [03_Build_and_Targets.md](./03_Build_and_Targets.md) | 构建配置 |
| [02_Architecture.md](./02_Architecture.md) | 内部架构 |
| [01_N-API_Reference.md](./01_N-API_Reference.md) | N-API 接口 |
