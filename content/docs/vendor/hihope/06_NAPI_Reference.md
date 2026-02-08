# 对外 N-API（JavaScript API）参考

## 文档信息

- **目的**：说明 vendor_hihope 仓库中的 N-API（JavaScript API）引用情况
- **适用范围**：vendor_hihope 仓库
- **最后更新**：2025-02-06
- **关键结论**：
  - vendor_hihope 仓库**不包含 N-API 实现**
  - vendor_hihope 仓库仅在 `sanitizer_check_list.gni` 中**引用**外部 N-API 模块
  - 实际 N-API 实现在 OpenHarmony 主源码树中

## N-API 概述

### N-API 定义

N-API（Native API）是 OpenHarmony 提供的 JavaScript/ArkTS 与原生 C/C++ 代码绑定的机制。

**作用**：
- 允许 ArkTS 应用调用高性能的原生代码
- 提供跨语言互操作性
- 封装系统底层能力给应用层

### vendor_hihope 仓库中的 N-API

**重要说明**：
- ⚠️ vendor_hihope 仓库**不包含任何 N-API 实现**
- ✅ vendor_hihope 仓库仅在 `sanitizer_check_list.gni` 中**列出**外部 N-API 模块
- 📁 实际 N-API 实现在 OpenHarmony 主源码树中

### N-API 引用证据

**配置文件**：`rk3568/security_config/sanitizer_check_list.gni`

该文件列出了 vendor 仓库产品引用的 N-API 模块，但这些模块**不在** vendor_hihope 仓库中实现。

### 引用的 N-API 模块分类

| 模块类别 | N-API 模块（引用的） | 实际实现位置 |
|----------|---------------|----------|
| **Ability Runtime** | ability_napi, abilitycontext_napi, abilitymanager_napi, missionmanager_napi | foundation/ability/ability_runtime |
| **Camera** | camera_napi | foundation/multimedia/camera_framework |
| **Audio Framework** | audio_framework_napi, avsession_napi | foundation/multimedia/audio_framework |
| **Graphics** | drawing_napi, text_napi | foundation/graphic/graphic_2d |
| **Crypto** | cipher_napi, cryptoframework_napi | base/security/crypto_framework |
| **User Auth** | userauthextensionability_napi, pin_auth_interface_napi, user_auth_interface_napi, fingerprint_auth_interface_napi | base/useriam/user_auth_framework |
| **Log** | libhilognapi_src | base/hiviewdfx/hilog |
| **System** | napi_utils | foundation/communication/netmanager_base |
| **Location** | location_gnss_napi, location_agnss_napi | foundation/geolocation |
| **Input** | input_napi | foundation/multimedia/input |
| **Network** | netmanager_ext_napi | foundation/communication/netmanager_ext |
| **File Access** | dlpm_permission_service | base/filemanagement |
| **HDF** | hdf_core_napi | drivers/hdf_core |

## N-API 实现位置（不在本仓库）

### Ability Runtime N-API

**实现位置**：`foundation/ability/ability_runtime`

**模块清单**：
- `ability_napi` - Ability 基础 N-API 绑定
- `abilitycontext_napi` - Ability Context N-API
- `abilitymanager_napi` - Ability Manager N-API
- `serviceextensionability_napi` - Service Extension Ability N-API
- `dataability_napi` - Data Ability N-API
- `continueability_napi` - Continue Ability N-API

**主要 N-API 函数**：
- `napi_define_properties` - 定义导出的属性和方法
- `napi_create_function` - 创建 N-API 函数
- `napi_call_function` - 调用 N-API 函数
- `napi_get_cb_info` - 获取回调信息
- `napi_get_value_*` - 获取 N-API 值
- `napi_set_value_*` - 设置 N-API 值

### Camera N-API

**实现位置**：`foundation/multimedia/camera_framework`

**模块清单**：
- `camera_napi` - 相机 N-API 绑定

**主要功能**：
- 相机预览
- 相机拍照
- 录像功能
- 相机配置

### Audio Framework N-API

**实现位置**：`foundation/multimedia/audio_framework`

**模块清单**：
- `audio_framework_napi` - 音频框架 N-API 绑定
- `avsession_napi` - 音频会话 N-API 绑定

**主要功能**：
- 音频播放
- 音频录制
- 音频管理
- 音效控制

### Graphics N-API

**实现位置**：`foundation/graphic/graphic_2d`

**模块清单**：
- `drawing_napi` - 2D 绘图 N-API 绑定
- `text_napi` - 文本 N-API 绑定

**主要功能**：
- Canvas 2D 绘图
- 文本渲染
- 图像处理

### Crypto Framework N-API

**实现位置**：`base/security/crypto_framework`

**模块清单**：
- `cipher_napi` - 加密 N-API 绑定
- `cryptoframework_napi` - 加密框架 N-API 绑定

**主要功能**：
- 对称加密
- 非对称加密
- 哈希算法
- 签名验证

### User Auth Framework N-API

**实现位置**：`base/useriam/user_auth_framework`

**模块清单**：
- `userauthextensionability_napi` - 用户认证扩展 N-API
- `pin_auth_interface_napi` - PIN 认证 N-API
- `user_auth_interface_napi` - 用户认证 N-API
- `fingerprint_auth_interface_napi` - 指纹认证 N-API

**主要功能**：
- 用户认证
- PIN 码认证
- 指纹识别
- 人脸识别

## N-API 注册机制

### 注册流程

```
┌─────────────────────────────────────┐
│  ArkTS 应用                 │
│  (JavaScript/ArkTS)           │
└────────────┬────────────────────┘
             │
        ┌────────────▼──────────────┐
        │  N-API Binding 层       │  [OpenHarmony 框架]
        │  （napi_xxx 模块）       │
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │   N-API 框架           │  [OpenHarmony 框架]
        │  （napi_api.h）          │
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │   V8 引擎绑定层         │  [OpenHarmony 框架]
        │  （napi_*.cpp）           │
        │                           │
        └────────────┬──────────────────┘
             │
        ┌────────────▼──────────────┐
        │    C/C++ 实现层        │  [OpenHarmony 框架]
        │  （系统服务/框架）         │
        └───────────────────────────────┘
```

### 注册 API

| API | 说明 | 参数 |
|-----|------|------|
| `napi_module_register` | 注册 N-API 模块 |
| `napi_define_properties` | 定义导出的属性和方法 |
| `napi_create_function` | 创建 N-API 函数 |
| `napi_set_named_property` | 设置命名属性 |

## N-API 参数校验

### 参数校验机制

**N-API 框架提供的校验**：
- ✅ 自动类型检查（基础类型 vs 对象类型）
- ✅ 参数数量检查
- ✅ 空值检查
- ✅ 可选参数处理
- ✅ 回调函数校验

### 常见校验规则

| 校验项 | 说明 | 实现方式 |
|---------|------|---------|
| **类型检查** | 确保 JavaScript 传入值类型匹配 | N-API 自动执行 |
| **空值检查** | 可选参数可以为 null | 使用 `napi_status_get_expected_type` |
| **数组检查** | 确保数组参数类型正确 | 使用 `napi_get_array_length` |
| **回调检查** | 验证回调函数存在且可调用 | 使用 `napi_typeof` |
| **范围检查** | 检查数值范围 | N-API 实现 |
| **字符串检查** | 检查字符串长度和编码 | N-API 实现 |

## N-API 错误处理

### 错误码

| 错误码 | 值 | 说明 |
|---------|------|------|
| `napi_ok` | 0 | 成功 |
| `napi_invalid_arg` | 401 | 无效参数 |
| `napi_object_expected` | 402 | 期望对象 |
| `napi_string_expected` | 403 | 期望字符串 |
| `napi_number_expected` | 404 | 期望数字 |
| `napi_boolean_expected` | 405 | 期望布尔值 |
| `napi_function_expected` | 406 | 期望函数 |
| `napi_generic_failure` | 500 | 通用失败 |

### 错误处理最佳实践

1. **检查返回值**：始终检查 N-API 函数返回的错误码
2. **记录错误**：使用日志系统记录错误信息
3. **资源清理**：发生错误时释放已分配的资源
4. **优雅降级**：无法完成功能时提供降级方案

## N-API 权限管理

### 权限检查

**在 N-API 实现中**：
- ✅ N-API 框架会自动检查应用权限
- ✅ N-API 提供权限检查 API
- ✅ 权限不足时自动返回错误

**权限类型**（证据：`wearable/preinstall-config/install_list_permissions.json`）：
- `ohos.permission.CAMERA` - 相机权限
- `ohos.permission.MICROPHONE` - 麦克风权限
- `ohos.permission.READ_CONTACTS` - 读取联系人权限
- `ohos.permission.WRITE_CONTACTS` - 写入联系人权限
- `ohos.permission.LOCATION` - 位置权限
- `ohos.permission.INTERNET` - 网络权限
- `ohos.permission.GET_INSTALLED_BUNDLE_LIST` - 获取已安装应用列表权限

## N-API 开发指南

### 如何添加 N-API

由于 vendor_hihope 仓库不包含 N-API 实现，添加 N-API 需要：

1. **在 OpenHarmony 主源码树中实现**
   - 路径：`foundation/` 对应子系统
   - 参考现有 N-API 实现（如 ability_napi）

2. **创建 N-API 模块**
   - 实现 `napi_module_register`
   - 使用 `napi_define_properties` 导出接口
   - 实现 C++ 函数

3. **注册模块到构建系统**
   - 在 `BUILD.gn` 中添加 N-API 模块
   - 设置依赖（`deps = ["napi:libace_napi.z.so"]`）

### N-API 示例代码

```cpp
// N-API 模块注册
#include "napi/native_api.h"
#include "napi_api.h"

static napi_value MyFunction(napi_env* env, napi_callback_info info) {
    // 1. 参数校验
    napi_valuetype valuetype = napi_typeof(args[0], env);
    
    // 2. 业务逻辑
    napi_value result;
    
    // 3. 返回结果
    return result;
}

// 模块初始化
extern "C" bool NAPI_ModuleRegister(napi_env* env, napi_value exports) {
    // 导出函数列表
    napi_property_descriptor desc[] = {
        declare_napi_property("myFunction", myFunction, nullptr, nullptr)
    };
    
    // 定义类
    napi_define_properties(env, exports, desc, 1, nullptr);
    
    return true;
}
```

### N-API 编译配置

**GN 构建配置**（标准产品中的引用）：
```gni
# 引用 N-API 模块
external_deps = [
  "napi:libace_napi.z.so",
  "napi:libarkui_napi.z.so"
]

# 禁用 N-API 模块
# if (disable_module) {
#   deps -= ["napi:libace_napi.z.so"]
# }
```

## 相关文档

由于 vendor_hihope 仓库不包含 N-API 实现，以下文档提供更详细的信息：

| 文档 | 位置 | 说明 |
|------|------|------|
| **OpenHarmony N-API 开发指南** | 官方文档 | N-API 开发规范和最佳实践 |
| **Ability Runtime 文档** | foundation/ability/ability_runtime | Ability 框架 N-API 文档 |
| **Camera 框架文档** | foundation/multimedia/camera_framework | 相机 N-API 文档 |
| **Audio 框架文档** | foundation/multimedia/audio_framework | 音频 N-API 文档 |
| **Graphics 框架文档** | foundation/graphic/graphic_2d | 图形 N-API 文档 |
| **N-API 参考** | OpenHarmony 官方文档 | 完整的 N-API API 参考 |

## 相关跳转

- [返回 Wiki 首页](SUMMARY.md)
- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
