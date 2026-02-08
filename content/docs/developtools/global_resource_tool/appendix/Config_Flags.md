# 关键宏和配置项

## 目的与适用范围

本文档汇总 `global_resource_tool` 中的关键宏定义、常量和配置项。

**适用对象**: 开发者、配置人员

---

## 版本信息

### 工具版本

**定义** (`include/resource_data.h:53`):
```cpp
static const std::string RESTOOL_VERSION = { " 6.1.0.003" };
static const std::string RESTOOL_NAME = "Restool";
static const std::string RESTOOLV2_NAME = "RestoolV2";
```

### 文件格式版本

| 常量 | 值 | 说明 |
|------|-----|------|
| `VERSION_MAX_LEN` | 128 | 版本字符串最大长度 |

---

## 路径常量

### 目录名称

**定义** (`include/resource_data.h:30-37`):
```cpp
const static std::string RESOURCES_DIR = "resources";
const static std::string CACHES_DIR = ".caches";
const static std::string RAW_FILE_DIR = "rawfile";
const static std::string RES_FILE_DIR = "resfile";
```

### 配置文件名

| 常量 | 值 | 说明 |
|------|-----|------|
| `CONFIG_JSON` | "config.json" | FA 模型配置文件 |
| `MODULE_JSON` | "module.json" | Stage 模型配置文件 |
| `ID_DEFINED_FILE` | "id_defined.json" | ID 定义文件 |
| `RESOURCE_INDEX_FILE` | "resources.index" | 资源索引文件 |
| `JSON_EXTENSION` | ".json" | JSON 文件扩展名 |

### 路径分隔符

**定义** (`include/resource_data.h:39-45`):
```cpp
#ifdef __WIN32
const static std::string SEPARATOR_FILE = "\\";
#else
const static std::string SEPARATOR_FILE = "/";
#endif
const static std::string SEPARATOR = "/";
const static std::string WIN_SEPARATOR = "\\";
const static std::string LONG_PATH_HEAD = "\\\\?\\";  // Windows 长路径前缀
```

---

## 资源类型

### ResType 枚举

**定义** (`include/resource_data.h:85-104`):
```cpp
enum class ResType {
    ELEMENT = 0,
    RAW = 6,
    INTEGER = 8,
    STRING = 9,
    STRARRAY = 10,
    INTARRAY = 11,
    BOOLEAN = 12,
    COLOR = 14,
    ID = 15,
    THEME = 16,
    PLURAL = 17,
    FLOAT = 18,
    MEDIA = 19,
    PROF = 20,
    PATTERN = 22,
    SYMBOL = 23,
    RES = 24,
    INVALID_RES_TYPE = -1,
};
```

### 资源类型映射

**文件类型映射** (`include/resource_data.h:242-251`):
```cpp
const std::map<std::string, ResType> g_copyFileMap = {
    { RAW_FILE_DIR, ResType::RAW },
    { RES_FILE_DIR, ResType::RES },
};

const std::map<std::string, ResType> g_fileClusterMap = {
    { "element", ResType::ELEMENT },
    { "media", ResType::MEDIA },
    { "profile", ResType::PROF },
};
```

**内容类型映射** (`include/resource_data.h:253-266`):
```cpp
const std::map<std::string, ResType> g_contentClusterMap = {
    { "id", ResType::ID },
    { "integer", ResType::INTEGER },
    { "string", ResType::STRING },
    { "strarray", ResType::STRARRAY },
    { "intarray", ResType::INTARRAY },
    { "color", ResType::COLOR },
    { "plural", ResType::PLURAL },
    { "boolean", ResType::BOOLEAN },
    { "pattern", ResType::PATTERN },
    { "theme", ResType::THEME },
    { "float", ResType::FLOAT },
    { "symbol", ResType::SYMBOL }
};
```

---

## 限定词类型

### KeyType 枚举

**定义** (`include/resource_data.h:69-83`):
```cpp
enum class KeyType {
    LANGUAGE = 0,
    REGION = 1,
    RESOLUTION = 2,
    ORIENTATION = 3,
    DEVICETYPE = 4,
    SCRIPT = 5,
    NIGHTMODE = 6,
    MCC = 7,
    MNC = 8,
    INPUTDEVICE = 10,
    KEY_TYPE_MAX,
    OTHER,
};
```

### 限定词字符串映射

**定义** (`include/resource_data.h:268-279`):
```cpp
const std::map<KeyType, std::string> g_keyTypeToStrMap = {
    {KeyType::MCC, "mcc"},
    {KeyType::MNC, "mnc"},
    {KeyType::LANGUAGE, "language"},
    {KeyType::SCRIPT, "script"},
    {KeyType::REGION, "region"},
    {KeyType::ORIENTATION, "orientation"},
    {KeyType::RESOLUTION, "density"},
    {KeyType::DEVICETYPE, "device"},
    {KeyType::NIGHTMODE, "colorMode"},
    {KeyType::INPUTDEVICE, "inputDevice"},
};
```

---

## 设备类型

### DeviceType 枚举

**定义** (`include/resource_data.h:116-124`):
```cpp
enum class DeviceType {
    PHONE = 0,
    TABLET = 1,
    CAR = 2,
    TV = 4,
    WEARABLE = 6,
    TWOINONE = 7,
};
```

**设备类型映射** (`include/resource_data.h:185-192`):
```cpp
const std::map<std::string, DeviceType> g_deviceMap = {
    { "phone", DeviceType::PHONE },
    { "tablet", DeviceType::TABLET },
    { "car", DeviceType::CAR },
    { "tv", DeviceType::TV },
    { "wearable", DeviceType::WEARABLE },
    { "2in1", DeviceType::TWOINONE },
};
```

---

## 分辨率类型

### ResolutionType 枚举

**定义** (`include/resource_data.h:126-133`):
```cpp
enum class ResolutionType {
    SDPI = 120,
    MDPI = 160,
    LDPI = 240,
    XLDPI = 320,
    XXLDPI = 480,
    XXXLDPI = 640,
};
```

**分辨率映射** (`include/resource_data.h:194-201`):
```cpp
const std::map<std::string, ResolutionType> g_resolutionMap = {
    { "sdpi", ResolutionType::SDPI },
    { "mdpi",  ResolutionType::MDPI },
    { "ldpi",  ResolutionType::LDPI },
    { "xldpi", ResolutionType::XLDPI },
    { "xxldpi", ResolutionType::XXLDPI },
    { "xxxldpi", ResolutionType::XXXLDPI },
};
```

---

## 资源 ID 范围

### ID 起始值

**定义** (`src/cmd/package_parser.cpp:134`):
```cpp
startId = ((it - moduleNames.begin()) + 1) * 0x01000000;
```

**有效范围** (`src/cmd/package_parser.cpp:303`):
```cpp
if ((id >= 0x01000000 && id < 0x06ffffff) || 
    (id >= 0x08000000 && id < 0xffffffff))
```

| 范围 | 说明 |
|------|------|
| 0x01000000 - 0x06FFFFFF | 应用资源 ID |
| 0x08000000 - 0xFFFFFFFF | 扩展资源 ID |
| 0x07000000 - 0x07FFFFFF | 保留/系统资源 |

### ResourceIdCluster 枚举

**定义** (`include/resource_data.h:145-149`):
```cpp
enum class ResourceIdCluster {
    RES_ID_APP = 0,    // 应用资源
    RES_ID_SYS,        // 系统资源
    RES_ID_TYPE_MAX,
};
```

---

## 线程池配置

### 默认线程数

**定义** (`include/resource_data.h:55`):
```cpp
constexpr static int DEFAULT_POOL_SIZE = 8;
```

---

## 缓冲区大小

### 错误信息缓冲区

**定义** (`include/restool_errors.h:31-32`):
```cpp
constexpr uint16_t BUFFER_SIZE = 4096;
constexpr uint16_t BUFFER_SIZE_SMALL = 128;
```

---

## API 版本常量

### 最低支持版本

**定义** (`include/resource_data.h:59-61`):
```cpp
const static int MIN_SUPPORT_NEW_MODULE_API_VERSION = 20;
const static int MIN_SUPPORT_TS_HEADER_API_VERSION = 23;
const static int API_VERSION_DIVISOR = 1000;
```

| 常量 | 值 | 说明 |
|------|-----|------|
| `MIN_SUPPORT_NEW_MODULE_API_VERSION` | 20 | 支持新模块类型的最低 API 版本 |
| `MIN_SUPPORT_TS_HEADER_API_VERSION` | 23 | 支持 TS 头文件的最低 API 版本 |

---

## 打包类型

### PackType 枚举

**定义** (`include/resource_data.h:106-109`):
```cpp
enum class PackType {
    NORMAL = 1,    // 普通打包
    OVERLAP        // 叠加打包
};
```

---

## 忽略类型

### IgnoreType 枚举

**定义** (`include/resource_data.h:63-67`):
```cpp
enum class IgnoreType {
    IGNORE_FILE,   // 忽略文件
    IGNORE_DIR,    // 忽略目录
    IGNORE_ALL     // 忽略全部
};
```

---

## 命令行选项

### Option 枚举

**定义** (`include/resource_data.h:151-178`):
```cpp
enum Option {
    END = -1,
    IDS = 1,
    DEFINED_IDS = 2,
    DEPENDENTRY = 3,
    ICON_CHECK = 4,
    TARGET_CONFIG = 5,
    DEFINED_SYSIDS = 6,
    COMPRESSED_CONFIG = 7,
    THREAD = 8,
    IGNORED_FILE = 9,
    IGNORED_PATH = 10,
    STARTID = 'e',
    FORCEWRITE = 'f',
    HELP = 'h',
    INPUTPATH = 'i',
    JSON = 'j',
    FILELIST = 'l',
    MODULES = 'm',
    OUTPUTPATH = 'o',
    PACKAGENAME = 'p',
    RESHEADER = 'r',
    VERSION = 'v',
    APPEND = 'x',
    COMBINE = 'z',
    // ...
};
```

---

## 平台宏定义

### 编译时宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `__LINUX__` | BUILD.gn:86 | Linux 平台 |
| `__MAC__` | BUILD.gn:89 | macOS 平台 |
| `_WIN32` | 代码中自动定义 | Windows 平台 |
| `__WIN32` | 代码中自动定义 | Windows 平台（内部使用） |

---

## 图标尺寸配置

### 标准图标尺寸

**定义** (`include/resource_data.h:302-315`):
```cpp
const std::map<std::string, std::vector<uint32_t>> g_normalIconMap = {
    { "sdpi-phone", {41, 144} },
    { "mdpi-phone", {54, 192} },
    { "ldpi-phone", {81, 288} },
    { "xldpi-phone", {108, 384} },
    { "xxldpi-phone", {162, 576} },
    { "xxxldpi-phone", {216, 768} },
    // ... 更多配置
};
```

---

## 相关文档

- [项目概览](../00_Overview.md) - 项目定位
- [内部接口](../04_Internal_API.md) - 模块接口
