# 附录：配置标志说明

## 命令行参数

### 打包工具参数（app_packing_tool.jar）

#### 模式参数

| 参数 | 说明 | 可选值 | 必填 |
|-----|------|-------|------|
| `--mode` | 打包模式 | hap, hsp, app, fastApp, multiApp, hqf, appqf, res, versionNormalize, packageNormalize | 是 |

#### 输入路径参数

| 参数 | 说明 | 示例 | 条件必填 |
|-----|------|------|---------|
| `--json-path` | module.json/config.json 路径 | `--json-path entry/src/main/module.json` | mode=hap/hsp/hqf 时必填 |
| `--resources-path` | 资源目录路径 | `--resources-path entry/build/res` | 否 |
| `--ets-path` | ArkTS 代码路径 | `--ets-path entry/build/ets` | 否 |
| `--hap-path` | HAP 文件路径 | `--hap-path entry.hap` | mode=app 时必填 |
| `--hsp-path` | HSP 文件路径 | `--hsp-path library.hsp` | 否 |
| `--pack-info-path` | pack.info 路径 | `--pack-info-path pack.info` | mode=app 时必填 |
| `--lib-path` | 库文件路径 | `--lib-path libs/arm64-v8a` | 否 |
| `--dex-path` | DEX 文件路径 | `--dex-path classes.dex` | FA 模型 |
| `--assets-path` | 资产目录路径 | `--assets-path assets` | 否 |
| `--js-path` | JS 代码路径 | `--js-path js` | 否 |
| `--an-path` | AN 文件路径 | `--an-path an` | 否 |
| `--ap-path` | AP 文件路径 | `--ap-path ap` | 否 |
| `--hnp-path` | HNP 文件路径 | `--hnp-path hnp` | 否 |
| `--dir-list` | 额外目录列表 | `--dir-list dir1,dir2` | 否 |

#### 输出参数

| 参数 | 说明 | 示例 | 必填 |
|-----|------|------|------|
| `--out-path` | 输出文件路径 | `--out-path output.hap` | 是 |
| `--force` | 强制覆盖 | `--force true` | 否 |

#### 打包选项参数

| 参数 | 说明 | 可选值 | 默认值 |
|-----|------|-------|-------|
| `--compress-level` | 压缩级别 | 1-9 | 1 |
| `--compress-native-libs` | 压缩原生库 | true, false | false |
| `--generate-build-hash` | 生成构建哈希 | true, false | false |

#### 签名参数

| 参数 | 说明 | 示例 | 条件必填 |
|-----|------|------|---------|
| `--signature-path` | 签名文件路径 | `--signature-path signature.p7b` | 否 |
| `--certificate-path` | 证书文件路径 | `--certificate-path cert.cer` | 否 |

#### 特殊参数

| 参数 | 说明 | 示例 | 适用模式 |
|-----|------|------|---------|
| `--encrypt-path` | 加密配置文件 | `--encrypt-path encrypt.json` | mode=app |
| `--pac-json-path` | PAC JSON 路径 | `--pac-json-path pac.json` | mode=app |
| `--replace-pack-info` | 替换 pack.info | `--replace-pack-info true` | mode=app |
| `--pkg-context-path` | 包上下文路径 | `--pkg-context-path pkgContextInfo.json` | mode=hap/hsp |
| `--pkg-sdk-info-path` | SDK 信息路径 | `--pkg-sdk-info-path pkgSdkInfo.json` | mode=hap/hsp |
| `--hqf-list` | HQF 列表 | `--hqf-list fix1.hqf,fix2.hqf` | mode=appqf |
| `--input-list` | 输入列表 | `--input-list 1.hap,2.hsp` | versionNormalize |
| `--version-code` | 版本号 | `--version-code 1000001` | versionNormalize |
| `--version-name` | 版本名 | `--version-name 1.0.1` | versionNormalize |
| `--bundle-name` | 包名 | `--bundle-name com.example` | packageNormalize |
| `--hsp-list` | HSP 列表 | `--hsp-list 1.hsp,2.hsp` | packageNormalize |

#### 原子服务参数

| 参数 | 说明 | 可选值 | 默认值 |
|-----|------|-------|-------|
| `--atomic-service-entry-size-limit` | Entry 大小限制 | 0-4194304 (KB) | 2048 |
| `--atomic-service-non-entry-size-limit` | 非 Entry 大小限制 | 0-4194304 (KB) | 2048 |
| `--atomic-service-total-size-limit` | 总大小限制 | 0-4194304 (KB) | 4194304 |

### 拆包工具参数（app_unpacking_tool.jar）

#### 模式参数

| 参数 | 说明 | 可选值 | 必填 |
|-----|------|-------|------|
| `--mode` | 拆包模式 | hap, hsp, app, appqf, har | 是 |

#### 输入参数

| 参数 | 说明 | 示例 | 必填 |
|-----|------|------|------|
| `--hap-path` | HAP 文件路径 | `--hap-path input.hap` | mode=hap |
| `--hsp-path` | HSP 文件路径 | `--hsp-path input.hsp` | mode=hsp |
| `--app-path` | APP 文件路径 | `--app-path input.app` | mode=app |
| `--appqf-path` | APPQF 文件路径 | `--appqf-path input.appqf` | mode=appqf |
| `--har-path` | HAR 文件路径 | `--har-path input.har` | mode=har |

#### 输出参数

| 参数 | 说明 | 示例 | 必填 |
|-----|------|------|------|
| `--out-path` | 输出目录 | `--out-path output/` | 是 |
| `--force` | 强制覆盖 | `--force true` | 否 |

#### 特殊选项

| 参数 | 说明 | 可选值 | 默认值 |
|-----|------|-------|-------|
| `--rpcid` | 只提取 rpcid | true, false | false |
| `--libs` | 按架构拆分 libs | true, false | false |
| `--cpu-abis` | 指定 CPU 架构 | `--cpu-abis arm64-v8a,armeabi-v7a` | 否 |
| `--unpack-apk` | 解压 APK | true, false | false |
| `--device-type` | 设备类型过滤 | phone, tablet, tv, car, wearable | 否 |

### 扫描工具参数（app_check_tool.jar）

| 参数 | 说明 | 示例 | 必填 |
|-----|------|------|------|
| `--mode` | 扫描模式 | scan | 是 |
| `--hap-path` | HAP 路径 | `--hap-path input.hap` | 是 |
| `--stat-file-size` | 统计文件大小 | `--stat-file-size 1024` | 否 |
| `--stat-suffix` | 按后缀统计 | `--stat-suffix .so,.png` | 否 |
| `--stat-duplicate` | 统计重复文件 | `--stat-duplicate true` | 否 |
| `--out-path` | 输出路径 | `--out-path result.json` | 否 |

## Utility 配置常量

### 模式常量

```java
// adapter/ohos/Utility.java
public static final String MODE_HAP = "hap";
public static final String MODE_HAR = "har";
public static final String MODE_APP = "app";
public static final String MODE_FAST_APP = "fastApp";
public static final String MODE_MULTI_APP = "multiApp";
public static final String MODE_HQF = "hqf";
public static final String MODE_APPQF = "appqf";
public static final String MODE_RES = "res";
public static final String MODE_HSP = "hsp";
public static final String MODE_HAPADDITION = "hapAddition";
public static final String VERSION_NORMALIZE = "versionNormalize";
public static final String PACKAGE_NORMALIZE = "packageNormalize";
public static final String GENERAL_NORMALIZE = "generalNormalize";
```

### 文件后缀常量

```java
// adapter/ohos/Constants.java
public static final String HAP_SUFFIX = ".hap";
public static final String HSP_SUFFIX = ".hsp";
public static final String HAR_SUFFIX = ".har";
public static final String APP_SUFFIX = ".app";
public static final String APPQF_SUFFIX = ".appqf";
public static final String HQF_SUFFIX = ".hqf";
public static final String RES_SUFFIX = ".res";
public static final String JSON_SUFFIX = ".json";
public static final String SO_SUFFIX = ".so";
public static final String APK_SUFFIX = ".apk";
public static final String DEX_SUFFIX = ".dex";
public static final String PNG_SUFFIX = ".png";
```

### JSON 键名常量

```java
// adapter/ohos/Constants.java
// Module JSON 键
public static final String MODULE = "module";
public static final String MODULE_NAME = "moduleName";
public static final String MODULE_TYPE = "moduleType";
public static final String DEVICE_TYPE = "deviceType";
public static final String DEVICE_TYPES = "deviceTypes";
public static final String ABILITIES = "abilities";
public static final String EXTENSION_ABILITIES = "extensionAbilities";

// App JSON 键
public static final String APP = "app";
public static final String BUNDLE_NAME = "bundleName";
public static final String VERSION_CODE = "versionCode";
public static final String VERSION_NAME = "versionName";
public static final String MIN_API_VERSION = "minAPIVersion";
public static final String TARGET_API_VERSION = "targetAPIVersion";
public static final String BUNDLE_TYPE = "bundleType";

// 其他键
public static final String NAME = "name";
public static final String TYPE = "type";
public static final String DISTRO = "distro";
public static final String FORMS = "forms";
public static final String META_DATA = "metaData";
public static final String DEPENDENCIES = "dependencies";
```

### 目录名常量

```java
// adapter/ohos/Constants.java
public static final String RES_DIR_NAME = "res/";
public static final String RESOURCES_DIR_NAME = "resources/";
public static final String LIBS_DIR_NAME = "libs/";
public static final String AN_DIR_NAME = "an/";
public static final String AP_PATH_NAME = "ap/";
public static final String ASSETS_DIR_NAME = "assets/";
public static final String SO_DIR_NAME = "maple/";
public static final String SHARED_LIBS_DIR_NAME = "shared_libs/";
public static final String JS_PATH = "js/";
public static final String ETS_PATH = "ets/";
public static final String HNP_PATH = "hnp/";
```

### 限制值常量

```java
// adapter/ohos/Compressor.java
// 原子服务大小限制（单位：KB）
private static final int ATOMIC_SERVICE_ENTRY_SIZE_LIMIT_DEFAULT = 2048;        // 2MB
private static final int ATOMIC_SERVICE_NON_ENTRY_SIZE_LIMIT_DEFAULT = 2048;    // 2MB
private static final int ATOMIC_SERVICE_TOTAL_SIZE_LIMIT_DEFAULT = 4194304;     // 4GB
private static final int ATOMIC_SERVICE_TOTAL_SIZE_LIMIT_MAX = 4194304;         // 4GB

// 缓冲区大小
private static final int BUFFER_BYTE_SIZE = 1024;
private static final int BUFFER_WRITE_SIZE = 1444;
private static final int BUFFER_SIZE = 40 * 1024;  // 40KB

// 其他限制
private static final int QUERY_SCHEMES_CHECK_COUNT = 50;
private static final int QUERY_SCHEMES_CHECK_MIN_API_VERSION = 21;
private static final int DEDUPLICATE_HAR_CHECK_MIN_API_VERSION = 21;
private static final int MIN_API_VERSION_ERROR_VALUE = -1;
```

## GN 构建配置

### 编译选项

```gn
# 安全编译选项
branch_protector_ret = "pac_ret"          # 分支保护
sanitize = {
  boundary_sanitize = true                 # 边界检查
  cfi = true                               # 控制流完整性
  cfi_cross_dso = true
  debug = false
  integer_overflow = true                  # 整数溢出检查
  ubsan = true                             # 未定义行为检查
}
cflags = [ "-fstack-protector-strong" ]    # 栈保护
cflags_cc = [
  "-fexceptions",                          # 启用异常
  "-fstack-protector-strong",
]
```

### 依赖组件

```gn
# 外部依赖
external_deps = [
  "json:nlohmann_json_static",             # C++ JSON
  "openssl:libcrypto_shared",              # OpenSSL
  "zlib:libz",                             # zlib
  "bounds_checking_function:libsec_static", # 边界检查
  "cJSON:cjson_static",                    # C JSON
  "hilog:libhilog",                        # 日志
]
```

## 环境变量

### 构建环境变量

| 变量 | 说明 | 示例 |
|-----|------|------|
| `JAVA_OPTS` | JVM 选项 | `-Xmx4g` |
| `PACKING_TOOL_LOG_LEVEL` | 日志级别 | `DEBUG` |
| `OUT_DIR` | 输出目录 | `out/` |
| `TARGET_OUT_DIR` | 目标输出目录 | `out/target/` |

### 运行时环境变量

| 变量 | 说明 | 示例 |
|-----|------|------|
| `user.dir` | 当前工作目录 | `/workspace` |
| `java.io.tmpdir` | 临时目录 | `/tmp` |

## 配置文件

### bundle.json

```json
{
  "name": "@ohos/packing_tool",
  "version": "3.2",
  "license": "Apache License 2.0",
  "component": {
    "name": "packing_tool",
    "subsystem": "developtools",
    "adapted_system_type": ["mini", "small", "standard"]
  }
}
```

### module.json 关键字段

```json
{
  "module": {
    "name": "entry",                    // 模块名，必填
    "type": "entry",                    // 类型：entry/feature/shared，必填
    "deviceTypes": ["phone", "tablet"], // 支持设备，必填
    "deliveryWithInstall": true,        // 随安装分发
    "installationFree": false,          // 免安装
    "pages": "$profile:main_pages",     // 页面配置
    "abilities": [...],                 // Ability 列表
    "extensionAbilities": [...],        // 扩展 Ability
    "requestPermissions": [...],        // 权限申请
    "dependencies": [...]               // 依赖
  },
  "app": {
    "bundleName": "com.example.app",    // 包名，必填
    "versionCode": 1000000,             // 版本号，必填
    "versionName": "1.0.0",             // 版本名
    "minAPIVersion": 9,                 // 最小 API 版本
    "targetAPIVersion": 12,             // 目标 API 版本
    "bundleType": "app"                 // 包类型
  }
}
```

## 快速参考

### 常用命令速查

```bash
# 打包 HAP
java -jar app_packing_tool.jar --mode hap --json-path module.json --out-path out.hap

# 打包 APP
java -jar app_packing_tool.jar --mode app --hap-path entry.hap,feature.hap --pack-info-path pack.info --out-path out.app

# 拆包 HAP
java -jar app_unpacking_tool.jar --mode hap --hap-path in.hap --out-path out/

# 解析 APP
java -jar app_unpacking_tool.jar --mode app --app-path in.app --parse-mode all

# 扫描 HAP
java -jar app_check_tool.jar --mode scan --hap-path in.hap
```

### 参数组合速查

| 场景 | 命令 |
|-----|------|
| Stage HAP | `--mode hap --json-path module.json` |
| FA HAP | `--mode hap --json-path config.json` |
| HSP | `--mode hsp --json-path module.json` |
| APP | `--mode app --hap-path *.hap --pack-info-path pack.info` |
| 多工程 APP | `--mode multiApp --hap-list 1.hap,2.hap --app-list 1.app` |
| HQF | `--mode hqf --json-path patch.json` |
| 版本归一化 | `--mode versionNormalize --input-list 1.hap,2.hsp --version-code 1000001` |
