# 构建系统

## 概述

本仓库使用 GN (Generate Ninja) 作为主要构建系统，配合独立的 `build.sh` 脚本实现 HAP 的独立构建。

## GN 构建配置

### BUILD.gn 入口

**位置**: `BUILD.gn:1-348`

**导入语句**:
```gn
import("//build/ohos.gni")
```

### 目标类型

#### ohos_prebuilt_etc

用于定义预构建 HAP 文件的 GN 模板。

**模板签名**:
```gn
ohos_prebuilt_etc("<target_name>") {
  source = "<hap_file>"           # 源文件路径
  module_install_dir = "..."     # 安装目录
  part_name = "..."               # Part 名称
  subsystem_name = "..."         # 子系统名称
}
```

**代码证据**: `BUILD.gn:16-21` (launcher_hap 定义)

### 主要 Targets

#### 系统应用 Targets

| Target | Source | 安装路径 | 证据 |
|--------|--------|----------|------|
| launcher_hap | Launcher.hap | app/com.ohos.launcher | BUILD.gn:16-21 |
| launcher_settings_hap | Launcher_Settings.hap | app/com.ohos.launcher | BUILD.gn:23-28 |
| settings_hap | Settings.hap | app/com.ohos.settings | BUILD.gn:30-35 |
| settings_faceauth_hap | Settings_FaceAuth.hap | app/com.ohos.settings.faceauth | BUILD.gn:216-221 |

#### SystemUI Targets

| Target | Source | 安装路径 | 证据 |
|--------|--------|----------|------|
| navigationBar_hap | SystemUI-NavigationBar.hap | app/com.ohos.systemui | BUILD.gn:51-56 |
| statusBar_hap | SystemUI-StatusBar.hap | app/com.ohos.systemui | BUILD.gn:58-63 |
| screenLock_hap | SystemUI-ScreenLock.hap | app/com.ohos.systemui | BUILD.gn:65-70 |
| systemui_hap | SystemUI.hap | app/com.ohos.systemui | BUILD.gn:93-98 |
| systemDialog_hap | SystemUI-SystemDialog.hap | app/com.ohos.systemui | BUILD.gn:100-105 |

#### 资源 Targets

| Target | Source | 证据 |
|--------|--------|------|
| demo.wav | resources/demo.wav | BUILD.gn:142-146 |
| dynamic.wav | resources/dynamic.wav | BUILD.gn:148-152 |
| capture.ogg | resources/capture.ogg | BUILD.gn:161-165 |

### 聚合 Target

**hap 组目标** (`BUILD.gn:272-347`):

```gn
group("hap") {
  deps = [
    ":audiopicker_hap",
    ":calendarData_hap",
    # ... 更多依赖
  ]
  if (defined(product_name) && product_name == "watchos") {
    # 手表产品排除某些应用
  } else if (defined(product_name) && product_name == "rk3568") {
    deps += [ "//applications/standard/admin_provisioning:adminprovisioning_hap" ]
  } else if (defined(product_name) && product_name == "ohos-arm64") {
    deps += [ "//applications/standard/admin_provisioning:adminprovisioning_hap" ]
  }
}
```

### 产品变体配置

#### watchos (手表)

**排除的 HAP**:
- CalendarData, PrintSpooler, SecurityPrivacyCenter
- SystemDialog, UpdateApp
- 所有示例应用 (Calc, Clock, Music, kikaInput)
- 通信应用 (CallUI, Contacts, Mms, MobileDataSettings)

**证据**: `BUILD.gn:309-339`

#### rk3568 / ohos-arm64

**额外包含**:
- admin_provisioning (设备管理 provisioning)

**证据**: `BUILD.gn:340-346`

## ohos.build 清单

**位置**: `ohos.build:1-18`

**配置结构**:
```json
{
  "subsystem": "applications",
  "parts": {
    "prebuilt_hap": {
      "hisysevent_config": [
        "//applications/standard/hap/hisysevent/com.ohos.camera/hisysevent.yaml",
        "//applications/standard/hap/hisysevent/com.ohos.launcher/hisysevent.yaml",
        "//applications/standard/hap/hisysevent/com.ohos.systemui/hisysevent.yaml",
        "//applications/standard/hap/hisysevent/com.ohos.photos/hisysevent.yaml",
        "//applications/standard/hap/hisysevent/com.ohos.security.privacycenter/hisysevent.yaml"
      ],
      "module_list": [
        "//applications/standard/hap",
        "//foundation/communication/netmanager_ext/frameworks/vpn_dialog/dialog_ui/vpn_dialog:dialog_hap"
      ]
    }
  }
}
```

**证据**: `ohos.build:1-18`

## build.sh 独立构建

### 脚本功能

**位置**: `build.sh:1-565`

**主要功能**:
1. SDK 环境配置与下载
2. 项目依赖解析
3. npm/ohpm 包安装
4. HAP 编译 (hvigor/hvigorjs)
5. 签名处理

### 使用方式

#### 基本用法

```bash
# 从项目路径构建
./build.sh --project=/path/to/project

# 从 Git URL 构建
./build.sh --url=https://gitee.com/xxx/project --branch=master
```

#### 常用参数

| 参数 | 说明 | 证据 |
|------|------|------|
| `--project` | 项目路径 | build.sh:22 |
| `--sdk-path` | SDK 路径 | build.sh:23 |
| `--build-sdk` | 是否构建 SDK | build.sh:24 |
| `--url` | Git 仓库 URL | build.sh:26 |
| `--branch` | Git 分支 | build.sh:27 |
| `--out-path` | 输出路径 | build.sh:30 |
| `--sign-tool` | 签名工具路径 | build.sh:31 |

### 构建流程详解

#### SDK 配置 (行 145-269)

```bash
# NPM 仓库配置
npm config set registry https://repo.huaweicloud.com/repository/npm/
npm config set @ohos:registry https://repo.harmonyos.com/npm/
```

**证据**: `build.sh:146-148`

#### 项目解析 (行 362-461)

1. 读取 `build-profile.json5`
2. 解析 `srcPath` 获取模块列表
3. 识别 entry/feature 模块
4. 构建依赖图

```bash
# 模块输出识别
is_entry=`cat ${arg_project}${pa}/src/main/module.json5 | sed 's/ //g' | grep "\"type\":\"entry\""`
is_feature=`cat ${arg_project}${pa}/src/main/module.json5 | sed 's/ //g' | grep "\"type\":\"feature\""`
```

**证据**: `build.sh:421-426`

#### 编译执行 (行 464-492)

```bash
# hvigor 构建
./hvigorw assembleHap --mode module -p product=default -p debuggable=false --no-daemon
```

**证据**: `build.sh:486`

#### 签名处理 (行 494-553)

```bash
# 使用签名工具
java -jar hap-sign-tool.jar sign-app \
  -keyAlias "openharmony application release" \
  -signAlg "SHA256withECDSA" \
  -mode "localSign" \
  -profileFile "${arg_p7b}" \
  -inFile "${nosign_hap_path}" \
  -outFile "${sign_hap_path}"
```

**证据**: `build.sh:540-541`

### SDK 版本支持

| SDK 版本 | 路径 | 证据 |
|----------|------|------|
| 10 | prebuilts/ohos-sdk/linux/10 | build.sh:151-167 |
| 11 | prebuilts/ohos-sdk/linux/11 | build.sh:168-184 |
| 12 | prebuilts/ohos-sdk/linux/12 | build.sh:185-201 |
| 14 | prebuilts/ohos-sdk/linux/14 | build.sh:202-218 |
| 18 | prebuilts/ohos-sdk/linux/18 | build.sh:219-235 |
| 19 | prebuilts/ohos-sdk/linux/19 | build.sh:236-252 |
| 20 | prebuilts/ohos-sdk/linux/20 | build.sh:253-269 |

## 下一步阅读

- [06_Artifacts.md](./06_Artifacts.md) - 编译产物清单
- [07_Security_Review.md](./07_Security_Review.md) - 安全风险分析
