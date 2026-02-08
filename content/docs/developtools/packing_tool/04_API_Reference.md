# 对外 API 参考

## 概述

Packing Tool 提供 Java 编程接口，支持程序化调用打包、拆包、解析功能。

**重要说明**: 本项目是命令行工具，**不提供 N-API 接口**。所有 API 均为 Java 静态方法，可直接调用。

## 快速开始

### Maven/Gradle 依赖

```xml
<!-- 直接使用打包生成的 JAR -->
<dependency>
    <groupId>ohos</groupId>
    <artifactId>packing-tool</artifactId>
    <version>3.2</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/app_packing_tool.jar</systemPath>
</dependency>
```

### 基础用法

```java
import ohos.CompressEntrance;
import ohos.UncompressEntrance;
import ohos.UncompressResult;

// 打包 HAP
boolean packResult = CompressEntrance.pack(
    "/path/to/hap/files",     // hapPath
    "/path/to/pack.info",     // packInfoPath  
    "/path/to/output.hap"     // outPath
);

// 拆包 APP
boolean unpackResult = UncompressEntrance.unpack(
    "/path/to/input.app",     // appPath
    "/path/to/output/dir",    // outPath
    "phone",                  // deviceType
    false                     // unpackApk
);

// 解析 HAP
UncompressResult result = UncompressEntrance.parseHap("/path/to/input.hap");
```

## 打包接口

### CompressEntrance

**文件位置**: `adapter/ohos/CompressEntrance.java`

#### 1. main 方法（命令行入口）

```java
public static void main(String[] args)
```

**功能**: 命令行打包入口。

**参数**:
- `args`: 命令行参数数组

**退出码**:
- `0`: 成功
- `1`: 失败

**示例**:
```bash
java -cp app_packing_tool.jar ohos.CompressEntrance --mode hap --json-path module.json --out-path out.hap
```

#### 2. pack 方法（程序化 API）

```java
public static boolean pack(String hapPath, String packInfoPath, String outPath)
```

**功能**: 打包 HAP 文件为 APP 包。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| `hapPath` | String | 是 | HAP 文件路径或目录，多个用逗号分隔 |
| `packInfoPath` | String | 是 | pack.info 文件路径 |
| `outPath` | String | 是 | 输出 APP 文件路径 |

**返回值**: 
- `true`: 打包成功
- `false`: 打包失败

**异常处理**: 内部捕获异常，返回 false，错误信息输出到日志

**示例**:
```java
boolean result = CompressEntrance.pack(
    "entry.hap,feature.hap",
    "pack.info", 
    "output.app"
);
if (!result) {
    System.err.println("Pack failed!");
}
```

#### 3. getHapSha256 方法

```java
public static String getHapSha256(String hapPath)
```

**功能**: 计算 HAP 文件的 SHA-256 哈希值。

**参数**:
- `hapPath`: HAP 文件路径，必须以 `.hap` 结尾

**返回值**:
- 成功: SHA-256 字符串（64位十六进制）
- 失败: 空字符串

**示例**:
```java
String sha256 = CompressEntrance.getHapSha256("/path/to/app.hap");
System.out.println("SHA-256: " + sha256);
```

## 拆包接口

### UncompressEntrance

**文件位置**: `adapter/ohos/UncompressEntrance.java`

#### 1. main 方法（命令行入口）

```java
public static void main(String[] args)
```

**功能**: 命令行拆包入口。

#### 2. unpack 方法（APP 拆包）

```java
public static boolean unpack(String appPath, String outPath, 
                            String deviceType, boolean unpackApk)
```

**功能**: 解压 APP 包到指定目录。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| `appPath` | String | 是 | APP 文件路径 |
| `outPath` | String | 是 | 输出目录路径 |
| `deviceType` | String | 否 | 设备类型过滤（如 phone、tablet），null 表示不过滤 |
| `unpackApk` | boolean | 否 | 是否解压 APK 文件 |

**返回值**:
- `true`: 拆包成功
- `false`: 拆包失败

**示例**:
```java
boolean result = UncompressEntrance.unpack(
    "input.app",
    "output_dir",
    "phone",    // 只解压支持 phone 的 HAP
    false       // 不解压 APK
);
```

#### 3. unpackHap 方法（HAP 拆包）

```java
public static boolean unpackHap(String hapPath, String outPath, boolean unpackApk)
```

**功能**: 解压单个 HAP 文件。

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| `hapPath` | String | 是 | HAP 文件路径 |
| `outPath` | String | 是 | 输出目录路径 |
| `unpackApk` | boolean | 否 | 是否解压 APK 文件 |

**示例**:
```java
boolean result = UncompressEntrance.unpackHap(
    "entry.hap",
    "output_dir",
    false
);
```

## 解析接口

### UncompressEntrance（续）

#### 4. parseApp 方法（APP 解析）

```java
// 方法 1: 使用枚举模式（推荐）
public static UncompressResult parseApp(String appPath, 
                                        ParseAppMode parseAppMode, 
                                        String hapName)

// 方法 2: 使用字符串模式（已废弃）
@Deprecated
public static UncompressResult parseApp(String appPath, 
                                        String parseMode, 
                                        String deviceType,
                                        String hapName,
                                        String outPath)
```

**功能**: 解析 APP 包，提取包信息而不解压。

**ParseAppMode 枚举**:

| 枚举值 | 说明 |
|-------|------|
| `ALL` | 返回所有信息（HAP 列表 + 每个 HAP 的详细信息）|
| `HAP_LIST` | 只返回 HAP 列表（pack.info）|
| `HAP_INFO` | 返回指定 HAP 的详细信息 |

**参数**:

| 参数 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| `appPath` | String | 是 | APP 文件路径 |
| `parseAppMode` | ParseAppMode | 是 | 解析模式 |
| `hapName` | String | 条件 | HAP 名称（HAP_INFO 模式必填）|

**返回值**: `UncompressResult` 对象，包含解析结果

**示例**:
```java
// 获取 APP 中所有 HAP 信息
UncompressResult result = UncompressEntrance.parseApp(
    "input.app",
    UncompressEntrance.ParseAppMode.ALL,
    null
);

// 获取 HAP 列表
UncompressResult result = UncompressEntrance.parseApp(
    "input.app",
    UncompressEntrance.ParseAppMode.HAP_LIST,
    null
);

// 获取特定 HAP 信息
UncompressResult result = UncompressEntrance.parseApp(
    "input.app",
    UncompressEntrance.ParseAppMode.HAP_INFO,
    "entry.hap"
);
```

#### 5. parseHap 方法（HAP 解析）

```java
public static UncompressResult parseHap(String hapPath)
```

**功能**: 解析单个 HAP 文件，提取配置信息。

**参数**:
- `hapPath`: HAP 或 HSP 文件路径

**返回值**: `UncompressResult` 对象

**示例**:
```java
UncompressResult result = UncompressEntrance.parseHap("entry.hap");
if (result.getResult()) {
    List<ProfileInfo> profiles = result.getProfileInfos();
    for (ProfileInfo profile : profiles) {
        System.out.println("Bundle: " + profile.appInfo.getBundleName());
    }
}
```

#### 6. parseHap (InputStream) 方法

```java
public static UncompressResult parseHap(InputStream input)
```

**功能**: 从输入流解析 HAP。

**参数**:
- `input`: HAP 文件输入流

**适用场景**: 网络传输、内存处理

**示例**:
```java
try (InputStream is = new FileInputStream("entry.hap")) {
    UncompressResult result = UncompressEntrance.parseHap(is);
}
```

#### 7. parseResource 方法

```java
public static List<ResourceIndexResult> parseResource(String hapPath) 
    throws BundleException, IOException
```

**功能**: 解析 HAP 中的资源索引。

**参数**:
- `hapPath`: HAP 文件路径

**返回值**: 资源索引结果列表

**异常**:
- `BundleException`: 解析失败
- `IOException`: IO 错误

#### 8. parseAPPQF 方法

```java
public static APPQFResult parseAPPQF(String appqfPath)
```

**功能**: 解析 APPQF（多 HQF 组合包）文件。

**参数**:
- `appqfPath`: APPQF 文件路径

**返回值**: `APPQFResult` 对象

## 扫描接口

### ScanEntrance

**文件位置**: `adapter/ohos/ScanEntrance.java`

#### main 方法

```java
public static void main(String[] args)
```

**功能**: 命令行扫描入口。

**扫描类型**:
- 重复文件检测
- 文件大小统计
- 后缀名统计

**示例**:
```bash
java -cp app_check_tool.jar ohos.ScanEntrance --mode scan --hap-path input.hap
```

## 数据模型

### UncompressResult

**文件位置**: `adapter/ohos/UncompressResult.java`

**功能**: 拆包/解析操作的结果容器。

**主要方法**:

```java
// 获取操作结果
public boolean getResult()

// 获取错误消息
public String getMessage()

// 获取 pack.info 信息列表
public List<PackInfo> getPackInfos()

// 获取应用配置信息列表
public List<ProfileInfo> getProfileInfos()

// 获取原始配置字符串列表
public List<String> getProfileInfosStr()

// 获取图标路径
public String getIcon()

// 获取标签
public String getLabel()

// 获取包大小（字节）
public long getPackageSize()
```

### ProfileInfo

**文件位置**: `adapter/ohos/ProfileInfo.java`

**功能**: HAP 的完整配置信息。

**主要字段**:

```java
public String hapName;                    // HAP 名称
public AppInfo appInfo;                   // 应用信息
public HapInfo hapInfo;                   // HAP 信息
public Map<String, DeviceConfig> deviceConfig;  // 设备配置
```

### AppInfo

**文件位置**: `adapter/ohos/AppInfo.java`

**功能**: 应用级信息。

**主要字段**:

```java
public String bundleName;                 // 包名
public String vendor;                     // 供应商
public String versionName;                // 版本名称
public String versionCode;                // 版本号
public int targetApiVersion;              // 目标 API 版本
public int compatibleApiVersion;          // 兼容 API 版本
public boolean debug;                     // 调试模式
public String bundleType;                 // 包类型（app/atomicService/shared）
public String icon;                       // 图标路径
public String label;                      // 应用标签
```

### HapInfo

**文件位置**: `adapter/ohos/HapInfo.java`

**功能**: HAP 模块信息。

**主要字段**:

```java
public AppModel appModel;                 // 应用模型（FA/Stage）
public String name;                       // 模块名称
public String moduleType;                 // 模块类型
public List<String> deviceType;           // 支持设备类型
public List<AbilityInfo> abilities;       // Ability 列表
public Distro distro;                     // 分发配置
public List<DependencyItem> dependencies; // 依赖列表
public long compressedSize;               // 压缩后大小
public long originalSize;                 // 原始大小
```

### PackInfo

**文件位置**: `adapter/ohos/PackInfo.java`

**功能**: pack.info 中的包信息。

**主要字段**:

```java
public String name;                       // 包名
public String moduleName;                 // 模块名
public String moduleType;                 // 模块类型
public List<String> deviceType;           // 设备类型
public boolean deliveryWithInstall;       // 随安装分发
```

## 常量定义

### 设备类型常量

```java
// UncompressEntrance.java
public static final String DEVICE_TYPE_DEFAULT = "default";
public static final String DEVICE_TYPE_PHONE = "phone";
public static final String DEVICE_TYPE_TABLET = "tablet";
public static final String DEVICE_TYPE_TV = "tv";
public static final String DEVICE_TYPE_CAR = "car";
public static final String DEVICE_TYPE_SMARTWATCH = "smartWatch";
public static final String DEVICE_TYPE_FITNESSWATCH = "fitnessWatch";
public static final String DEVICE_TYPE_FITNESSBAND = "fitnessBand";
```

### 解析模式常量

```java
// UncompressEntrance.java
public static final String PARSE_MODE_HAPLIST = "hap-list";
public static final String PARSE_MODE_HAPINFO = "hap-info";
public static final String PARSE_MODE_ALL = "all";
```

## 错误处理

### BundleException

**文件位置**: `adapter/ohos/BundleException.java`

**功能**: 包操作异常。

```java
try {
    List<ResourceIndexResult> resources = UncompressEntrance.parseResource(hapPath);
} catch (BundleException e) {
    // 处理解析错误
    System.err.println("Parse failed: " + e.getMessage());
}
```

### 错误码

打包工具使用日志输出错误信息，主要错误类型：

| 错误类型 | 说明 |
|---------|------|
| 参数错误 | 命令行参数缺失或格式错误 |
| 文件不存在 | 输入文件路径错误 |
| 验证失败 | HAP 验证不通过 |
| IO 错误 | 文件读写错误 |
| 解析错误 | JSON 解析失败 |

## 使用示例

### 示例 1: 完整的 APP 解析流程

```java
import ohos.UncompressEntrance;
import ohos.UncompressResult;
import ohos.ProfileInfo;
import ohos.AppInfo;
import ohos.HapInfo;
import ohos.PackInfo;

public class AppAnalyzer {
    public void analyzeApp(String appPath) {
        // 解析 APP 获取所有信息
        UncompressResult result = UncompressEntrance.parseApp(
            appPath,
            UncompressEntrance.ParseAppMode.ALL,
            null
        );
        
        if (!result.getResult()) {
            System.err.println("Parse failed: " + result.getMessage());
            return;
        }
        
        // 输出包信息
        System.out.println("Package size: " + result.getPackageSize() + " bytes");
        System.out.println("Icon: " + result.getIcon());
        System.out.println("Label: " + result.getLabel());
        
        // 遍历 HAP 列表
        for (PackInfo packInfo : result.getPackInfos()) {
            System.out.println("HAP: " + packInfo.name);
            System.out.println("  Module: " + packInfo.moduleName);
            System.out.println("  Type: " + packInfo.moduleType);
            System.out.println("  Device: " + packInfo.deviceType);
        }
        
        // 遍历详细配置
        for (ProfileInfo profile : result.getProfileInfos()) {
            AppInfo app = profile.appInfo;
            HapInfo hap = profile.hapInfo;
            
            System.out.println("Bundle: " + app.bundleName);
            System.out.println("Version: " + app.versionName + " (" + app.versionCode + ")");
            System.out.println("Module: " + hap.name);
            System.out.println("Type: " + hap.moduleType);
            System.out.println("Abilities: " + hap.abilities.size());
        }
    }
}
```

### 示例 2: 批量处理 HAP 文件

```java
import ohos.CompressEntrance;
import ohos.UncompressEntrance;
import ohos.UncompressResult;
import java.io.File;

public class BatchProcessor {
    public void processHapDirectory(String dirPath) {
        File dir = new File(dirPath);
        File[] hapFiles = dir.listFiles((d, name) -> name.endsWith(".hap"));
        
        if (hapFiles == null) return;
        
        for (File hapFile : hapFiles) {
            String hapPath = hapFile.getAbsolutePath();
            
            // 计算 SHA-256
            String sha256 = CompressEntrance.getHapSha256(hapPath);
            System.out.println("File: " + hapFile.getName());
            System.out.println("SHA-256: " + sha256);
            
            // 解析信息
            UncompressResult result = UncompressEntrance.parseHap(hapPath);
            if (result.getResult() && !result.getProfileInfos().isEmpty()) {
                String bundleName = result.getProfileInfos().get(0).appInfo.bundleName;
                System.out.println("Bundle: " + bundleName);
            }
            System.out.println("---");
        }
    }
}
```

### 示例 3: 设备特定的 HAP 提取

```java
import ohos.UncompressEntrance;

public class DeviceExtractor {
    public void extractForDevice(String appPath, String outputDir, String deviceType) {
        // 只提取支持特定设备的 HAP
        boolean result = UncompressEntrance.unpack(
            appPath,
            outputDir,
            deviceType,  // "phone", "tablet", "tv", etc.
            false
        );
        
        if (result) {
            System.out.println("Extracted for " + deviceType + " successfully");
        } else {
            System.err.println("Extraction failed");
        }
    }
}
```

## 注意事项

1. **路径格式**: 支持绝对路径和相对路径，自动处理路径分隔符
2. **编码**: 使用 UTF-8 编码处理文本文件
3. **并发**: Compressor 内部使用并行压缩，但 API 本身是同步的
4. **资源释放**: 自动关闭文件流，无需手动释放
5. **日志**: 使用内部 Log 类输出日志，可通过配置控制级别
