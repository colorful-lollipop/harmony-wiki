# 配置项与宏定义

## 1. 编译宏定义

### 1.1 BUILD.gn 中的 defines

**文件**：`BUILD.gn:41-45`

```gn
defines = [
  "ENABLE_WINDOW=1",
  "ABILITY_WINDOW_SUPPORT",
  "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
]
```

| 宏 | 值 | 用途 |
|----|-----|------|
| ENABLE_WINDOW | 1 | 启用窗口支持 |
| ABILITY_WINDOW_SUPPORT | - | Ability 窗口支持 |
| OHOS_APPEXECFWK_BMS_BUNDLEMANAGER | - | BundleManager 支持 |

---

## 2. UI 配置常量

### 2.1 动画参数

**文件**：`ui_config.h:22-23`

```cpp
static constexpr uint16_t IMAGE_ANIMATOR_TIME_S = 2 * 1000;  // 2秒
static constexpr uint8_t IMAGE_TOTEL_NUM = 5;                // 5张图片
```

| 常量 | 值 | 说明 |
|------|-----|------|
| IMAGE_ANIMATOR_TIME_S | 2000ms | 图片切换间隔 |
| IMAGE_TOTEL_NUM | 5 | 屏保图片数量 |

### 2.2 图片路径

**文件**：`ui_config.h:25-34`

```cpp
static const char* const IMG_DEFAULT_001_PATH = "/storage/app/run/com.huawei.screensaver/...";
static const char* const IMG_DEFAULT_002_PATH = "/storage/app/run/com.huawei.screensaver/...";
static const char* const IMG_DEFAULT_003_PATH = "/storage/app/run/com.huawei.screensaver/...";
static const char* const IMG_DEFAULT_004_PATH = "/storage/app/run/com.huawei.screensaver/...";
static const char* const IMG_DEFAULT_005_PATH = "/storage/app/run/com.huawei.screensaver/...";
```

| 常量 | 路径格式 | 说明 |
|------|----------|------|
| IMG_DEFAULT_001_PATH | `/storage/app/run/com.huawei.screensaver/screensaver/assets/screensaver/resources/base/media/img_default_1.png` | 第1张屏保图片 |
| IMG_DEFAULT_002_PATH | 同上格式 | 第2张屏保图片 |
| ... | 同上格式 | ... |

---

## 3. 应用配置

### 3.1 config.json

**文件**：`screensaver/src/main/config.json`

```json
{
    "app": {
        "bundleName": "com.huawei.screensaver",
        "vendor": "huawei",
        "version": {
            "code": 1,
            "name": "1.0"
        },
        "apiVersion": {
            "compatible": 3,
            "target": 4
        }
    },
    "module": {
        "package": "com.huawei.screensaver",
        "name": ".MyHarmonyAbilityPackage",
        "deviceType": ["phone", "tv", "tablet", "car", "smartWatch", "sportsWatch", "smartVision"],
        "distro": {
            "deliveryWithInstall": true,
            "moduleName": "screensaver",
            "moduleType": "entry"
        },
        "abilities": [{
            "name": "ScreensaverAbility",
            "label": "screensaver",
            "launchType": "standard",
            "type": "page",
            "visible": true
        }]
    }
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| bundleName | com.huawei.screensaver | 应用包名 |
| version.code | 1 | 版本号 |
| deviceType | phone, tv, ... | 支持的设备类型 |
| moduleType | entry | 模块类型 |
| launchType | standard | 启动类型 |
| type | page | Ability 类型 |

---

## 4. bundle.json 配置

**文件**：`bundle.json`

```json
{
    "name": "@ohos/camera_screensaver_app",
    "version": "3.1",
    "component": {
        "name": "camera_screensaver_app",
        "subsystem": "applications",
        "adapted_system_type": ["mini", "small"],
        "build": {
            "sub_component": ["//applications/sample/camera/screensaver:screensaver_hap"]
        }
    }
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| subsystem | applications | 所属子系统 |
| adapted_system_type | mini, small | 适配系统类型 |
| sub_component | screensaver_hap | 构建入口 |

---

## 5. 文档导航

- **返回**：[构建系统](06_Build.md) → 构建配置
- **相关**：[概览](01_Overview.md) → 项目配置
