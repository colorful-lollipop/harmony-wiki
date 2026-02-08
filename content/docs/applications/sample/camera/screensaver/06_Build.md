# 构建系统

## 1. GN 构建配置

### 1.1 构建入口

**文件**：`BUILD.gn`

**关键 Targets**：
| Target | 类型 | 输出 | 说明 |
|--------|------|------|------|
| `screensaver` | shared_library | `libscreensaver.so` | C++ 共享库 |
| `screensaver_hap` | hap_pack | `screensaver.hap` | HAP 应用包 |

**证据**：`BUILD.gn:16-58`

### 1.2 screensaver target 详解

```gn
shared_library("screensaver") {
  sources = [
    "screensaver/src/main/cpp/screensaver_ability.cpp",
    "screensaver/src/main/cpp/screensaver_ability_slice.cpp",
  ]

  deps = [
    "${aafwk_lite_path}/frameworks/ability_lite:aafwk_abilitykit_lite",
    "${appexecfwk_lite_path}/frameworks/bundle_lite:bundle",
    "//foundation/arkui/ui_lite:ui_lite",
    "//foundation/communication/ipc/interfaces/innerkits/c/ipc:ipc_single",
    "//foundation/distributeddatamgr/kv_store/interfaces/inner_api/kv_store:kv_store",
    "//foundation/graphic/graphic_utils_lite:utils_lite",
    "//foundation/graphic/surface_lite",
    "//foundation/systemabilitymgr/samgr_lite/samgr:samgr",
  ]

  include_dirs = [
    "screensaver/src/main/cpp",
    "${aafwk_lite_path}/interfaces/kits/ability_lite",
    "${appexecfwk_lite_path}/interfaces/kits/bundle_lite",
    "${aafwk_lite_path}/interfaces/kits/want_lite",
    "//base/startup/syspara_lite/interfaces/kits",
  ]

  defines = [
    "ENABLE_WINDOW=1",
    "ABILITY_WINDOW_SUPPORT",
    "OHOS_APPEXECFWK_BMS_BUNDLEMANAGER",
  ]
}
```

**证据**：`BUILD.gn:16-46`

### 1.3 screensaver_hap target 详解

```gn
hap_pack("screensaver_hap") {
  deps = [ ":screensaver" ]
  mode = "hap"
  json_path = "screensaver/src/main/config.json"
  ability_so_path = "$root_out_dir/libscreensaver.so"
  force = "true"
  cert_profile = "cert/com.huawei.screensaver_AppProvision_release.p7b"
  resources_path = "screensaver/src/main/resources"
  hap_name = "screensaver"
  privatekey = "HOS Application Provision Release"
}
```

**证据**：`BUILD.gn:48-58`

---

## 2. 组件配置

### 2.1 bundle.json

**文件**：`bundle.json`

```json
{
    "name": "@ohos/camera_screensaver_app",
    "description": "screensaver related samples for small system.",
    "version": "3.1",
    "component": {
        "name": "camera_screensaver_app",
        "subsystem": "applications",
        "adapted_system_type": ["mini", "small"],
        "build": {
            "sub_component": [
                "//applications/sample/camera/screensaver:screensaver_hap"
            ],
            "inner_kits": [],
            "test": []
        }
    }
}
```

**证据**：`bundle.json:1-50`

### 2.2 依赖组件

| 组件 | 系统依赖 | 用途 |
|------|----------|------|
| ability_lite | 是 | Ability 框架 |
| bundle_framework_lite | 是 | Bundle 框架 |
| surface_lite | 是 | 图形表面 |
| ui_lite | 是 | UI 组件 |
| graphic_utils_lite | 是 | 图形工具 |
| kv_store | 是 | 键值存储 |
| syspara_lite | 是 | 系统参数 |
| samgr_lite | 是 | 系统能力管理 |
| utils_base | 是 | 基础工具 |

### 2.3 第三方依赖

| 库 | 用途 |
|----|------|
| libjpeg | JPEG 图像解码 |
| libpng | PNG 图像解码 |
| giflib | GIF 动画支持 |
| cjson | JSON 解析 |
| bounds_checking_function | 安全字符串函数 |

---

## 3. 编译产物

### 3.1 产物清单

| 产物 | 路径 | 类型 | 说明 |
|------|------|------|------|
| libscreensaver.so | `out/.../libscreensaver.so` | 共享库 | C++ 实现 |
| screensaver.hap | `out/.../screensaver.hap` | 应用包 | 可安装应用 |
| 资源文件 | `screensaver/src/main/resources/` | 目录 | 图片、字符串 |

### 3.2 运行时加载关系

```
screensaver.hap
    │
    ├── libscreensaver.so (Ability 实现)
    │       │
    │       ├── libabilitykit_lite.so (Ability 框架)
    │       ├── libui_lite.so (UI 框架)
    │       ├── libsurface_lite.so (图形表面)
    │       └── ...
    │
    ├── resources/ (资源文件)
    │       └── base/media/img_default_*.png
    │
    └── config.json (应用配置)
```

### 3.3 安装路径

HAP 安装后文件布局：
```
/storage/app/run/com.huawei.screensaver/
├── screensaver/
│   ├── assets/
│   │   └── screensaver/
│   │       └── resources/
│   │           └── base/media/
│   │               └── img_default_*.png
│   └── libscreensaver.so
```

**证据**：`ui_config.h:25-34`

---

## 4. 构建命令

### 4.1 使用 hb 构建

```bash
# 完整构建
hb build -f

# 指定产品
hb build -p <product_name>
```

### 4.2 使用 GN 直接构建

```bash
python build.py -p ipcamera_hi3516dv300 -b release
```

### 4.3 查看产物

```bash
# 查看 out 目录
ls out/<product>/applications/sample/camera/screensaver/
```

---

## 5. 文档导航

- **返回**：[内部 API](05_Inner_API.md) → 模块接口
- **下一步**：[安全评审](07_Security.md) → 安全风险
- **相关**：[故障排查](08_Troubleshooting.md) → 构建问题
