# 目录结构

## 顶层结构

```
applications/standard/hap/
├── resources/                      # 预制资源目录
├── hisysevent/                    # 系统事件配置
│   ├── com.ohos.systemui/         # SystemUI 事件定义
│   ├── com.ohos.launcher/         # Launcher 事件定义
│   ├── com.ohos.camera/          # Camera 事件定义
│   ├── com.ohos.photos/           # Photos 事件定义
│   ├── com.ohos.security.privacycenter/  # 隐私中心事件定义
│   └── com.huawei.MigrationAssistant/   # 迁移助手事件定义
├── *.hap                          # 预构建 HAP 包 (35+)
├── BUILD.gn                       # GN 构建配置
├── build.sh                       # HAP 构建脚本
├── ohos.build                     # 子系统构建清单
├── pipline.md                     # 流水线归档记录
├── README.md / README_zh.md       # 项目说明
└── wiki/                         # 本 Wiki 文档
```

## resources/ 目录详解

### 音频资源

| 文件 | 大小 | 用途 |
|------|------|------|
| `demo.wav` | ~7.5MB | 样本音频文件 |
| `dynamic.wav` | ~12MB | 动态音频文件 |
| `capture.ogg` | ~12KB | 相机捕获音效 |

### 模板资源

| 文件 | 类型 | 用途 |
|------|------|------|
| `downloadTemplate.abc` | ABC 字节码 | 下载模板 |
| `downloadTemplate.js` | JavaScript | 下载模板脚本 |
| `external.json` | JSON | 外部配置 |

### 代码证据

> "ohos_prebuilt_etc("demo.wav")" - `BUILD.gn:142-146`
> "ohos_prebuilt_etc("dynamic.wav")" - `BUILD.gn:148-152`
> "ohos_prebuilt_etc("capture.ogg")" - `BUILD.gn:161-165`

## hisysevent/ 目录详解

### 配置结构

每个应用的 `hisysevent.yaml` 定义了该应用需要上报的系统事件：

```
hisysevent/
├── com.ohos.systemui/
│   └── hisysevent.yaml        # SYSTEMUI_APP 域事件
├── com.ohos.launcher/
│   └── hisysevent.yaml        # LAUNCHER_APP 域事件
├── com.ohos.camera/
│   └── hisysevent.yaml        # CAMERA_APP 域事件
├── com.ohos.photos/
│   └── hisysevent.yaml        # PHOTOS_APP 域事件
├── com.ohos.security.privacycenter/
│   └── hisysevent.yaml        # SECURITY_APP 域事件
└── com.huawei.MigrationAssistant/
    └── hisysevent.yaml        # MIGRATION_APP 域事件
```

### 事件域配置

| 应用 | Domain | 事件类型 | 说明 |
|------|--------|----------|------|
| SystemUI | SYSTEMUI_APP | FAULT | UI 故障监控 |
| Launcher | LAUNCHER_APP | BEHAVIOR, FAULT | 启动行为 + 故障 |
| Camera | 待确认 | - | 相机相关事件 |
| Photos | 待确认 | - | 图库相关事件 |

### 代码证据

> "hisysevent_config: [ ... ]" - `ohos.build:5-11`

## HAP 文件分类

### 按功能分类

| 分类 | HAP 文件 | 安装路径 |
|------|----------|----------|
| 桌面 | Launcher.hap, Launcher_Settings.hap | app/com.ohos.launcher |
| SystemUI | SystemUI-*.hap (8个) | app/com.ohos.systemui |
| 设置 | Settings.hap, Settings_FaceAuth.hap | app/com.ohos.settings* |
| 媒体 | Camera.hap, Photos.hap | app/com.ohos.media* |
| 通信 | Contacts.hap, Mms.hap, CallUI.hap | app/com.ohos.comm* |
| 工具 | ScreenShot.hap, PrintSpooler.hap | app/com.ohos.tools* |

### 按类型分类

| 类型 | 数量 | 说明 |
|------|------|------|
| 系统应用 | 20+ | 核心系统功能 |
| 示例应用 | 5+ | 演示和参考用途 |
| 资源文件 | 3+ | 音频和模板 |

## 构建配置目录

### BUILD.gn

**位置**: 仓库根目录

**功能**: 定义所有预构建 HAP 目标的 GN 配置

**关键模式**:
```gn
ohos_prebuilt_etc("<target_name>") {
  source = "<hap_file>"
  module_install_dir = "app/<bundle_name>"
  part_name = "prebuilt_hap"
  subsystem_name = "applications"
}
```

**证据**: `BUILD.gn:16-21` (launcher_hap 定义)

### build.sh

**位置**: 仓库根目录

**功能**: 独立 HAP 构建脚本，支持从源码构建

**关键参数**:
- `--project`: 项目路径
- `--sdk-path`: SDK 路径
- `--build-sdk`: 构建 SDK
- `--url`: Git 仓库 URL
- `--branch`: Git 分支

### ohos.build

**位置**: 仓库根目录

**功能**: 子系统构建清单，定义 part 和模块列表

**配置内容**:
- hisysevent 配置文件列表
- 模块列表（包含 VPN dialog）

## wiki/ 目录结构

```
wiki/
├── README.md              # Wiki 说明
├── SUMMARY.md             # 全站导航
├── 00_Overview.md         # 项目概览
├── 01_Directory_Structure.md  # 目录结构（本文档）
├── 05_Build_System.md     # 构建系统
├── 06_Artifacts.md        # 编译产物
├── 07_Security_Review.md  # 安全评审
└── _work/                 # 工作笔记
    ├── NOTES.md           # 事实记录
    └── PLAN.md            # 任务计划
```

## 下一步阅读

- [05_Build_System.md](./05_Build_System.md) - GN 构建详解
- [06_Artifacts.md](./06_Artifacts.md) - 编译产物清单
- [07_Security_Review.md](./07_Security_Review.md) - 安全风险分析
