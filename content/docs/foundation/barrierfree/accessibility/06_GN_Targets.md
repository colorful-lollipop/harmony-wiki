# GN 构建系统

## 目的

本文档详细说明 Accessibility 子系统的 GN 构建系统，包括主要 targets、类型分类、依赖关系和配置参数。

## 适用范围

- 构建工程师
- 需要理解构建系统的开发者
- 需要添加新模块的开发者

## 关键结论

### 主要 Targets 分类

共 **5 类** targets：

1. **Services 层** - 系统服务 targets
2. **Innerkits 层** - C/C++ 内部接口 targets
3. **NAPI 层** - JS/TS 接口 targets
4. **ANI 层** - ArkTS Native Interface targets
5. **CJ 层** - Cangjie FFI targets

### 依赖方向

```
accessibility_common (最底层）
    ↓
accessibility_interface
    ↓
accessibleability / accessibilityconfig / accessibilityclient
    ↓
accessibleabilityms / 所有 Kits
```

### 配置参数

主要特性开关定义在 `accessibility_manager_service.gni`。

---

## 详细内容

### 1. Services 层 Targets

#### accessibleabilityms

| 属性 | 值 |
|------|-----|
| 目标路径 | `//services/aams:accessibleabilityms` |
| 类型 | ohos_shared_library |
| 输出名称 | libaccessibleabilityms.z.so |
| 主要依赖 | accessibility_interface, accessibility_common |
| 外部依赖 | window_manager, input, eventhandler, 等 30+ 个 |

**证据**: `services/aams/BUILD.gn`

#### aams_ext

| 属性 | 值 |
|------|-----|
| 目标路径 | `//services/aams_ext:aams_ext` |
| 类型 | ohos_shared_library |
| 输出名称 | libaams_ext.so |
| 主要依赖 | graphic_2d, window_manager |
| 说明 | 扩展服务（放大镜窗口/菜单） |

**证据**: `services/aams_ext/BUILD.gn`

#### SA 配置

| 目标 | 类型 | 输出 | 说明 |
|------|------|------|------|
| `//services/aams:accessibility_service.rc` | ohos_prebuilt_etc | accessibility_service.rc | 服务启动配置 |
| `//sa_profile:aams_sa_profile` | ohos_sa_profile | 801.json | SA 801 配置 |
| `//sa_profile:accessibility_cfg` | ohos_prebuilt_etc | accessibility.cfg | SA 配置 |
| `//services/etc:ohos.para.dac` | ohos_prebuilt_etc | accessibility.para.dac | 参数配置 |

**证据**: `services/aams/BUILD.gn`, `sa_profile/BUILD.gn`

### 2. Innerkits 层 Targets

#### accessibility_common

| 属性 | 值 |
|------|-----|
| 目标路径 | `//interfaces/innerkits/common:accessibility_common` |
| 类型 | ohos_shared_library |
| 输出名称 | libaccessibility_common.so |
| 主要源文件 | accessibility_element_info.cpp, accessibility_event_info.cpp, 等 |
| 依赖 | 基础 external_deps（c_utils, hilog, 等） |

**证据**: `interfaces/innerkits/common/BUILD.gn`

#### accessibility_interface

| 属性 | 值 |
|------|-----|
| 目标路径 | `//common/interface:accessibility_interface` |
| 类型 | ohos_shared_library |
| 输出名称 | libaccessibility_interface.so |
| 主要源文件 | 所有 Proxy/Stub 实现 + Parcel 类 |
| 依赖 | accessibility_common + IDL 生成代码 |

**证据**: `common/interface/BUILD.gn`

#### accessibleability (AAkit)

| 属性 | 值 |
|------|-----|
| 目标路径 | `//interfaces/innerkits/aafwk:accessibleability` |
| 类型 | ohos_shared_library |
| 输出名称 | libaccessibleability.so |
| 主要依赖 | accessibility_interface, accessibility_common |
| 公开头文件 | `interfaces/innerkits/aafwk/include/` |

**证据**: `interfaces/innerkits/aafwk/BUILD.gn`

#### accessibilityconfig (ACkit)

| 属性 | 值 |
|------|-----|
| 目标路径 | `//interfaces/innerkits/acfwk:accessibilityconfig` |
| 类型 | ohos_shared_library |
| 输出名称 | libaccessibilityconfig.so |
| 主要依赖 | accessibility_interface, accessibility_common |

**证据**: `interfaces/innerkits/acfwk/BUILD.gn`

#### accessibilityclient (ASACkit)

| 属性 | 值 |
|------|-----|
| 目标路径 | `//interfaces/innerkits/asacfwk:accessibilityclient` |
| 类型 | ohos_shared_library |
| 输出名称 | libaccessibilityclient.so |
| 主要依赖 | accessibility_interface, accessibility_common |

**证据**: `interfaces/innerkits/asacfwk/BUILD.gn`

### 3. NAPI 层 Targets

#### 主 N-API 模块

| 目标 | 输出名 | 安装路径 | 证据 |
|------|---------|---------|------|
| `//interfaces/kits/napi:accessibility_napi` | libaccessibility_napi.so | module/ | `interfaces/kits/napi/BUILD.gn` |

#### 配置 N-API

| 目标 | 输出名 | 安装路径 | 证据 |
|------|---------|---------|------|
| `//interfaces/kits/napi/accessibility_config:config_napi` | libconfig_napi.so | module/accessibility/ | `interfaces/kits/napi/accessibility_config/BUILD.gn` |

#### Extension N-API

| 目标 | 输出名 | 安装路径 | 证据 |
|------|---------|---------|------|
| `//interfaces/kits/napi/accessibility_extension:accessibilityextensionability_napi` | libaccessibilityextensionability_napi.so | module/application/ | `interfaces/kits/napi/accessibility_extension/BUILD.gn` |
| `//interfaces/kits/napi/accessibility_extension_context:accessibilityextensioncontext_napi` | libaccessibilityextensioncontext_napi.so | module/application/ | `interfaces/kits/napi/accessibility_extension_context/BUILD.gn` |
| `//interfaces/kits/napi/accessibility_extension_module_loader:accessibility_extension_module` | libaccessibility_extension_module.so | extensionability/ | `interfaces/kits/napi/accessibility_extension_module_loader/BUILD.gn` |

#### 手势 N-API

| 目标 | 输出名 | 安装路径 | 证据 |
|------|---------|---------|------|
| `//interfaces/kits/napi/accessibility_gesture_path:gesturepath_napi` | libgesturepath_napi.so | module/accessibility/ | `interfaces/kits/napi/accessibility_gesture_path/BUILD.gn` |
| `//interfaces/kits/napi/accessibility_gesture_point:gesturepoint_napi` | libgesturepoint_napi.so | module/accessibility/ | `interfaces/kits/napi/accessibility_gesture_point/BUILD.gn` |

#### Group Targets

| 目标 | 说明 | 证据 |
|------|------|------|
| `//interfaces/kits/napi:napi_packages` | 聚合所有 N-API targets | `interfaces/kits/napi/BUILD.gn` |

### 4. ANI 层 Targets

| 目标 | 输出名 | 证据 |
|------|---------|------|
| `//interfaces/kits/ani:accessibility_ani` | libaccessibility_ani.so | `interfaces/kits/ani/BUILD.gn` |
| `//interfaces/kits/ani:accessibility_ani_abc_etc` | accessibility.abc | `interfaces/kits/ani/BUILD.gn` |
| `//interfaces/kits/ani/accessibility_config:accessibility_config` | libaccessibility_config.so | `interfaces/kits/ani/accessibility_config/BUILD.gn` |
| `//interfaces/kits/ani/accessibility_config:accessibility_config_ani_abc_etc` | accessibility_config.abc | `interfaces/kits/ani/accessibility_config/BUILD.gn` |
| `//interfaces/kits/ani/accessibility_extension:accessibility_extension_ability_ani_abc_etc` | accessibility_extension_ability.abc | `interfaces/kits/ani/accessibility_extension/BUILD.gn` |
| `//interfaces/kits/ani/accessibility_extension:accessibility_extension_context_ani_abc_etc` | accessibility_extension_context.abc | `interfaces/kits/ani/accessibility_extension/BUILD.gn` |

#### Group Targets

| 目标 | 说明 | 证据 |
|------|------|------|
| `//interfaces/kits/ani:ani_packages` | 聚合所有 ANI targets | `interfaces/kits/ani/BUILD.gn` |

### 5. CJ 层 Targets

| 目标 | 输出名 | 证据 |
|------|---------|------|
| `//interfaces/kits/cj:cj_accessibility_ffi` | libcj_accessibility_ffi.so | `interfaces/kits/cj/BUILD.gn` |

### 依赖关系图

```
bundle.json
    |
    +-------+-------+-------+
    |       |       |       |
base_group  fwk_group  service_group
    |       |       |
    |       |       +---- aams_sa_profile
    |       |       |
    |       |       +---- accessibleabilityms
    |       |       |
    |       +---- aams_ext
    |
    +---- ohos.para.dac
    |
    +---- api_event_etc
    |
    +---- ani_packages
    |
    +---- napi_packages
    |
    +---- accessibleability (AAfwk)
    |
    +---- accessibilityconfig (ACfwk)
    |
    +---- accessibilityclient (ASACfwk)
    |
    +---- accessibility_common
    |
    +---- accessibility_interface
```

**证据**: `bundle.json:79-98`

### 配置参数

#### accessibility_manager_service.gni

| 参数名 | 默认值 | 说明 |
|--------|---------|------|
| accessibility_feature_power_manager | true | 电源管理功能 |
| accessibility_feature_display_manager | true | 显示管理功能 |
| accessibility_feature_data_share | true | 数据共享功能 |
| accessibility_use_rosen_drawing | false | Rosen 绘图 |
| accessibility_watch_feature | false | 手表特性 |
| accessibility_feature_hiviewdfx_hitrace | true | HiTrace 追踪 |
| accessibility_feature_hiviewdfx_hisysevent | true | HiSysEvent 事件 |
| accessibility_dynamic_support | false | 动态支持 |
| security_component_enable | false | 安全组件 |

**证据**: `accessibility_manager_service.gni`

#### accessibility_aafwk.gni

定义 AAFWK 相关路径。

**证据**: `accessibility_aafwk.gni`

### Bundle.json Group 定义

| Group | Targets |
|-------|----------|
| base_group | `//interfaces/kits/ani:ani_packages`, `//interfaces/kits/napi:napi_packages` |
| fwk_group | `//interfaces/innerkits/aafwk:accessibleability`, `//interfaces/innerkits/acfwk:accessibilityconfig`, `//interfaces/innerkits/asacfwk:accessibilityclient`, `//interfaces/innerkits/common:accessibility_common`, `//common/interface:accessibility_interface`, `//common/etc:api_event_etc` |
| service_group | `//sa_profile:aams_sa_profile`, `//sa_profile:accessibility_cfg`, `//services/aams:accessibleabilityms`, `//services/etc:ohos.para.dac`, `//services/aams_ext:aams_ext` |

**证据**: `bundle.json:79-98`

---

## 相关链接

- [项目概览](00_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [编译产物](07_Build_Artifacts.md)
- [附录：配置标志](appendix/Config_Flags.md)

---

最后更新: 2026-02-06
