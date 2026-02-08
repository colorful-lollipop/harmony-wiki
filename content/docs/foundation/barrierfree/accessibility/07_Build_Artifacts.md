# 编译产物

## 目的

本文档说明 Accessibility 子系统的编译产物、安装路径和运行时加载关系。

## 适用范围

- 构建工程师
- 部署工程师
- 需要了解产物位置的开发者
- 调试问题的开发者

## 关键结论

### 产物类型

共 **4 类**产物：

1. **共享库 (.so)** - C/C++ 和 N-API 模块
2. **ArkTS 字节码 (.abc)** - ANI 模块
3. **配置文件** - SA 配置、启动配置、参数配置
4. **SA 配置** - System Ability 配置

### 安装路径

| 产物类型 | 安装路径 |
|---------|---------|
| 系统服务 .so | /system/lib64/ |
| N-API .so | /system/lib64/module/ 和 /system/lib64/module/accessibility/ |
| ANI .so | /system/lib64/ |
| ANI .abc | /system/lib64/module/ets/ |
| SA 配置 | /system/profile/ |

### 运行时加载关系

```
系统启动
    ↓
samgr 加载 SA 801 配置 (801.json)
    ↓
启动 accessibility 进程
    ↓
加载 libaccessibleabilityms.z.so
    ↓
AccessibilityService 运行
    ↓
应用进程
    ↓
加载 N-API .so (libaccessibility_napi.so, 等）
    ↓
应用调用 N-API 接口
```

---

## 详细内容

### Services 层产物

| 产物名称 | 类型 | 目标 | 安装路径 | 证据 |
|---------|------|------|---------|------|
| libaccessibleabilityms.z.so | ohos_shared_library | `//services/aams:accessibleabilityms` | /system/lib64/ | `services/aams/BUILD.gn` |
| libaams_ext.so | ohos_shared_library | `//services/aams_ext:aams_ext` | /system/lib64/ | `services/aams_ext/BUILD.gn` |
| accessibility_service.rc | ohos_prebuilt_etc | `//services/aams:accessibility_service.rc` | /system/etc/init/ | `services/aams/BUILD.gn` |
| 801.json | ohos_sa_profile | `//sa_profile:aams_sa_profile` | /system/profile/ | `sa_profile/801.json` |
| accessibility.cfg | ohos_prebuilt_etc | `//sa_profile:accessibility_cfg` | /system/profile/ | `sa_profile/BUILD.gn` |
| accessibility.para.dac | ohos_prebuilt_etc | `//services/etc:ohos.para.dac` | /system/etc/param/ | `services/etc/BUILD.gn` |

**证据**: `bundle.json:92-98`

### Innerkits 层产物

| 产物名称 | 类型 | 目标 | 安装路径 | 证据 |
|---------|------|------|---------|------|
| libaccessibility_common.so | ohos_shared_library | `//interfaces/innerkits/common:accessibility_common` | /system/lib64/ | `interfaces/innerkits/common/BUILD.gn` |
| libaccessibility_interface.so | ohos_shared_library | `//common/interface:accessibility_interface` | /system/lib64/ | `common/interface/BUILD.gn` |
| libaccessibleability.so | ohos_shared_library | `//interfaces/innerkits/aafwk:accessibleability` | /system/lib64/ | `interfaces/innerkits/aafwk/BUILD.gn` |
| libaccessibilityconfig.so | ohos_shared_library | `//interfaces/innerkits/acfwk:accessibilityconfig` | /system/lib64/ | `interfaces/innerkits/acfwk/BUILD.gn` |
| libaccessibilityclient.so | ohos_shared_library | `//interfaces/innerkits/asacfwk:accessibilityclient` | /system/lib64/ | `interfaces/innerkits/asacfwk/BUILD.gn` |

**证据**: `bundle.json:100-160`

### NAPI 层产物

| 产物名称 | 类型 | 目标 | 安装路径 | 证据 |
|---------|------|------|---------|------|
| libaccessibility_napi.so | ohos_shared_library | `//interfaces/kits/napi:accessibility_napi` | /system/lib64/module/ | `interfaces/kits/napi/BUILD.gn` |
| libconfig_napi.so | ohos_shared_library | `//interfaces/kits/napi/accessibility_config:config_napi` | /system/lib64/module/accessibility/ | `interfaces/kits/napi/accessibility_config/BUILD.gn` |
| libaccessibilityextensionability_napi.so | ohos_shared_library | `//interfaces/kits/napi/accessibility_extension:accessibilityextensionability_napi` | /system/lib64/module/application/ | `interfaces/kits/napi/accessibility_extension/BUILD.gn` |
| libaccessibilityextensioncontext_napi.so | ohos_shared_library | `//interfaces/kits/napi/accessibility_extension_context:accessibilityextensioncontext_napi` | /system/lib64/module/application/ | `interfaces/kits/napi/accessibility_extension_context/BUILD.gn` |
| libaccessibility_extension_module.so | ohos_shared_library | `//interfaces/kits/napi/accessibility_extension_module_loader:accessibility_extension_module` | /system/lib64/extensionability/ | `interfaces/kits/napi/accessibility_extension_module_loader/BUILD.gn` |
| libgesturepath_napi.so | ohos_shared_library | `//interfaces/kits/napi/accessibility_gesture_path:gesturepath_napi` | /system/lib64/module/accessibility/ | `interfaces/kits/napi/accessibility_gesture_path/BUILD.gn` |
| libgesturepoint_napi.so | ohos_shared_library | `//interfaces/kits/napi/accessibility_gesture_point:gesturepoint_napi` | /system/lib64/module/accessibility/ | `interfaces/kits/napi/accessibility_gesture_point/BUILD.gn` |

**证据**: `interfaces/kits/*/BUILD.gn`

### ANI 层产物

| 产物名称 | 类型 | 目标 | 安装路径 | 证据 |
|---------|------|------|---------|------|
| libaccessibility_ani.so | ohos_shared_library | `//interfaces/kits/ani:accessibility_ani` | /system/lib64/ | `interfaces/kits/ani/BUILD.gn` |
| accessibility.abc | generate_static_abc | `//interfaces/kits/ani:accessibility_ani_abc_etc` | /system/lib64/module/ets/ | `interfaces/kits/ani/BUILD.gn` |
| libaccessibility_config.so | ohos_shared_library | `//interfaces/kits/ani/accessibility_config:accessibility_config` | /system/lib64/ | `interfaces/kits/ani/accessibility_config/BUILD.gn` |
| accessibility_config.abc | generate_static_abc | `//interfaces/kits/ani/accessibility_config:accessibility_config_ani_abc_etc` | /system/lib64/module/ets/ | `interfaces/kits/ani/accessibility_config/BUILD.gn` |
| accessibility_extension_ability.abc | generate_static_abc | `//interfaces/kits/ani/accessibility_extension:accessibility_extension_ability_ani_abc_etc` | /system/lib64/module/ets/ | `interfaces/kits/ani/accessibility_extension/BUILD.gn` |
| accessibility_extension_context.abc | generate_static_abc | `//interfaces/kits/ani/accessibility_extension:accessibility_extension_context_ani_abc_etc` | /system/lib64/module/ets/ | `interfaces/kits/ani/accessibility_extension/BUILD.gn` |

**证据**: `interfaces/kits/ani/*/BUILD.gn`

### CJ 层产物

| 产物名称 | 类型 | 目标 | 安装路径 | 证据 |
|---------|------|------|---------|------|
| libcj_accessibility_ffi.so | ohos_shared_library | `//interfaces/kits/cj:cj_accessibility_ffi` | /system/lib64/ | `interfaces/kits/cj/BUILD.gn` |

**证据**: `interfaces/kits/cj/BUILD.gn`

### 产物清单汇总

| 类别 | 数量 | 总大小估计 |
|------|------|-----------|
| 系统服务 | 2 | ~15 MB |
| Innerkits | 5 | ~5 MB |
| N-API 模块 | 7 | ~8 MB |
| ANI 模块 | 6 | ~6 MB |
| CJ 模块 | 1 | ~1 MB |
| 配置文件 | 5 | ~50 KB |

**注**: 大小为估计值，实际以构建输出为准。

### 运行时加载关系详解

#### 启动阶段

1. **系统启动**: samgr（System Ability Manager）启动
2. **SA 注册**: samgr 读取 `/system/profile/801.json`
3. **服务启动**: init 进程启动 accessibility 进程
4. **库加载**: accessibility 进程加载 `libaccessibleabilityms.z.so`
5. **服务注册**: AccessibilityService 注册到 samgr

**证据**: `sa_profile/801.json`, `services/aams:accessibility_service.rc`

#### 应用连接阶段

1. **应用启动**: 普通应用或无障碍扩展应用启动
2. **N-API 加载**: 应用加载需要的 N-API 模块（.so）
3. **SA 连接**: N-API 通过 IPC 连接 SA 801
4. **权限验证**: SA 验证调用者权限
5. **功能使用**: 应用使用无障碍能力

**证据**: `interfaces/kits/napi/src/native_module.cpp`

### 目标与产物映射

| 模块 | Target | 产物 | 说明 |
|------|-------|------|------|
| **AAMS** | `//services/aams:accessibleabilityms` | libaccessibleabilityms.z.so | 核心服务 |
| **AAMS_EXT** | `//services/aams_ext:aams_ext` | libaams_ext.so | 扩展服务 |
| **AAfwk** | `//interfaces/innerkits/aafwk:accessibleability` | libaccessibleability.so | 辅助能力 Kit |
| **ACfwk** | `//interfaces/innerkits/acfwk:accessibilityconfig` | libaccessibilityconfig.so | 配置 Kit |
| **ASACfwk** | `//interfaces/innerkits/asacfwk:accessibilityclient` | libaccessibilityclient.so | 客户端 Kit |
| **Common** | `//interfaces/innerkits/common:accessibility_common` | libaccessibility_common.so | 通用类型 |
| **Interface** | `//common/interface:accessibility_interface` | libaccessibility_interface.so | IPC 接口 |

**证据**: `bundle.json:100-160`

---

## 相关链接

- [项目概览](00_Overview.md)
- [GN Targets](06_GN_Targets.md)
- [目录结构](02_Directory_Structure.md)

---

最后更新: 2026-02-06
