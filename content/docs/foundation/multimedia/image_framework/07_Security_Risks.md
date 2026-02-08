# 安全风险分析

## 目的

本文档基于代码证据分析 Image Framework 的安全风险，包括攻击面、信任边界和可被利用点。

## 分析范围

| 组件 | 分析内容 |
|------|----------|
| 文件操作 | 路径处理、文件读写 |
| 内存操作 | 缓冲区分配、内存拷贝 |
| 输入验证 | 图像尺寸、格式、元数据 |
| 权限控制 | 系统应用检查、访问控制 |
| IPC/RPC | 跨进程通信 |
| 动态加载 | 插件加载、库加载 |

## 攻击面清单

### 1. 文件系统攻击面

**组件**: ImageSource 文件路径接口

**证据**: `frameworks/innerkitsimpl/stream/src/file_source_stream.cpp:81`

```cpp
bool FileSourceStream::Init()
{
    if (PathToRealPath(pathName_, realPath_)) {
        sourceFilePtr_ = fopen(realPath_.c_str(), "rb");
    }
}
```

**安全措施**:
- ✅ 使用 `PathToRealPath()` (内部调用 `realpath()`) 规范化路径
- ✅ 路径规范化后才进行文件操作

**位置**: `frameworks/innerkitsimpl/utils/src/image_utils.cpp:451`

```cpp
char tmpPath[PATH_MAX] = { 0 };
if (realpath(path.c_str(), tmpPath) == nullptr) {
    IMAGE_LOGE("path to realpath is nullptr");
    return false;
}
```

**风险评估**: **低风险** - 路径遍历攻击已被有效防护

---

### 2. 内存安全攻击面

#### 2.1 整数溢出

**组件**: 图像尺寸计算

**防护措施**: `frameworks/innerkitsimpl/utils/src/image_utils.cpp:627-668`

```cpp
bool ImageUtils::CheckMulOverflow(int32_t width, int32_t bytesPerPixel)
{
    if (width == 0 || bytesPerPixel == 0) {
        return true;
    }
    int32_t rowSize = width * bytesPerPixel;
    if ((rowSize / width) != bytesPerPixel) {
        IMAGE_LOGE("width * bytesPerPixel overflow!");
        return true;
    }
    return false;
}

bool ImageUtils::CheckMulOverflow(int32_t width, int32_t height, int32_t bytesPerPixel)
{
    // 类似的三元乘法溢出检测
}
```

**证据**: `frameworks/innerkitsimpl/codec/src/image_source.cpp:2036-2043`

```cpp
static const uint32_t MAX_SOURCE_SIZE = 300 * 1024 * 1024;  // 300MB

if (data == nullptr || size == 0) {
    IMAGE_LOGE("input data is nullptr or size:%{public}d is zero!", size);
    errorCode = ERR_IMAGE_DATA_ABNORMAL;
    return nullptr;
}
if (size > MAX_SOURCE_SIZE) {
    IMAGE_LOGE("source size:%{public}d is too large!", size);
    errorCode = ERR_IMAGE_DATA_ABNORMAL;
    return nullptr;
}
```

**风险评估**: **低风险** - 已实施多层溢出保护

#### 2.2 缓冲区溢出

**防护措施**: 使用 `memcpy_s`/`memset_s` (安全 CRT)

**证据**: 600+ 处使用 `memcpy_s`

```cpp
// 示例：frameworks/innerkitsimpl/common/src/pixel_map.cpp
if (memcpy_s(dstPixels, bufferSize, srcPixels, bufferSize) != EOK) {
    IMAGE_LOGE("memcpy pixel data failed");
    return ERR_IMAGE_DATA_ABNORMAL;
}
```

**风险评估**: **低风险** - 全面使用安全内存拷贝函数

---

### 3. 输入验证攻击面

#### 3.1 图像尺寸验证

**证据**: `frameworks/innerkitsimpl/utils/src/image_utils.cpp:518-530`

```cpp
constexpr int MAX_DIMENSION = INT32_MAX >> 2;  // ~536M pixels

bool ImageUtils::IsValidImageInfo(const ImageInfo &info)
{
    if (info.size.width <= 0 || info.size.height <= 0 || 
        info.size.width > MAX_DIMENSION || info.size.height > MAX_DIMENSION) {
        IMAGE_LOGE("width(%{public}d) or height(%{public}d) is invalid.", 
                   info.size.width, info.size.height);
        return false;
    }
    return true;
}
```

#### 3.2 PixelMap 大小限制

**证据**: `interfaces/innerkits/include/pixel_map.h:89`

```cpp
constexpr int32_t PIXEL_MAP_MAX_RAM_SIZE = 600 * 1024 * 1024;  // 600MB
```

**证据**: `interfaces/innerkits/include/pixel_map.h:875`

```cpp
static constexpr size_t MAX_IMAGEDATA_SIZE = 128 * 1024 * 1024;  // 128MB
```

**风险评估**: **低风险** - 严格的尺寸和大小限制

---

### 4. 权限控制攻击面

#### 4.1 系统应用检查

**证据**: `frameworks/kits/js/common/image_napi_utils.cpp:330-338`

```cpp
bool ImageNapiUtils::IsSystemApp()
{
#if !defined(CROSS_PLATFORM)
    static bool isSys = Security::AccessToken::TokenIdKit::IsSystemAppByFullTokenID(
        IPCSkeleton::GetSelfTokenID());
    return isSys;
#else
    return false;
#endif
}
```

**用途**: 限制部分敏感 API 仅限系统应用使用

#### 4.2 EXIF GPS 权限

**证据**: `plugins/common/libs/image/libjpegplugin/src/exif_info.cpp:83`

```cpp
constexpr int PERMISSION_GPS_TYPE = 0;
```

**证据**: `frameworks/innerkitsimpl/common/src/image_source.cpp:1043`

EXIF 修改操作检查 FD 写入权限

**风险评估**: **中风险** - 权限检查存在，但需确保所有敏感操作都有检查

---

### 5. IPC/RPC 攻击面

#### 5.1 Parcel 序列化

**组件**: PixelMap, Picture 的 IPC 传输

**证据**: `frameworks/innerkitsimpl/common/src/pixel_map_parcel.cpp:113`

```cpp
if (bufferSize > 0 && bufferSize <= PIXEL_MAP_MAX_RAM_SIZE) {
    // 处理缓冲区
}
```

**证据**: `frameworks/innerkitsimpl/common/src/pixel_map.cpp:2608`

```cpp
SurfaceBuffer *sbBuffer = reinterpret_cast<SurfaceBuffer *>(surfaceBuffer_.GetRefPtr());
```

**风险点**: 
- ⚠️ IPC 数据流需要严格验证
- ⚠️ SurfaceBuffer 引用传递需要安全检查

**风险评估**: **中风险** - IPC 路径需要额外安全审查

---

### 6. 动态加载攻击面

#### 6.1 插件加载

**证据**: `plugins/manager/src/framework/plugin.cpp` (dlopen 使用)

插件通过 `dlopen()` 动态加载，路径来自 `.pluginmeta` 配置

#### 6.2 外部库加载

**证据**: `frameworks/innerkitsimpl/codec/src/image_source.cpp:301-328`

```cpp
// 加载 libtextureSuperCompress.so
void* handle = dlopen("libtextureSuperCompress.so", RTLD_LAZY);
```

**证据**: `plugins/common/libs/image/libextplugin/src/texture_encode/astc_codec.cpp:157-170`

```cpp
// 加载 ASTC 编码库
void* handle = dlopen("libtextureSuperCompress.so", RTLD_NOW);
```

**证据**: `plugins/common/libs/image/libextplugin/src/jpeg_yuv_decoder/yuv_helper.cpp:35-51`

```cpp
// 加载 libyuv.so
yuvLibraryHandle_ = dlopen("libyuv.so", RTLD_NOW);
```

**风险点**:
- ⚠️ **无库签名验证** - 加载前未验证库签名
- ⚠️ **路径信任** - 依赖系统路径安全

**风险评估**: **中高风险** - 动态加载缺乏签名验证

---

### 7. 可被利用点详细分析

#### 可被利用点 #1: 动态库加载无签名验证

**证据**: 
- `image_source.cpp:301-328`
- `astc_codec.cpp:157-170`
- `yuv_helper.cpp:35-51`

**触发路径**:
```
JS: 解码 ASTC 图像或 HEIF 图像
    ↓
Native: 调用解码器
    ↓
Plugin: 尝试硬件解码或 YUV 转换
    ↓
dlopen("libtextureSuperCompress.so")
    或
dlopen("libyuv.so")
```

**影响**: 
- 如果系统库被替换，可能执行恶意代码
- 影响范围：所有使用 ASTC/YUV 转换的图像操作

**修复建议**:
```cpp
// 建议添加签名验证
bool VerifyLibrarySignature(const char* path) {
    // 1. 读取库文件
    // 2. 验证数字签名
    // 3. 验证哈希值
    return isValid;
}

void* SafeDlopen(const char* path) {
    if (!VerifyLibrarySignature(path)) {
        IMAGE_LOGE("Library signature verification failed: %{public}s", path);
        return nullptr;
    }
    return dlopen(path, RTLD_NOW);
}
```

---

#### 可被利用点 #2: 硬编码测试路径

**证据**: `frameworks/kits/js/common/image_receiver_napi.cpp:809`

```cpp
// 硬编码测试路径
std::string path = "/data/receiver/test.jpg";
```

**证据**: `frameworks/kits/taihe/image_receiver_taihe.cpp:240`

```cpp
std::string path = "/data/receiver/test.jpg";
```

**影响**: 
- 测试代码混入生产环境
- 可能导致意外文件写入

**修复建议**: 移除或条件编译测试代码

---

#### 可被利用点 #3: IPC 缓冲区大小验证依赖上层

**证据**: `frameworks/innerkitsimpl/common/src/pixel_map.cpp:3035`

```cpp
sptr<SurfaceBuffer> surfaceBuffer_;
// 直接从 IPC 获取 SurfaceBuffer
```

**风险**: 
- SurfaceBuffer 大小依赖上层验证
- 如果验证不足，可能导致内存问题

**修复建议**: 在 PixelMap 内部添加独立的大小验证

---

#### 可被利用点 #4: EXIF 元数据解析复杂

**组件**: JPEG EXIF 解析器

**证据**: `plugins/common/libs/image/libjpegplugin/src/exif_info.cpp`

**风险**: 
- EXIF 结构复杂，可能存在解析漏洞
- 大 EXIF 数据可能导致内存压力

**缓解措施**: 
- 已实施 MAX_SOURCE_SIZE (300MB) 限制
- 使用 bounds_checking_function 库

---

#### 可被利用点 #5: 插件系统依赖配置文件

**证据**: `.pluginmeta` 文件

**风险**: 
- 插件配置被篡改可能导致加载恶意插件
- 配置文件中类名、路径等未加密

**缓解措施**: 
- 插件位于 `/system/lib64/`（受保护分区）
- 配置文件位于 `/system/etc/image/`（受保护分区）

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  应用层 (Untrusted)                                         │
│  - JS/TS 代码                                               │
│  - 用户输入                                                 │
└───────────────────────┬─────────────────────────────────────┘
                        │ N-API 边界
┌───────────────────────▼─────────────────────────────────────┐
│  N-API 层 (Semi-Trusted)                                    │
│  - 参数验证                                                 │
│  - 权限检查 (IsSystemApp)                                   │
└───────────────────────┬─────────────────────────────────────┘
                        │ Native 边界
┌───────────────────────▼─────────────────────────────────────┐
│  Native 层 (Trusted)                                        │
│  - ImageSource/ImagePacker                                  │
│  - PixelMap 操作                                            │
│  - 内存分配                                                 │
└───────────────────────┬─────────────────────────────────────┘
                        │ IPC 边界
┌───────────────────────▼─────────────────────────────────────┐
│  系统服务层 (System)                                        │
│  - Graphic 服务                                             │
│  - Codec 硬件                                               │
└─────────────────────────────────────────────────────────────┘
```

## 安全建议汇总

| 优先级 | 建议 | 影响 |
|--------|------|------|
| **高** | 为动态库添加签名验证 | 防止恶意库注入 |
| **高** | 移除/隔离测试代码中的硬编码路径 | 防止意外文件操作 |
| **中** | 加强 IPC 数据验证 | 防止 IPC 攻击 |
| **中** | 实施插件配置签名 | 防止配置篡改 |
| **低** | 增加 fuzz 测试覆盖率 | 发现潜在漏洞 |

## 检查范围与局限性

### 已覆盖
- ✅ 文件路径操作
- ✅ 内存分配与拷贝
- ✅ 整数溢出保护
- ✅ 输入验证
- ✅ 权限检查
- ✅ IPC/RPC 接口
- ✅ 动态加载

### 未覆盖/局限性
- ⚠️ 第三方库 (Skia, libjpeg-turbo 等) 内部安全
- ⚠️ 硬件解码器固件安全
- ⚠️ 复杂图像文件的解析器深度分析

## 相关文档

- [N-API 接口](03_NAPI_Reference.md) - API 安全边界
- [架构说明](02_Architecture.md) - 组件关系
- [内部 API](04_Inner_API.md) - 接口详情
