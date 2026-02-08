# 内部 API 文档

## 目的

本文档描述 Image Framework 的 C++ 内部 API，供系统开发者使用。

## 核心类

### ImageSource

**头文件**: `interfaces/innerkits/include/image_source.h`  
**实现**: `frameworks/innerkitsimpl/common/src/image_source.cpp`

#### 创建方法

```cpp
// 从文件路径创建
static std::unique_ptr<ImageSource> CreateImageSource(
    const std::string &pathName,
    const SourceOptions &opts, 
    uint32_t &errorCode);

// 从文件描述符创建
static std::unique_ptr<ImageSource> CreateImageSource(
    const int fd, 
    const SourceOptions &opts,
    uint32_t &errorCode);

// 从内存缓冲区创建
static std::unique_ptr<ImageSource> CreateImageSource(
    const uint8_t *data, uint32_t size,
    const SourceOptions &opts, 
    uint32_t &errorCode,
    bool isUserBuffer = false);

// 从输入流创建
static std::unique_ptr<ImageSource> CreateImageSource(
    std::unique_ptr<std::istream> is,
    const SourceOptions &opts, 
    uint32_t &errorCode);

// 创建渐进式源
static std::unique_ptr<ImageSource> CreateIncrementalImageSource(
    const IncrementalSourceOptions &opts,
    uint32_t &errorCode);
```

#### 解码方法

```cpp
// 创建 PixelMap
std::unique_ptr<PixelMap> CreatePixelMap(
    const DecodeOptions &opts, 
    uint32_t &errorCode);

// 创建指定索引帧
std::unique_ptr<PixelMap> CreatePixelMapEx(
    uint32_t index, 
    const DecodeOptions &opts,
    uint32_t &errorCode);

// 创建渐进式 PixelMap
std::unique_ptr<IncrementalPixelMap> CreateIncrementalPixelMap(
    uint32_t index,
    const DecodeOptions &opts,
    uint32_t &errorCode);

#if !defined(IOS_PLATFORM) && !defined(ANDROID_PLATFORM)
// 创建 Picture (HDR 支持)
std::unique_ptr<Picture> CreatePicture(
    const DecodingOptionsForPicture &opts, 
    uint32_t &errorCode);

// 创建指定索引 Picture
std::unique_ptr<Picture> CreatePictureAtIndex(
    uint32_t index, 
    uint32_t &errorCode);

// 创建缩略图
std::unique_ptr<PixelMap> CreateThumbnail(
    const DecodingOptionsForThumbnail &opts,
    uint32_t &errorCode);
#endif

// 更新渐进式数据
uint32_t UpdateData(const uint8_t *data, uint32_t size, bool isCompleted);
```

#### 信息获取

```cpp
// 获取图像信息
uint32_t GetImageInfo(ImageInfo &imageInfo);
uint32_t GetImageInfo(uint32_t index, ImageInfo &imageInfo);

// 获取源信息
const SourceInfo &GetSourceInfo(uint32_t &errorCode);

// 获取图像属性
uint32_t GetImagePropertyInt(uint32_t index, const std::string &key, int32_t &value);
uint32_t GetImagePropertyString(uint32_t index, const std::string &key, std::string &value);

// 修改属性
uint32_t ModifyImageProperty(uint32_t index, const std::string &key, 
    const std::string &value, const std::string &path);
```

#### 动图支持

```cpp
// 获取动图帧数
uint32_t GetFrameCount(uint32_t &errorCode);

// 获取帧延迟
std::unique_ptr<std::vector<int32_t>> GetDelayTime(uint32_t &errorCode);

// 获取帧处理方式
std::unique_ptr<std::vector<int32_t>> GetDisposalType(uint32_t &errorCode);

// 获取循环次数
int32_t GetLoopCount(uint32_t &errorCode);
```

### ImagePacker

**头文件**: `interfaces/innerkits/include/image_packer.h`

#### 方法

```cpp
class ImagePacker {
public:
    // 获取支持的编码格式
    static uint32_t GetSupportedFormats(std::set<std::string> &formats);
    
    // 开始打包（缓冲区）
    uint32_t StartPacking(uint8_t *data, uint32_t maxSize, const PackOption &option);
    
    // 开始打包（文件路径）
    uint32_t StartPacking(const std::string &filePath, const PackOption &option);
    
    // 开始打包（文件描述符）
    uint32_t StartPacking(const int &fd, const PackOption &option);
    
    // 开始打包（输出流）
    uint32_t StartPacking(std::ostream &outputStream, const PackOption &option);
    
    // 添加图像
    uint32_t AddImage(PixelMap &pixelMap);
    uint32_t AddImage(ImageSource &source);
    uint32_t AddImage(ImageSource &source, uint32_t index);
    
#if !defined(IOS_PLATFORM) && !defined(ANDROID_PLATFORM)
    // 添加 Picture
    uint32_t AddPicture(Picture &picture);
#endif
    
    // 完成打包
    uint32_t FinalizePacking();
    uint32_t FinalizePacking(int64_t &packedSize);
};
```

#### PackOption 结构

```cpp
struct PackOption {
    std::string format;                    // 输出格式
    uint8_t quality = 100;                 // 压缩质量 0-100
    uint32_t numberHint = 1;               // 图像数量提示
    EncodeDynamicRange desiredDynamicRange = EncodeDynamicRange::SDR;
    uint16_t loop = 0;                     // 循环次数（GIF）
    std::vector<uint16_t> delayTimes = {};    // 帧延迟（GIF）
    std::vector<uint8_t> disposalTypes = {};  // 帧处理方式（GIF）
    bool needsPackProperties = false;      // 是否打包属性
    bool isEditScene = true;               // 是否为编辑场景
};
```

### PixelMap

**头文件**: `interfaces/innerkits/include/pixel_map.h`

#### 创建方法

```cpp
class PixelMap : public Parcelable {
public:
    // 从像素数据创建
    static std::unique_ptr<PixelMap> Create(
        const uint32_t *colors, uint32_t colorLength,
        const InitializationOptions &opts);
    
    // 从像素数据（带偏移和步长）
    static std::unique_ptr<PixelMap> Create(
        const uint32_t *colors, uint32_t colorLength, 
        int32_t offset, int32_t stride,
        const InitializationOptions &opts);
    
    // 从 InitializationOptions 创建
    static std::unique_ptr<PixelMap> Create(
        const InitializationOptions &opts);
    
    // 从现有 PixelMap 创建
    static std::unique_ptr<PixelMap> Create(
        PixelMap &source, const InitializationOptions &opts);
    
    // 从现有 PixelMap（带裁剪区域）
    static std::unique_ptr<PixelMap> Create(
        PixelMap &source, const Rect &srcRect,
        const InitializationOptions &opts);
    
    // 从 ASTC 转换
    static std::unique_ptr<PixelMap> ConvertFromAstc(
        PixelMap *source, uint32_t &errorCode,
        PixelFormat destFormat);
};
```

#### 像素操作

```cpp
// 获取像素地址
const uint8_t *GetPixel(int32_t x, int32_t y);
const uint8_t *GetPixel8(int32_t x, int32_t y);
const uint16_t *GetPixel16(int32_t x, int32_t y);
const uint32_t *GetPixel32(int32_t x, int32_t y);

// 获取/设置像素颜色
bool GetARGB32Color(int32_t x, int32_t y, uint32_t &color);
uint32_t WritePixel(const Position &pos, const uint32_t &color);

// 批量读写
uint32_t ReadPixels(const RWPixelsOptions &opts);
uint32_t WritePixels(const RWPixelsOptions &opts);
```

#### 图像变换

```cpp
// 缩放
void scale(float xAxis, float yAxis);
void scale(float xAxis, float yAxis, const AntiAliasingOption &option);
bool resize(float xAxis, float yAxis);

// 旋转
void rotate(float degrees);

// 平移
void translate(float xAxis, float yAxis);

// 翻转
void flip(bool xAxis, bool yAxis);

// 裁剪
uint32_t crop(const Rect &rect);
```

#### 属性获取

```cpp
int32_t GetWidth();
int32_t GetHeight();
int32_t GetPixelBytes();
int32_t GetRowBytes();
int32_t GetByteCount();
uint32_t GetAllocationByteCount();
PixelFormat GetPixelFormat();
AlphaType GetAlphaType();
AllocatorType GetAllocatorType();
bool IsEditable();
bool IsModifiable();
```

### Picture

**头文件**: `interfaces/innerkits/include/picture.h`

```cpp
class Picture : public Parcelable {
public:
    // 获取主图
    std::shared_ptr<PixelMap> GetMainPixelMap();
    
    // 设置主图
    void SetMainPixelMap(std::shared_ptr<PixelMap> pixelMap);
    
    // 获取 HDR 合成图
    std::shared_ptr<PixelMap> GetHdrComposedPixelMap();
    
    // 获取辅助图
    std::shared_ptr<AuxiliaryPicture> GetAuxiliaryPicture(
        AuxiliaryPictureType type);
    
    // 设置辅助图
    void SetAuxiliaryPicture(
        std::shared_ptr<AuxiliaryPicture> auxiliaryPicture);
    
    // 删除辅助图
    void DropAuxiliaryPicture(AuxiliaryPictureType type);
    
    // 获取元数据
    std::shared_ptr<ImageMetadata> GetMetadata(MetadataType type);
    
    // 设置元数据
    void SetMetadata(std::shared_ptr<ImageMetadata> metadata);
    
    // 序列化/反序列化
    bool Marshalling(Parcel &data) const override;
    static Picture *Unmarshalling(Parcel &data);
};
```

## 类型定义

**头文件**: `interfaces/innerkits/include/image_type.h`

### 枚举类型

```cpp
// 像素格式
enum class PixelFormat : int32_t {
    UNKNOWN = 0,
    RGBA_8888 = 1,
    BGRA_8888 = 2,
    RGB_565 = 3,
    RGBA_F16 = 4,
    NV21 = 5,           // YUV420SP
    NV12 = 6,           // YUV420SP
    RGBA_1010102 = 10,
    YCBCR_P010 = 11,
    YCRCB_P010 = 12,
    ASTC_4x4 = 25,
    // ... 更多格式
};

// Alpha 类型
enum class AlphaType : int32_t {
    IMAGE_ALPHA_TYPE_UNKNOWN = 0,
    IMAGE_ALPHA_TYPE_OPAQUE = 1,      // 不透明
    IMAGE_ALPHA_TYPE_PREMUL = 2,      // 预乘
    IMAGE_ALPHA_TYPE_UNPREMUL = 3,    // 非预乘
};

// 内存分配器类型
enum class AllocatorType : int32_t {
    DEFAULT = 0,
    HEAP_ALLOC = 1,       // 堆内存
    SHARE_MEM_ALLOC = 2,  // 共享内存
    CUSTOM_ALLOC = 3,     // 自定义分配
    DMA_ALLOC = 4,        // DMA (SurfaceBuffer)
};

// 缩放模式
enum class ScaleMode : int32_t {
    FIT_TARGET_SIZE = 0,      // 适应目标尺寸
    OVER_SCALE_CROP = 1,      // 居中裁剪
    RESIZE_NEAREST_NEIGHBOR = 2,  // 最近邻
};

// 抗锯齿选项
enum class AntiAliasingOption : int32_t {
    NONE = 0,
    LOW = 1,
    MEDIUM = 2,
    HIGH = 3,
};

// HDR 类型
enum class ImageHdrType : int32_t {
    SDR = 0,
    HDR = 1,              // 单 HDR
    HDR_DUAL = 2,         // 双 HDR
    HDR_VIVID_DUAL = 3,   // 鲜艳 HDR 双图
    HDR_VIVID_SINGLE = 4, // 鲜艳 HDR 单图
    HDR_ISO_DUAL = 5,     // ISO HDR 双图
    HDR_ISO_SINGLE = 6,   // ISO HDR 单图
};

// 辅助图类型
enum class AuxiliaryPictureType : int32_t {
    NONE = 0,
    GAINMAP = 1,          // 增益图
    DEPTH_MAP = 2,        // 深度图
    UNREFOCUS_MAP = 3,    // 失焦图
    LINEAR_MAP = 4,       // 线性图
    FRAGMENT_MAP = 5,     // 碎片图
};

// 元数据类型
enum class MetadataType : int32_t {
    EXIF = 0,
    FRAGMENT = 1,
    GIF = 2,
    HEIF = 3,
    DNG = 4,
    MAKER_NOTE = 5,
};
```

### 结构体

```cpp
// 尺寸
struct Size {
    int32_t width;
    int32_t height;
};

// 矩形区域
struct Rect {
    int32_t left;
    int32_t top;
    int32_t width;
    int32_t height;
};

// 位置
struct Position {
    int32_t x;
    int32_t y;
};

// 图像信息
struct ImageInfo {
    Size size;
    ColorSpace colorSpace;
    PixelFormat pixelFormat;
    AlphaType alphaType;
    bool isHdr = false;
};

// 初始化选项
struct InitializationOptions {
    Size size;
    PixelFormat srcPixelFormat = PixelFormat::BGRA_8888;
    PixelFormat pixelFormat = PixelFormat::UNKNOWN;
    AlphaType alphaType = AlphaType::IMAGE_ALPHA_TYPE_UNKNOWN;
    ScaleMode scaleMode = ScaleMode::FIT_TARGET_SIZE;
    AllocatorType allocatorType = AllocatorType::DEFAULT;
    bool editable = false;
    bool useSourceIfMatch = false;
    bool useDMA = false;
};

// 解码选项
struct DecodeOptions {
    Rect CropRect;
    Size desiredSize;
    int32_t rotateNewDegrees = 0;
    PixelFormat desiredPixelFormat = PixelFormat::UNKNOWN;
    bool editable = false;
    int32_t sampleSize = 1;
    AllocatorType allocatorType = AllocatorType::DEFAULT;
    bool useSourceIfMatch = false;
    ScaleMode scaleMode = ScaleMode::FIT_TARGET_SIZE;
    int32_t fitDensity = 0;
    bool isHdrDecoderNeed = false;
};

// 读写像素选项
struct RWPixelsOptions {
    const uint8_t *pixels = nullptr;
    uint64_t bufferSize = 0;
    uint32_t offset = 0;
    uint32_t stride = 0;
    Rect region;
    PixelFormat pixelFormat = PixelFormat::BGRA_8888;
};
```

## 错误码

```cpp
// 通用错误码
constexpr uint32_t SUCCESS = 0;
constexpr uint32_t ERR_IMAGE_BASE = 0x00010000;
constexpr uint32_t ERR_IMAGE_INIT_ABNORMAL = ERR_IMAGE_BASE + 1;
constexpr uint32_t ERR_IMAGE_DATA_ABNORMAL = ERR_IMAGE_BASE + 2;
constexpr uint32_t ERR_IMAGE_TOO_LARGE = ERR_IMAGE_BASE + 3;
constexpr uint32_t ERR_IMAGE_TRANSFORM = ERR_IMAGE_BASE + 4;
constexpr uint32_t ERR_IMAGE_COLOR_CONVERT = ERR_IMAGE_BASE + 5;
constexpr uint32_t ERR_IMAGE_CROP = ERR_IMAGE_BASE + 6;
constexpr uint32_t ERR_IMAGE_SOURCE_DATA = ERR_IMAGE_BASE + 7;
constexpr uint32_t ERR_IMAGE_SOURCE_FORMAT = ERR_IMAGE_BASE + 8;
constexpr uint32_t ERR_IMAGE_DECODER_UNSUPPORT = ERR_IMAGE_BASE + 9;
constexpr uint32_t ERR_IMAGE_PLUGIN_REGISTER_FAILED = ERR_IMAGE_BASE + 10;
constexpr uint32_t ERR_IMAGE_PLUGIN_LOAD_FAILED = ERR_IMAGE_BASE + 11;
```

## 插件接口

### 解码器接口

**头文件**: `plugins/manager/include/image/abs_image_decoder.h`

```cpp
class AbsImageDecoder : public PluginClassBase {
public:
    DECLARE_INTERFACE(AbsImageDecoder, IMAGE_DECODER_IID)
    
    // 设置数据源
    virtual void SetSource(InputDataStream &source) = 0;
    
    // 设置解码选项
    virtual void SetDecodeOptions(uint32_t index, 
        const PixelDecodeOptions &opts, PlImageInfo &info) = 0;
    
    // 解码
    virtual uint32_t Decode(uint32_t index, DecodeContext &context) = 0;
    
    // 获取图像信息
    virtual uint32_t GetImageInfo(uint32_t index, ImageInfo &info) = 0;
    
    // 获取图像属性
    virtual uint32_t GetImagePropertyString(uint32_t index, 
        const std::string &key, std::string &value) = 0;
    
    // 设置图像属性
    virtual uint32_t SetImageProperty(uint32_t index, 
        const std::string &key, const std::string &value) = 0;
    
    // 释放资源
    virtual void Release() {}
};
```

### 编码器接口

**头文件**: `plugins/manager/include/image/abs_image_encoder.h`

```cpp
class AbsImageEncoder : public PluginClassBase {
public:
    DECLARE_INTERFACE(AbsImageEncoder, IMAGE_ENCODER_IID)
    
    // 获取编码器名称
    virtual std::string GetEncoderName() = 0;
    
    // 开始编码
    virtual uint32_t StartEncode(OutputDataStream &output, 
        const PlEncodeOptions &options) = 0;
    
    // 添加图像
    virtual uint32_t AddImage(Media::PixelMap &pixelMap) = 0;
    
    // 完成编码
    virtual uint32_t FinalizeEncode() = 0;
    
    // 设置图像属性
    virtual uint32_t SetImageProperty(uint32_t index, 
        const std::string &key, const std::string &value) = 0;
};
```

### 格式检测接口

**头文件**: `plugins/manager/include/pluginbase/abs_image_format_agent.h`

```cpp
class AbsImageFormatAgent : public PluginClassBase {
public:
    DECLARE_INTERFACE(AbsImageFormatAgent, IMAGE_FORMAT_AGENT_IID)
    
    // 检查格式
    virtual bool CheckFormat(const void *header, int32_t headerSize) = 0;
    
    // 获取格式类型
    virtual std::string GetFormatType() = 0;
};
```

## 内存管理

### MemoryManager

**头文件**: `frameworks/innerkitsimpl/common/include/memory_manager.h`

```cpp
class MemoryManager {
public:
    // 创建内存
    static std::unique_ptr<AbsMemory> CreateMemory(
        AllocatorType type, 
        uint32_t capacity,
        uint32_t &errorCode);
};
```

### AbsMemory

```cpp
class AbsMemory {
public:
    virtual void *GetData() = 0;
    virtual void *GetFd() = 0;
    virtual uint32_t GetCapacity() = 0;
    virtual bool Write(const uint8_t *data, uint32_t size, uint32_t offset) = 0;
};
```

## 相关文档

- [N-API 接口](03_NAPI_Reference.md) - JS 层 API
- [架构说明](02_Architecture.md) - 系统设计
- [GN 构建](05_GN_Targets.md) - 构建配置
