# 构建配置

## GN 构建系统

广告子系统使用 GN (Generate Ninja) 构建系统，产物为 OpenHarmony HAP/HSP 格式。

## BUILD.gn 文件清单

| # | 文件路径 |
|---|----------|
| 1 | `BUILD.gn` (根目录) |
| 2 | `common/BUILD.gn` |
| 3 | `frameworks/js/napi/ads/BUILD.gn` |
| 4 | `frameworks/js/napi/adcomponent/BUILD.gn` |
| 5 | `frameworks/js/napi/autoadcomponent/BUILD.gn` |
| 6 | `frameworks/js/napi/adsservice_extension_ability/BUILD.gn` |
| 7 | `frameworks/js/napi/adsservice_extension_context/BUILD.gn` |
| 8 | `frameworks/js/napi/extension/BUILD.gn` |
| 9 | `frameworks/cj/ffi/ads/BUILD.gn` |

## Targets 清单

### 根目录 Targets

**advertising_native_packages** (group)
- 聚合所有 native packages 的入口
- 依赖: 8 个子模块

### common 模块

**advertising_common** (ohos_source_set)
- 类型: 静态库
- 产物: `libadvertising_common.a`
- CFI: 启用
- 源码:
  - `ipc/src/ad_load_callback_stub.cpp`
  - `ipc/src/ad_load_proxy.cpp`
  - `ipc/src/ad_request_body_stub.cpp`
  - `utils/src/ad_common_util.cpp`
  - `utils/src/ad_json_util.cpp`
- 依赖:
  - `ability_base:want`
  - `bundle_framework:*`
  - `cJSON:cjson`
  - `hilog:libhilog`
  - `ipc:ipc_core`
  - `safwk:system_ability_fwk`
  - `samgr:samgr_proxy`

### ads N-API 模块

**advertising** (ohos_shared_library)
- 类型: 共享库
- 产物: `libadvertising.so`
- 安装目录: `module/`
- 源码:
  - `src/ad_init.cpp`
  - `src/ad_load_napi_common.cpp`
  - `src/ad_load_service.cpp`
  - `src/ad_napi_common_error.cpp`
  - `src/advertising.cpp`
- 依赖:
  - `:advertising_abc` (JS 编译产物)
  - `:advertising_js` (JS 编译产物)
  - `:advertising_common`
  - `ability_runtime:*`
  - `ace_engine:ace_uicontent`
  - `bundle_framework:*`
  - `cJSON:cjson`
  - `hilog:libhilog`
  - `ipc:ipc_core`
  - `napi:ace_napi`
  - `safwk:system_ability_fwk`
  - `samgr:samgr_proxy`

**gen_advertising_abc** (es2abc_gen_abc)
- 类型: JS 编译器
- 输入: `src/advertising.js`
- 输出: `advertising.abc` (Ark Bytecode)

**advertising_js** (gen_js_obj)
- 类型: JS 编译器
- 输入: `src/advertising.js`
- 输出: `advertising.o`

**advertising_abc** (gen_js_obj)
- 类型: JS 编译器
- 输入: `advertising.abc`
- 输出: `advertising_abc.o`

**ad_service_config_json** (ohos_prebuilt_etc)
- 类型: 配置文件
- 安装目录: `advertising/ads_framework/`

### adcomponent 模块

**adcomponent** (ohos_shared_library)
- 产物: `libadcomponent.so`
- 安装目录: `module/advertising/`

### autoadcomponent 模块

**autoadcomponent** (ohos_shared_library)
- 产物: `libautoadcomponent.so`
- 安装目录: `module/advertising/`

### adsservice_extension_ability 模块

**adsserviceextensionability_napi** (ohos_shared_library)
- 产物: `libadsserviceextensionability_napi.so`
- 安装目录: `module/advertising/`

### adsservice_extension_context 模块

**adsserviceextensioncontext_napi** (ohos_shared_library)
- 产物: `libadsserviceextensioncontext_napi.so`
- 安装目录: `module/advertising/`

### extension 模块

**libadsservice_extension** (ohos_shared_library)
- 产物: `libadsservice_extension.so`
- 源码:
  - `src/adsservice_extension.cpp`
  - `src/adsservice_extension_context.cpp`
  - `src/js_adsservice_extension.cpp`
  - `src/js_adsservice_extension_context.cpp`

**adsservice_extension_module** (ohos_shared_library)
- 产物: `libadsservice_extension_module.so`
- 安装目录: `extensionability/`

### CJ FFI 模块

**cj_advertising_ffi** (ohos_shared_library)
- 产物: `libcj_advertising_ffi.so`
- Inner API Tag: `platformsdk`
- 源码:
  - `src/cj_advertising_common.cpp`
  - `src/cj_advertising_ffi.cpp`
  - `src/cj_advertising_impl.cpp`
  - `src/cj_advertising_load_service.cpp`

## 产物清单

### 共享库 (.so)

| 产物 | 路径 | 说明 |
|------|------|------|
| `libadvertising.so` | `module/` | 主广告 N-API |
| `libadcomponent.so` | `module/advertising/` | 广告组件 |
| `libautoadcomponent.so` | `module/advertising/` | 自动广告组件 |
| `libadsserviceextensionability_napi.so` | `module/advertising/` | SA 扩展能力 |
| `libadsserviceextensioncontext_napi.so` | `module/advertising/` | SA 上下文 |
| `libadsservice_extension.so` | (默认) | 扩展核心库 |
| `libadsservice_extension_module.so` | `extensionability/` | 扩展模块 |
| `libcj_advertising_ffi.so` | (默认) | Cangjie FFI |

### 静态库 (.a)

| 产物 | 路径 | 说明 |
|------|------|------|
| `libadvertising_common.a` | `common/` | 公共静态库 |

### 配置文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `ad_service_config.json` | `advertising/ads_framework/` | 服务配置 |

## 安全编译选项

所有共享库启用以下安全特性：

| 选项 | 说明 |
|------|------|
| `pac_ret` | PAC-RET 分支保护 |
| `cfi` | 控制流完整性 |
| `cfi_cross_dso` | 跨 DSO CFI |
| `boundary_sanitize` | 边界检查 |
| `integer_overflow` | 整数溢出检查 |
| `ubsan` | 未定义行为检查 |

## 构建命令

```bash
# 完整构建
hb set
hb build

# 仅构建 advertising
hb build advertising

# 单独编译
gn gen out/advertising
ninja -C out/advertising advertising_native_packages
```

## 相关文档

- [架构说明](Architecture.md) - 模块依赖关系
- [N-API 参考](NAPI_Reference.md) - API 调用链
- [安全评审](Security_Review.md) - 编译安全考量
