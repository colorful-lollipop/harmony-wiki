# 目录结构

## 顶层目录结构

```
developtools/packing_tool/
├── adapter/                    # 适配层代码（Java 实现）
│   ├── ohos/                  # OpenHarmony 适配实现
│   │   ├── validator/         # 验证器抽象层
│   │   └── restool/           # 资源解析工具
│   └── scanner/               # 扫描器适配
├── configcheck/               # 配置校验 JSON Schema
├── jar/                       # 预构建 JAR 依赖
├── modulecheck/               # 模块配置校验 Schema
├── ohos_packing_tool/         # C++ 轻量实现
│   └── frameworks/
│       ├── include/           # C++ 头文件
│       ├── src/               # C++ 源码
│       └── BUILD.gn           # GN 构建配置
├── packing_tool/              # C++ 完整实现
│   └── frameworks/
│       ├── include/           # 头文件
│       ├── src/               # 源码
│       └── test/              # 单元测试
├── META-INF/                  # 打包配置清单
├── img/                       # 文档图片
├── bundle.json                # 组件配置
├── BUILD.gn                   # 根 GN 构建文件
├── packingtool.gni            # GN 模板定义
├── build.py                   # Python 构建脚本
├── packingTool.sh             # 打包脚本
├── unpackingTool.sh           # 拆包脚本
├── checkTool.sh               # 检查脚本
├── haptobin.sh                # hap 转 bin 脚本
├── README_zh.md               # 中文使用说明
└── LICENSE                    # Apache 2.0 许可证
```

## Java 实现详细结构

### adapter/ohos/ 目录

```
adapter/ohos/
├── CompressEntrance.java           # 打包入口类
├── UncompressEntrance.java         # 拆包入口类
├── ScanEntrance.java               # 扫描入口类
├── CommandParser.java              # 命令行解析器
├── Utility.java                    # 配置参数类
│
├── Compressor.java                 # 压缩引擎核心
├── Uncompress.java                 # 解压引擎核心
├── Scan.java                       # 扫描引擎核心
│
├── CompressVerify.java             # 打包参数验证
├── UncompressVerify.java           # 拆包参数验证
├── HapVerify.java                  # HAP 包验证
├── HQFVerify.java                  # HQF 包验证
├── ScanVerify.java                 # 扫描参数验证
├── VerifyCollection.java           # 验证工具集合
│
├── FileUtils.java                  # 文件操作工具
├── PackageUtil.java                # 包操作工具
├── JsonUtil.java                   # JSON 解析工具
├── ModuleJsonUtil.java             # 模块 JSON 工具
├── Log.java                        # 日志工具
├── ShowHelp.java                   # 帮助信息
│
├── AppInfo.java                    # 应用信息数据类
├── HapInfo.java                    # HAP 信息数据类
├── HapVerifyInfo.java              # HAP 验证信息
├── HQFInfo.java                    # HQF 信息
├── PackInfo.java                   # 包信息
├── ProfileInfo.java                # 配置信息
├── UncompressResult.java           # 解压结果
├── APPQFResult.java                # APPQF 结果
│
├── ModuleJsonInfo.java             # 模块 JSON 信息
├── ModuleInfo.java                 # 模块信息
├── ModuleAppInfo.java              # 模块应用信息
├── ModuleProfileInfo.java          # 模块配置信息
├── ModuleApiVersion.java           # 模块 API 版本
├── ModuleDeviceType.java           # 模块设备类型
├── ModuleShortcut.java             # 模块快捷方式
├── ModuleAtomicService.java        # 模块原子服务
├── ModuleMetadataInfo.java         # 模块元数据
├── ModuleAdaption.java             # 模块适配
├── ModuleResult.java               # 模块结果
│
├── AbilityInfo.java                # Ability 信息
├── ExtensionAbilityInfo.java       # 扩展 Ability 信息
├── AbilityFormInfo.java            # Ability 卡片信息
├── ModuleAbilityInfo.java          # 模块 Ability 信息
│
├── FormInfo.java                   # 卡片信息
├── MetaData.java                   # 元数据
├── MetaDataInfo.java               # 元数据信息
├── IntentInfo.java                 # Intent 信息
├── SkillInfo.java                  # Skill 信息
├── Want.java                       # Want 信息
├── UriInfo.java                    # URI 信息
├── JsInfo.java                     # JS 信息
│
├── Distro.java                     # 分发配置
├── DistroFilter.java               # 分发过滤器
├── DeviceConfig.java               # 设备配置
├── ApiVersion.java                 # API 版本
├── Version.java                    # 版本信息
├── CountryCode.java                # 国家码
├── ScreenDensity.java              # 屏幕密度
├── ScreenShape.java                # 屏幕形状
├── ScreenWindow.java               # 屏幕窗口
│
├── DefPermission.java              # 定义权限
├── DefinePermission.java           # 权限定义
├── DefPermissionGroup.java         # 权限组
├── ReqPermission.java              # 请求权限
├── UsedScene.java                  # 使用场景
│
├── DependencyItem.java             # 依赖项
├── PreloadItem.java                # 预加载项
├── ProxyDataItem.java              # 代理数据项
├── MultiAppMode.java               # 多应用模式
├── CustomizeData.java              # 自定义数据
│
├── Shortcut.java                   # 快捷方式
├── CommonEvent.java                # 公共事件
├── ResourceIndexResult.java        # 资源索引结果
├── HapZipInfo.java                 # HAP Zip 信息
│
├── Constants.java                  # 常量定义
├── ErrorMsg.java                   # 错误消息
├── PackingToolErrMsg.java          # 打包工具错误
├── PackFormatter.java              # 包格式化
├── ScanErrorEnum.java              # 扫描错误枚举
│
├── PackageNormalize.java           # 包归一化
├── PackageUtil.java                # 包工具
├── IncrementalPack.java            # 增量打包
├── ConvertHapToBin.java            # HAP 转二进制
├── BinaryTool.java                 # 二进制工具
├── CollectBinInfo.java             # 收集二进制信息
│
├── BundleException.java            # 异常类
│
├── validator/                      # 验证器包
│   ├── AbstractPackValidator.java  # 抽象验证器
│   ├── HapValidator.java           # HAP 验证器
│   ├── HspValidator.java           # HSP 验证器
│   └── PackValidatorFactory.java   # 验证器工厂
│
└── restool/                        # 资源工具包
    ├── ResourcesParser.java        # 资源解析器接口
    ├── ResourcesParserV1.java      # V1 解析器
    ├── ResourcesParserV2.java      # V2 解析器
    └── ResourcesParserFactory.java # 解析器工厂
```

## C++ 实现详细结构

### ohos_packing_tool/frameworks/ 目录（轻量版）

```
ohos_packing_tool/frameworks/
├── include/
│   ├── constants.h               # 常量定义
│   ├── packager.h                # 打包器基类
│   ├── shell_command.h           # 命令行处理
│   ├── hap_packager.h            # HAP 打包器
│   └── hsp_packager.h            # HSP 打包器
├── src/
│   ├── main.cpp                  # 程序入口
│   ├── packager.cpp              # 打包器实现
│   ├── shell_command.cpp         # 命令行实现
│   ├── hap_packager.cpp          # HAP 实现
│   └── hsp_packager.cpp          # HSP 实现
└── BUILD.gn                      # GN 构建配置
```

### packing_tool/frameworks/ 目录（完整版）

```
packing_tool/frameworks/
├── include/
│   ├── app_log_wrapper.h         # 日志包装器
│   ├── app_packager.h            # APP 打包器
│   ├── appqf_packager.h          # APPQF 打包器
│   ├── constants.h               # 常量
│   ├── fast_app_packager.h       # 快速 APP 打包器
│   ├── general_normalize.h       # 通用归一化
│   ├── hap_packager.h            # HAP 打包器
│   ├── hqf_packager.h            # HQF 打包器
│   ├── hqf_verify.h              # HQF 验证
│   ├── hsp_packager.h            # HSP 打包器
│   ├── incremental_pack.h        # 增量打包
│   ├── log.h                     # 日志
│   ├── multiapp_packager.h       # 多应用打包器
│   ├── package_normalize.h       # 包归一化
│   ├── packager.h                # 打包器基类
│   ├── res_packager.h            # 资源打包器
│   ├── scan_statdulpicate.h      # 重复扫描
│   ├── shell_command.h           # 命令行
│   ├── unzip_wrapper.h           # 解压包装器
│   ├── utils.h                   # 工具
│   ├── version_normalize.h       # 版本归一化
│   ├── zip_constants.h           # Zip 常量
│   ├── zip_utils.h               # Zip 工具
│   ├── zip_wrapper.h             # Zip 包装器
│   └── json/                     # JSON 相关头文件
│       ├── dependency_item.h
│       ├── distro_filter.h
│       ├── general_normalize_version.h
│       ├── general_normalize_version_utils.h
│       ├── hap_verify_info.h
│       ├── hap_verify_utils.h
│       ├── json_utils.h
│       ├── module_api_version.h
│       ├── module_json.h
│       ├── module_json_utils.h
│       ├── module_metadata_info.h
│       ├── multi_app_mode.h
│       ├── normalize_version.h
│       ├── normalize_version_utils.h
│       ├── pack_info.h
│       ├── pack_info_utils.h
│       ├── patch_json.h
│       ├── patch_json_utils.h
│       ├── preload_item.h
│       └── pt_json.h
├── src/
│   ├── main.cpp                  # 入口
│   ├── app_packager.cpp          # APP 打包
│   ├── appqf_packager.cpp        # APPQF 打包
│   ├── fast_app_packager.cpp     # 快速 APP 打包
│   ├── general_normalize.cpp     # 通用归一化
│   ├── hap_packager.cpp          # HAP 打包
│   ├── hqf_packager.cpp          # HQF 打包
│   ├── hqf_verify.cpp            # HQF 验证
│   ├── hsp_packager.cpp          # HSP 打包
│   ├── incremental_pack.cpp      # 增量打包
│   ├── log.cpp                   # 日志
│   ├── multiapp_packager.cpp     # 多应用打包
│   ├── package_normalize.cpp     # 包归一化
│   ├── packager.cpp              # 打包器基类
│   ├── res_packager.cpp          # 资源打包
│   ├── scan_statdulpicate.cpp    # 重复扫描
│   ├── shell_command.cpp         # 命令行
│   ├── unzip_wrapper.cpp         # 解压
│   ├── utils.cpp                 # 工具
│   ├── version_normalize.cpp     # 版本归一化
│   ├── zip_utils.cpp             # Zip 工具
│   ├── zip_wrapper.cpp           # Zip 包装
│   └── json/                     # JSON 实现
│       ├── distro_filter.cpp
│       ├── general_normalize_version_utils.cpp
│       ├── hap_verify_info.cpp
│       ├── hap_verify_utils.cpp
│       ├── json_utils.cpp
│       ├── module_json.cpp
│       ├── module_json_fa.cpp
│       ├── module_json_stage.cpp
│       ├── module_json_utils.cpp
│       ├── normalize_version_utils.cpp
│       ├── pack_info.cpp
│       ├── pack_info_utils.cpp
│       ├── patch_json.cpp
│       ├── patch_json_utils.cpp
│       └── pt_json.cpp
└── test/                         # 单元测试（按需求排除）
```

## 配置校验目录

### configcheck/ 目录

```
configcheck/
├── BUILD.gn                      # GN 构建配置
├── configSchema_lite.json        # 轻量系统配置 Schema
└── configSchema_rich.json        # 富设备配置 Schema
```

### modulecheck/ 目录

```
modulecheck/
├── BUILD.gn
├── app.json                      # app.json Schema
├── module.json                   # module.json Schema
├── pages.json                    # pages.json Schema
├── forms.json                    # forms.json Schema
├── shortcuts.json                # shortcuts.json Schema
├── commonEvents.json             # commonEvents.json Schema
├── distroFilter.json             # distroFilter.json Schema
├── appStartup.json               # appStartup.json Schema
├── appStartupInner.json          # 内部启动配置 Schema
├── insightIntent.json            # 意图 Schema
├── insightIntentInner.json       # 内部意图 Schema
├── configuration.json            # 配置 Schema
├── routerMap.json                # 路由映射 Schema
├── menu.json                     # 菜单 Schema
├── arkDataSchema.json            # ArkData Schema
├── customUtds.json               # 自定义 UTD Schema
├── crossAppSharedConfig.json     # 跨应用共享配置
└── themeConfig.json              # 主题配置 Schema
```

## 构建脚本目录

### 根目录脚本

| 文件 | 用途 | 调用关系 |
|-----|------|---------|
| `build.py` | Python 构建主脚本 | 被 GN 调用 |
| `packingTool.sh` | 打包工具编译脚本 | 被 build.py 调用 |
| `unpackingTool.sh` | 拆包工具编译脚本 | 被 build.py 调用 |
| `checkTool.sh` | 检查工具编译脚本 | 被 build.py 调用 |
| `haptobin.sh` | HAP 转 BIN 编译脚本 | 被 build.py 调用 |

## 模块职责说明

### 入口模块（Entrance）

| 类 | 职责 | 对应 JAR |
|---|------|---------|
| `CompressEntrance` | 打包功能入口，处理命令行参数 | app_packing_tool.jar |
| `UncompressEntrance` | 拆包/解析功能入口 | app_unpacking_tool.jar |
| `ScanEntrance` | 扫描功能入口 | app_check_tool.jar |

### 核心引擎模块（Core）

| 类 | 职责 | 关键方法 |
|---|------|---------|
| `Compressor` | 执行打包逻辑，支持多种包类型 | `compressProcess()`, `compressHap()`, `compressHsp()` |
| `Uncompress` | 执行拆包逻辑 | `unpackageProcess()`, `uncompressHap()` |
| `Scan` | 执行扫描逻辑 | `scanProcess()` |

### 验证模块（Verify）

| 类 | 职责 | 验证内容 |
|---|------|---------|
| `CompressVerify` | 打包参数验证 | 路径、文件存在性、参数范围 |
| `HapVerify` | HAP 包验证 | bundleName、version、module 一致性 |
| `HapValidator` | HAP SDK 验证 | pkgSdkInfo.json 校验 |
| `HspValidator` | HSP SDK 验证 | pkgSdkInfo.json 校验 |

### 工具模块（Utils）

| 类 | 职责 | 关键功能 |
|---|------|---------|
| `FileUtils` | 文件操作 | 读写、删除、路径验证、SHA256 |
| `PackageUtil` | 包操作 | 并行压缩、Zip 操作 |
| `JsonUtil` | JSON 解析 | FastJSON 封装 |
| `ModuleJsonUtil` | 模块 JSON 解析 | module.json 专用解析 |

### 数据模型模块（Model）

| 类 | 职责 | 对应 JSON |
|---|------|----------|
| `AppInfo` | 应用级信息 | app.json / module.json#app |
| `HapInfo` | HAP 模块信息 | module.json#module |
| `HapVerifyInfo` | HAP 验证数据 | 运行时生成 |
| `PackInfo` | 包清单信息 | pack.info |
| `ProfileInfo` | 完整配置信息 | 综合信息 |

## 文件命名规范

### Java 文件

| 后缀 | 含义 | 示例 |
|-----|------|------|
| `*Entrance.java` | 入口类 | `CompressEntrance.java` |
| `*Verify.java` | 验证类 | `HapVerify.java` |
| `*Info.java` | 数据类 | `HapInfo.java` |
| `*Util.java` | 工具类 | `FileUtils.java` |
| `*Result.java` | 结果类 | `UncompressResult.java` |

### C++ 文件

| 后缀 | 含义 | 示例 |
|-----|------|------|
| `*_packager.h/cpp` | 打包器实现 | `hap_packager.cpp` |
| `*_utils.h/cpp` | 工具实现 | `json_utils.cpp` |
| `*_wrapper.h/cpp` | 包装器 | `zip_wrapper.cpp` |

## 代码统计（非测试代码）

| 类型 | 文件数 | 主要功能 |
|-----|-------|---------|
| Java 源文件 | ~97 | 完整打包拆包功能 |
| C++ 头文件 | ~35 | 类型定义和接口 |
| C++ 源文件 | ~30 | 原生实现 |
| JSON Schema | ~18 | 配置校验 |
| 构建脚本 | ~6 | GN、Python、Shell |
