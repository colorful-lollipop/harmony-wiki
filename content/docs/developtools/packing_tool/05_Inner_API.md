# 内部 API 说明

## 核心类设计

### 1. 入口类架构

#### CompressEntrance

**职责**: 打包功能的统一入口

**核心流程**:
```
main() / pack()
  ↓
CommandParser.commandParser()    // 解析参数
  ↓
CompressVerify.commandVerify()   // 验证参数
  ↓
Compressor.compressProcess()     // 执行打包
```

**关键代码** (`adapter/ohos/CompressEntrance.java:96-128`):
```java
public static void main(String[] args) {
    Utility utility = new Utility();
    
    // 1. 解析命令行
    if (!CommandParser.commandParser(utility, args)) {
        System.exit(EXIT_STATUS_EXCEPTION);
    }
    
    // 2. 验证参数
    if (!CompressVerify.commandVerify(utility)) {
        System.exit(EXIT_STATUS_EXCEPTION);
    }
    
    // 3. 执行压缩
    Compressor compressor = new Compressor();
    if (!compressor.compressProcess(utility)) {
        System.exit(EXIT_STATUS_EXCEPTION);
    }
    
    System.exit(EXIT_STATUS_NORMAL);
}
```

#### UncompressEntrance

**职责**: 拆包和解析功能的统一入口

**支持的操作模式**:
- `MODE_HAP`: 拆包 HAP
- `MODE_APP`: 拆包 APP
- `MODE_APPQF`: 拆包 APPQF
- `MODE_HSP`: 拆包 HSP

**解析模式** (`ParseAppMode` 枚举):
- `ALL`: 获取所有信息
- `HAP_LIST`: 只获取 HAP 列表
- `HAP_INFO`: 获取指定 HAP 信息

### 2. 配置类 Utility

**文件**: `adapter/ohos/Utility.java`

**职责**: 集中管理所有打包/拆包参数

**设计特点**:
- 150+ 个字段，覆盖所有命令行参数
- 使用 getter/setter 模式
- 支持路径列表的格式化存储

**核心字段分类**:

```java
// 模式配置
private String mode;                    // 操作模式

// 输入路径
private String jsonPath;                // module.json/config.json
private String resourcesPath;           // 资源路径
private String etsPath;                 // ArkTS 代码路径
private String hapPath;                 // HAP 路径
private String appPath;                 // APP 路径
private String harPath;                 // HAR 路径

// 输出配置
private String outPath;                 // 输出路径
private String forceRewrite;            // 强制覆盖

// 打包选项
private int compressLevel;              // 压缩级别 1-9
private String generateBuildHash;       // 生成构建哈希

// 设备类型
private String deviceType;              // 目标设备类型

// 列表数据（逗号分隔后格式化）
private List<String> formattedHapPathList;
private List<String> formattedHspPathList;
```

### 3. 压缩引擎 Compressor

**文件**: `adapter/ohos/Compressor.java`

**职责**: 执行实际的打包操作

**核心方法**:

#### compressProcess

```java
public boolean compressProcess(Utility utility)
```

**处理流程**:
1. 根据 mode 分发到具体处理方法
2. 处理版本归一化、包归一化等特殊模式
3. 默认流程：创建输出目录 → 并行压缩 → 清理

#### 各模式处理方法

```java
// HAP 打包
private void compressHap(Utility utility) throws BundleException

// HSP 打包
private void compressHsp(Utility utility) throws BundleException

// APP 打包（多 HAP 组合）
private void compressAppMode(Utility utility) throws BundleException

// HQF 打包
private void compressHQFMode(Utility utility) throws BundleException

// APPQF 打包
private void compressAPPQFMode(Utility utility) throws BundleException
```

#### 并行压缩实现

```java
// 使用 Apache Commons Compress 的 ParallelScatterZipCreator
private void compressFileList(List<String> fileList, ...) throws BundleException {
    // 1. 创建并行压缩器
    ParallelScatterZipCreator scatterZipCreator = new ParallelScatterZipCreator();
    
    // 2. 提交压缩任务
    for (String file : fileList) {
        InputStreamSupplier supplier = () -> createInputStream(file);
        scatterZipCreator.addArchiveEntry(zipArchiveEntry, supplier);
    }
    
    // 3. 写入 Zip 目录
    scatterZipCreator.writeTo(zipOut);
}
```

### 4. 解压引擎 Uncompress

**文件**: `adapter/ohos/Uncompress.java`

**职责**: 执行拆包和解析操作

**核心方法**:

#### unpackageProcess

```java
static boolean unpackageProcess(Utility utility)
```

**处理流程**:
1. 验证参数
2. 创建输出目录
3. 根据 mode 分发处理
4. 异常处理和清理

#### 解析方法

```java
// 从路径解析 APP
static UncompressResult uncompressAppByPath(Utility utility)

// 从输入流解析 APP
static UncompressResult uncompressAppByInput(Utility utility, InputStream input)

// 解析 HAP
static UncompressResult uncompressHap(Utility utility)

// 从路径解析 HAP
static UncompressResult uncompressHapByPath(String deviceType, String hapPath)
```

### 5. 验证框架

#### 类层次结构

```
CompressVerify          # 打包参数验证
├── 路径验证
├── 文件存在性检查
└── 参数范围检查

UncompressVerify        # 拆包参数验证
├── 路径验证
├── 文件扩展名检查
└── 模式有效性检查

HapVerify              # HAP 包一致性验证
├── checkHapIsValid()      # 验证 HAP 列表一致性
├── checkFileSizeIsValid() # 验证文件大小限制
└── 检查项：bundleName、version、moduleName、API 版本

AbstractPackValidator  # 验证器抽象基类（新增）
├── validate()             # 模板方法
└── isVerifyValid()        # 子类实现

HapValidator           # HAP SDK 验证器
HspValidator           # HSP SDK 验证器
PackValidatorFactory   # 验证器工厂
```

#### HapVerify 验证逻辑

**文件**: `adapter/ohos/HapVerify.java`

**关键验证项**:

```java
// 1. 检查 bundleName 一致性
private static boolean checkAppFieldsAreSame(...)

// 2. 检查 moduleName 唯一性
private static boolean checkModuleNameIsValid(...)

// 3. 检查 API 版本兼容性
private static boolean checkApiVersion(...)

// 4. 检查原子服务大小限制
public static boolean checkFileSizeIsValid(List<HapVerifyInfo> hapVerifyInfoList)
```

### 6. 工具类

#### FileUtils

**文件**: `adapter/ohos/FileUtils.java`

**核心功能**:

```java
// 文件读取
public static Optional<String> getFileContent(String filePath)
public static byte[] getFileData(String filePath)

// 路径验证
public static boolean matchPattern(String filePath)
// 使用正则: [0-9A-Za-z/].{0,4095}

// 文件操作
public static void deleteFile(String filePath)
public static void deleteDirectory(File file)
public static String getSha256(String filePath)

// Zip 操作
public static void unzip(String srcPath, String destDirPath)
```

#### JsonUtil

**文件**: `adapter/ohos/JsonUtil.java`

**核心功能**:

```java
// JSON 解析（使用 FastJSON）
public static Optional<JSONObject> parseJson(String jsonPath)
public static List<PackInfo> parseHapList(String deviceType, String jsonString)
public static ProfileInfo parseProfileInfo(String jsonString, ...)

// 特定配置解析
public static AppInfo parseAppInfo(JSONObject jsonObject)
public static HapInfo parseHapInfo(JSONObject jsonObject)
```

#### PackageUtil

**文件**: `adapter/ohos/PackageUtil.java`

**核心功能**:

```java
// 多线程压缩
public static void compress(List<String> fileList, String outputPath)

// Zip 操作封装
public static void unzipFile(String zipPath, String destPath)
```

## 数据模型

### Info 类体系

```
AppInfo                 # 应用级信息
├── bundleName          # 包名
├── versionCode         # 版本号
├── versionName         # 版本名
├── targetApiVersion    # 目标 API
└── ...

HapInfo                 # HAP 模块信息
├── name                # 模块名
├── moduleType          # 模块类型
├── deviceType          # 支持设备
├── abilities           # Ability 列表
├── distro              # 分发配置
└── ...

ProfileInfo             # 完整配置
├── hapName             # HAP 名称
├── appInfo             # AppInfo 对象
├── hapInfo             # HapInfo 对象
└── deviceConfig        # 设备配置

PackInfo                # pack.info 信息
├── name                # 包名
├── moduleName          # 模块名
├── moduleType          # 模块类型
└── deviceType          # 设备类型

HapVerifyInfo           # HAP 验证信息
├── bundleName          # 用于一致性检查
├── moduleName          # 用于唯一性检查
├── versionCode         # 用于版本检查
└── ...
```

## 设计模式

### 1. 模板方法模式

**应用**: `AbstractPackValidator`

```java
public abstract class AbstractPackValidator {
    // 模板方法
    public boolean validate(String path) {
        // 1. 前置检查
        if (!preCheck(path)) return false;
        
        // 2. 具体验证（子类实现）
        return isVerifyValid(path);
    }
    
    // 抽象方法
    protected abstract boolean isVerifyValid(String path);
}
```

### 2. 工厂模式

**应用**: `PackValidatorFactory`

```java
public class PackValidatorFactory {
    private static final Map<String, Supplier<AbstractPackValidator>> validators = new HashMap<>();
    
    static {
        validators.put("hap", HapValidator::new);
        validators.put("hsp", HspValidator::new);
    }
    
    public static AbstractPackValidator getValidator(String type) {
        return validators.getOrDefault(type, () -> null).get();
    }
}
```

### 3. 策略模式

**应用**: `ResourcesParser`

```java
public interface ResourcesParser {
    List<ResourceIndexResult> parse(String hapPath);
}

public class ResourcesParserV1 implements ResourcesParser { ... }
public class ResourcesParserV2 implements ResourcesParser { ... }
```

### 4. 建造者模式

**应用**: `Utility` 配置构建

```java
Utility utility = new Utility();
utility.setMode(Utility.MODE_HAP);
utility.setJsonPath("module.json");
utility.setOutPath("output.hap");
// ... 链式设置
```

## 关键算法

### 1. HAP 一致性验证算法

```java
// 检查多个 HAP 是否可以打包到同一个 APP
public static boolean checkHapIsValid(List<HapVerifyInfo> hapVerifyInfos) {
    // 1. 检查 bundleName 一致性
    if (!checkAppFieldsAreSame(hapVerifyInfos, "bundleName")) return false;
    
    // 2. 检查 versionCode 一致性
    if (!checkAppFieldsAreSame(hapVerifyInfos, "versionCode")) return false;
    
    // 3. 检查 moduleName 唯一性
    if (!checkModuleNameIsValid(hapVerifyInfos)) return false;
    
    // 4. 检查 API 版本兼容性
    if (!checkApiVersion(hapVerifyInfos)) return false;
    
    return true;
}
```

### 2. 并行压缩算法

```java
// 使用多线程并行压缩多个文件
public void parallelCompress(List<File> files, OutputStream out) {
    ZipArchiveOutputStream zipOut = new ZipArchiveOutputStream(out);
    ParallelScatterZipCreator creator = new ParallelScatterZipCreator();
    
    for (File file : files) {
        ZipArchiveEntry entry = new ZipArchiveEntry(file.getName());
        InputStreamSupplier supplier = () -> new FileInputStream(file);
        creator.addArchiveEntry(entry, supplier);
    }
    
    try {
        creator.writeTo(zipOut);
    } catch (Exception e) {
        handleError(e);
    }
}
```

### 3. 设备类型过滤算法

```java
// 根据设备类型过滤 HAP 列表
public List<PackInfo> filterByDeviceType(List<PackInfo> infos, String deviceType) {
    return infos.stream()
        .filter(info -> info.deviceType.contains(deviceType))
        .collect(Collectors.toList());
}
```

## 扩展点

### 1. 添加新的打包模式

```java
// 在 Compressor.java 中添加
private void compressNewMode(Utility utility) throws BundleException {
    // 1. 验证参数
    // 2. 收集文件
    // 3. 执行压缩
}

// 在 compressProcess 中分发
case Utility.MODE_NEW:
    compressNewMode(utility);
    break;
```

### 2. 添加新的验证器

```java
// 继承 AbstractPackValidator
public class NewValidator extends AbstractPackValidator {
    @Override
    protected boolean isVerifyValid(String path) {
        // 实现验证逻辑
        return true;
    }
}

// 注册到工厂
validators.put("new", NewValidator::new);
```

### 3. 添加新的解析器

```java
// 实现 ResourcesParser 接口
public class ResourcesParserV3 implements ResourcesParser {
    @Override
    public List<ResourceIndexResult> parse(String hapPath) {
        // 实现解析逻辑
    }
}

// 注册到工厂
```

## 代码规范

### 命名约定

| 类型 | 命名规范 | 示例 |
|-----|---------|------|
| 类名 | 大驼峰 | `CompressEntrance` |
| 方法名 | 小驼峰 | `compressProcess` |
| 常量 | 全大写下划线 | `MODE_HAP` |
| 字段 | 小驼峰 | `bundleName` |

### 错误处理

```java
// 使用 BundleException 抛出业务错误
if (!file.exists()) {
    throw new BundleException("File not found: " + path);
}

// 使用 LOG 记录错误
LOG.error("Error message: " + e.getMessage());

// 返回结果对象
UncompressResult result = new UncompressResult();
result.setResult(false);
result.setMessage("Error description");
```

### 资源管理

```java
// 使用 try-with-resources
try (InputStream is = new FileInputStream(file)) {
    // 使用流
} catch (IOException e) {
    LOG.error("IO error: " + e.getMessage());
}

// 或使用 Utility.closeStream
Utility.closeStream(inputStream);
```
