# 编译产物

## 目的

本文档列出 PrintSpooler 项目的编译产物，包括产物清单、安装路径和运行时加载关系。

## 适用范围

本文档适用于：

- DevOps 工程师
- 部署人员
- 需要了解产物结构的技术人员

## 关键结论

1. **主产物**：entry-default.hap（应用包）
2. **共享库**：common.har, ippPrint.har
3. **驱动扩展**：driverEntry-default.hap
4. **安装路径**：`/system/app/com.ohos.spooler/`
5. **运行时加载**：HAP 包通过系统 PackageManager 加载

---

## 产物清单

### 主要产物

| 产物文件 | 类型 | 大小估算 | 用途 |
|---------|------|----------|------|
| entry-default.hap | HAP | ~10-20 MB | 主应用包 |
| common.har | HAR | ~1-2 MB | 公共库 |
| ippPrint.har | HAR | ~2-3 MB | IPP 功能库 |
| driverEntry-default.hap | HAP | ~1-5 MB | 驱动扩展包 |

**证据**：根据模块配置和构建系统推断

---

## 安装路径

### Entry 模块

**安装路径**：`/system/app/com.ohos.spooler/entry-default.hap`

**权限要求**：系统应用，需要系统签名

**安装命令**：
```bash
# 上传到设备
hdc file send entry-default.hap /system/app/com.ohos.spooler/entry-default.hap

# 设置权限
hdc shell chmod 644 /system/app/com.ohos.spooler/entry-default.hap
```

**证据**：
- README.md:344-348
- AppScope/app.json5:3

### DriverEntry 模块

**安装路径**：`/system/app/com.ohos.spooler/driverEntry-default.hap`

**说明**：驱动扩展与主应用在同一个包名下

**证据**：README.md 安装说明

---

## 产物结构

### Entry HAP 结构

```
entry-default.hap
├── config.json              # 配置信息
├── resources/               # 资源文件
│   ├── base/
│   ├── en_US/
│   ├── zh_CN/
│   └── ...
├── ets/                    # ArkTS 代码（编译后）
│   ├── MainAbility/
│   ├── ServiceExtAbility/
│   ├── Controller/
│   ├── Model/
│   ├── Common/
│   ├── workers/
│   └── pages/
└── libs/                    # 依赖的 HAR 文件
    ├── common.har
    └── ippPrint.har
```

**证据**：根据模块配置和 HAP 结构推断

---

## 运行时加载关系

### 启动流程

```
系统启动
    ↓
[PackageManager]
    ↓
加载 entry-default.hap
    ↓
解压 libs/
    ↓
加载 common.har
    ↓
加载 ippPrint.har
    ↓
[Entry Ability 启动]
    ↓
MainAbility.onCreate()
    ↓
初始化服务和发现
```

### 模块加载顺序

1. **系统加载 HAP**
   - 解压 entry-default.hap
   - 读取 config.json
   - 加载资源文件

2. **加载依赖 HAR**
   - 加载 common.har 到运行时
   - 加载 ippPrint.har 到运行时

3. **启动主 Ability**
   - MainAbility.onCreate()
   - 初始化 AppStorage
   - 创建服务实例

4. **初始化扩展**
   - PrintExtension.onCreate()
   - 初始化发现服务

**证据**：
- Ability 生命周期：`entry/src/main/ets/MainAbility/MainAbility.ets:42`
- 服务初始化：`entry/src/main/ets/ServiceExtAbility/PrintExtension.ts:40-62`

---

## 运行时依赖

### 运行时依赖

| 模块 | 依赖 | 运行时提供 |
|------|------|----------|
| entry | common.har | 工具类、常量 |
| entry | ippPrint.har | 发现、连接、IPP 协议 |
| ippPrint | common.har | 日志、工具类 |
| driverEntry | 无 | 独立运行 |

**证据**：oh-package.json5 依赖声明

---

## 资源文件

### 资源类型

| 资源类型 | 目录 | 用途 |
|---------|------|------|
| 字符串 | resources/base/element/string.json | 应用文本 |
| 颜色 | resources/base/element/color.json | 颜色定义 |
| 浮点数 | resources/base/element/float.json | 浮点数值 |
| 媒体 | resources/base/media/ | 图片、图标等 |

**证据**：
- entry/src/main/resources/
- 代码中的引用：`entry/src/main/ets/pages/PrintPage.ets:48`

### 国际化

| 语言 | 目录 |
|-----|------|
| 中文（简体） | resources/zh_CN/element/ |
| 英文 | resources/en_US/element/ |
| 其他 | resources/zz_ZX/element/ |

**证据**：
- entry/src/main/resources/zh_CN/
- entry/src/main/resources/en_US/

---

## DriverEntry 特殊产物

### 驱动库文件

```
driverEntry/libs/arm64-v8a/
├── rastertopwg              # CUPS filter
├── HUAWEI_PixLab_xxx.ppd  # PPD 文件
├── libsane-pantumxxx.so    # SANE backend
└── lpd                      # CUPS backend
```

**配置路径**（证据：`driverEntry/src/main/module.json5:22-50`）：

| 库文件 | 配置路径 | 说明 |
|---------|----------|------|
| rastertopwg | /print_service/cups/serverbin/filter | CUPS 过滤器 |
| HUAWEI_PixLab_xxx.ppd | /print_service/cups/datadir/model | CUPS PPD 文件 |
| libsane-pantumxxx.so | /print_service/sane/backend | SANE 后端 |
| lpd | /print_service/cups/serverbin/backend | CUPS 后端 |

---

## 安装和部署

### 标准安装流程

1. **构建 HAP 包**
   ```bash
   hvigorw --mode module -p module=entry@default
   ```

2. **签名 HAP 包**
   - 使用 signcenter_tool 进行签名
   - 生成签名后的 HAP 文件

3. **上传到设备**
   ```bash
   hdc file send signed-entry.hap /system/app/com.ohos.spooler/
   ```

4. **设置权限**
   ```bash
   hdc shell chmod 644 /system/app/com.ohos.spooler/*.hap
   ```

5. **重启系统**
   ```bash
   hdc shell
   reboot
   ```

**证据**：README.md:345-356

---

## 相关跳转

- [目录结构](02_Directory_Structure.md) - 模块组织
- [构建配置](06_GN_Build.md) - 构建系统配置
- [对外 API](04_External_API.md) - OpenHarmony 系统 API 使用
