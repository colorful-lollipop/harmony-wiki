# 架构说明

## 系统架构

### 整体架构图

```mermaid
graph TB
    subgraph Entry[入口层]
        E1[CompressEntrance] -->|打包| P1[Compressor]
        E2[UncompressEntrance] -->|拆包| P2[Uncompress]
        E3[ScanEntrance] -->|扫描| P3[Scan]
    end
    
    subgraph Process[处理层]
        P1 --> V1[CompressVerify]
        P2 --> V2[UncompressVerify]
        P3 --> V3[ScanVerify]
    end
    
    subgraph Verify[验证层]
        V1 --> H1[HapVerify]
        V2 --> H2[HapVerify]
        V1 --> F1[ValidatorFactory]
    end
    
    subgraph Data[数据层]
        D1[Utility] -->|配置| P1
        D2[Info Classes] -->|数据模型| P1
        D3[JSON Utils] -->|解析| P1
    end
    
    subgraph Utils[工具层]
        U1[FileUtils] -->|文件操作| P1
        U2[PackageUtil] -->|压缩| P1
        U3[JsonUtil] -->|JSON| P1
    end
```

## 组件架构

### Java 实现架构

```mermaid
classDiagram
    class CompressEntrance {
        +main(String[] args)
        +pack(String, String, String) boolean
        +getHapSha256(String) String
    }
    
    class UncompressEntrance {
        +main(String[] args)
        +unpack(String, String, String, boolean) boolean
        +unpackHap(String, String, boolean) boolean
        +parseApp(String, ParseAppMode, String) UncompressResult
        +parseHap(String) UncompressResult
    }
    
    class ScanEntrance {
        +main(String[] args)
    }
    
    class Utility {
        -String mode
        -String jsonPath
        -String outPath
        +getMode() String
        +setMode(String)
    }
    
    class Compressor {
        +compressProcess(Utility) boolean
        +compressHap(Utility)
        +compressHsp(Utility)
        +compressApp(Utility)
    }
    
    class Uncompress {
        +unpackageProcess(Utility) boolean
        +uncompressHap(Utility) UncompressResult
        +parseApp(Utility) UncompressResult
    }
    
    class CompressVerify {
        +commandVerify(Utility) boolean
        +isPathValid(String, boolean, String) boolean
    }
    
    class HapVerify {
        +checkHapIsValid(List) boolean
        +checkFileSizeIsValid(List) boolean
    }
    
    class FileUtils {
        +getFileData(String) byte[]
        +matchPattern(String) boolean
        +deleteFile(String)
    }
    
    CompressEntrance --> Utility
    CompressEntrance --> Compressor
    CompressEntrance --> CompressVerify
    UncompressEntrance --> Utility
    UncompressEntrance --> Uncompress
    Compressor --> HapVerify
    Compressor --> FileUtils
    Uncompress --> FileUtils
```

### C++ 实现架构

```mermaid
classDiagram
    class ShellCommand {
        +ExecCommand() string
        +RunAsHelpCommand()
        +RunAsPackCommand()
        +RunAsUnpackCommand()
    }
    
    class Packager {
        <<abstract>>
        +InitAllowedParam()
        +PreProcess() boolean
        +Process() boolean
        +PostProcess() boolean
    }
    
    class HapPackager {
        +Pack() boolean
    }
    
    class HspPackager {
        +Pack() boolean
    }
    
    ShellCommand --> Packager
    Packager <|-- HapPackager
    Packager <|-- HspPackager
```

## 数据流

### 打包流程数据流

```mermaid
sequenceDiagram
    participant User as 用户/IDE
    participant CE as CompressEntrance
    participant CP as CommandParser
    participant CV as CompressVerify
    participant Comp as Compressor
    participant HV as HapVerify
    participant PU as PackageUtil
    
    User->>CE: 调用 main(args)
    CE->>CP: commandParser(utility, args)
    CP-->>CE: 解析结果
    
    CE->>CV: commandVerify(utility)
    CV->>CV: 验证参数合法性
    CV-->>CE: 验证结果
    
    CE->>Comp: compressProcess(utility)
    
    alt MODE_HAP
        Comp->>Comp: compressHap()
        Comp->>HV: 验证 HAP 配置
    else MODE_HSP
        Comp->>Comp: compressHsp()
    else MODE_APP
        Comp->>Comp: compressApp()
        Comp->>HV: 验证多 HAP 一致性
    end
    
    Comp->>PU: 并行压缩文件
    PU-->>Comp: 压缩结果
    
    Comp-->>CE: 处理结果
    CE-->>User: 退出状态
```

### 拆包流程数据流

```mermaid
sequenceDiagram
    participant User as 用户
    participant UE as UncompressEntrance
    participant UV as UncompressVerify
    participant Unc as Uncompress
    participant FU as FileUtils
    
    User->>UE: 调用 unpack()
    UE->>UV: commandVerify(utility)
    UV-->>UE: 验证结果
    
    UE->>Unc: unpackageProcess(utility)
    
    alt MODE_HAP
        Unc->>Unc: unpackageHapMode()
        Unc->>FU: dataTransferAllFiles()
    else MODE_APP
        Unc->>Unc: dataTransferFilesByApp()
    end
    
    FU->>FU: unzip()
    FU-->>Unc: 解压结果
    
    Unc-->>UE: 处理结果
    UE-->>User: 返回 boolean
```

### 解析流程数据流

```mermaid
sequenceDiagram
    participant User as 应用市场/分析工具
    participant UE as UncompressEntrance
    participant Unc as Uncompress
    participant JU as JsonUtil
    participant Result as UncompressResult
    
    User->>UE: parseApp(appPath, mode, hapName)
    
    alt PARSE_MODE_HAPLIST
        UE->>Unc: uncompressAppByPath()
        Unc->>Unc: 提取 pack.info
        Unc->>JU: parseHapList()
    else PARSE_MODE_HAPINFO
        UE->>Unc: uncompressHapAndHspFromAppPath()
        Unc->>Unc: 提取指定 HAP
        Unc->>JU: parseProfileInfo()
    else PARSE_MODE_ALL
        UE->>Unc: uncompressAllAppByPath()
        Unc->>JU: 解析所有信息
    end
    
    JU-->>Unc: 解析结果
    Unc->>Result: 设置结果对象
    Result-->>UE: 返回结果
    UE-->>User: UncompressResult
```

## 线程模型

### 并行压缩架构

```mermaid
graph TB
    subgraph MainThread[主线程]
        M1[Compressor.compressProcess] --> M2[创建 ParallelScatterZipCreator]
        M2 --> M3[提交压缩任务]
    end
    
    subgraph ThreadPool[线程池]
        T1[工作线程 1] -->|压缩| Z1[ZipArchiveEntry]
        T2[工作线程 2] -->|压缩| Z2[ZipArchiveEntry]
        T3[工作线程 N] -->|压缩| Z3[ZipArchiveEntry]
    end
    
    subgraph Output[输出]
        Z1 --> O[ZipArchiveOutputStream]
        Z2 --> O
        Z3 --> O
    end
    
    M3 -.-> T1
    M3 -.-> T2
    M3 -.-> T3
```

### 线程安全设计

| 组件 | 线程安全策略 |
|-----|------------|
| **Compressor** | 每次调用创建新实例，无共享状态 |
| **Utility** | 配置对象，单线程使用 |
| **FileUtils** | 静态工具方法，无状态 |
| **PackageUtil** | 并行压缩使用线程池，线程安全 |

## 关键时序

### HAP 打包时序

```mermaid
sequenceDiagram
    participant Main as Main Thread
    participant Parser as CommandParser
    participant Verifier as CompressVerify
    participant HapVerifier as HapVerify
    participant Compressor as Compressor
    participant ZipCreator as ParallelScatterZipCreator
    
    Main->>Parser: 1. 解析命令行参数
    Parser->>Main: Utility 配置对象
    
    Main->>Verifier: 2. 验证参数
    Verifier->>Verifier: 检查路径、文件存在性
    Verifier->>Main: 验证结果
    
    Main->>Compressor: 3. 开始压缩
    Compressor->>Compressor: 3.1 解析 module.json
    Compressor->>HapVerifier: 3.2 验证 HAP 配置
    HapVerifier->>Compressor: 验证通过
    
    Compressor->>Compressor: 3.3 收集文件列表
    Compressor->>ZipCreator: 3.4 创建并行压缩器
    
    loop 每个文件
        Compressor->>ZipCreator: 3.5 提交压缩任务
    end
    
    ZipCreator->>Compressor: 3.6 等待完成
    Compressor->>Compressor: 3.7 写入中央目录
    Compressor->>Main: 4. 返回结果
```

### APP 打包时序（多 HAP 验证）

```mermaid
sequenceDiagram
    participant Comp as Compressor
    participant HV as HapVerify
    participant HVI as HapVerifyInfo
    participant File as HAP File
    
    Comp->>Comp: 1. 收集所有 HAP 路径
    
    loop 每个 HAP
        Comp->>File: 2.1 读取 HAP
        Comp->>HVI: 2.2 提取验证信息
        HVI->>Comp: HapVerifyInfo 对象
    end
    
    Comp->>HV: 3. 验证 HAP 列表
    HV->>HV: 3.1 检查 bundleName 一致性
    HV->>HV: 3.2 检查 versionCode 一致性
    HV->>HV: 3.3 检查 moduleName 唯一性
    HV->>HV: 3.4 检查 API 版本兼容性
    HV->>Comp: 验证结果
    
    alt 验证通过
        Comp->>Comp: 4. 打包所有 HAP
    else 验证失败
        Comp->>Comp: 4. 报错退出
    end
```

## 模块依赖关系

```mermaid
graph TB
    subgraph EntryModule[入口模块]
        E1[CompressEntrance]
        E2[UncompressEntrance]
        E3[ScanEntrance]
    end
    
    subgraph CoreModule[核心模块]
        C1[Compressor]
        C2[Uncompress]
        C3[Scan]
    end
    
    subgraph VerifyModule[验证模块]
        V1[CompressVerify]
        V2[UncompressVerify]
        V3[HapVerify]
        V4[ValidatorFactory]
    end
    
    subgraph UtilsModule[工具模块]
        U1[FileUtils]
        U2[PackageUtil]
        U3[JsonUtil]
        U4[ModuleJsonUtil]
    end
    
    subgraph DataModule[数据模块]
        D1[Utility]
        D2[Info Classes]
        D3[UncompressResult]
    end
    
    E1 --> C1
    E2 --> C2
    E3 --> C3
    
    C1 --> V1
    C2 --> V2
    C3 --> V3
    
    V1 --> V3
    V1 --> V4
    
    C1 --> U1
    C1 --> U2
    C1 --> U3
    C1 --> U4
    
    C2 --> U1
    C2 --> U3
    
    E1 --> D1
    C1 --> D2
    C2 --> D3
```

## 设计模式应用

| 模式 | 应用位置 | 说明 |
|-----|---------|------|
| **模板方法** | `AbstractPackValidator` | `validate()` 定义流程，子类实现 `isVerifyValid()` |
| **工厂模式** | `PackValidatorFactory` | 根据类型创建对应验证器 |
| **策略模式** | `ResourcesParser` | `ResourcesParserV1/V2` 实现不同解析策略 |
| **单例模式** | `Log` | 日志工具类 |
| **建造者模式** | `Utility` | 通过 setter 构建配置对象 |
