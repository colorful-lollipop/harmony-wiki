# 编译产物

> 本文档描述 Form Fwk 的编译产物清单、安装路径与运行时加载关系

## 产物总览

| 产物 | 类型 | 路径 | 说明 |
|------|------|------|------|
| libfms.so | System Ability | system/lib64/libfms.so | Form Manager Service 主库 |
| libform_manager.so | Inner API | system/lib64/libform_manager.so | Form 管理器接口 |
| libfmskit_native.so | Native SDK | system/lib64/libfmskit_native.so | Native SDK 库 |
| libfmskit_provider_client.so | Native SDK | system/lib64/libfmskit_provider_client.so | Provider Client |
| Form_Render_Service.hap | HAP | system/app/FormRenderService/ | 渲染服务 |
| *.so (N-API) | N-API 模块 | system/lib64/module/ | 各 N-API 模块 |

## System Ability 产物

### libfms.so

- **路径**: `out/{device}/system/lib64/libfms.so`
- **类型**: System Ability (shlib_type = "sa")
- **SA ID**: 403 (form_fwk脚本**: libfms.map
)
- **版本- **功能**: Form Manager Service 主服务

**依赖模块**:
```
libfms.so
├── libform_manager.so
├── libability_base.so
├── libbundle_framework.so
├── libipc.so
├── libsafwk.so
├── libsamgr.so
├── libhilog.so
└── ... (40+ 依赖)
```

### SA Profile

- **静态 SA**: `sa_profile/403.json`
- **动态 SA**: `sa_profile/403_dynamic.json`
- **控制开关**: `form_fwk_dynamic_support`

## Native SDK 产物

### libfmskit_native.so

- **路径**: `out/{device}/system/lib64/libfmskit_native.so`
- **导出接口**: `interfaces/kits/native/include/`

**头文件**:
```c
// interfaces/kits/native/include/
├── form_mgr.h
├── form_callback_interface.h
└── form_host_client.h
```

### libfmskit_provider_client.so

- **路径**: `out/{device}/system/lib64/libfmskit_provider_client.so`
- **导出接口**: `interfaces/kits/native/include/`

**头文件**:
```c
interfaces/kits/native/include/
└── form_provider_client.h
```

## N-API 模块产物

### 产物列表

| 模块 | 产物 | 安装路径 |
|------|------|----------|
| formbindingdata | libformbindingdata.so | system/lib64/module/app/form/ |
| formbindingdata_napi | libformbindingdata_napi.so | system/lib64/module/application/ |
| forminfo | libforminfo.so | system/lib64/module/app/form/ |
| forminfo_napi | libforminfo_napi.so | system/lib64/module/application/ |
| formhost | libformhost.so | system/lib64/module/app/form/ |
| formhost_napi | libformhost_napi.so | system/lib64/module/application/ |
| formobserver | libformobserver.so | system/lib64/module/app/form/ |
| formprovider | libformprovider.so | system/lib64/module/app/form/ |
| formprovider_napi | libformprovider_napi.so | system/lib64/module/application/ |
| formagent | libformagent.so | system/lib64/module/app/form/ |
| formerror_napi | libformerror_napi.so | system/lib64/module/application/ |
| formutil_napi | libformutil_napi.so | system/lib64/ |

### Stage 模型模块

| 模块 | 产物 | 安装路径 |
|------|------|----------|
| formextensionability | libformextensionability.so | system/lib64/module/app/form/ |
| formextensioncontext_napi | libformextensioncontext_napi.so | system/lib64/module/application/ |
| formeditextensionability_napi | libformeditextensionability_napi.so | system/lib64/module/app/form/ |
| formeditextensioncontext_napi | libformeditextensioncontext_napi.so | system/lib64/module/application/ |
| liveformextensionability_napi | libliveformextensionability_napi.so | system/lib64/module/app/form/ |
| liveformextensioncontext_napi | libliveformextensioncontext_napi.so | system/lib64/module/application/ |

## HAP 产物

### FormRenderService.hap

- **路径**: `out/{device}/system/app/FormRenderService/Form_Render_Service.hap`
- **功能**: Form Render Service 系统应用

**结构**:
```
Form_Render_Service.hap
├── config.json
├── lib/
│   ├── libformrender.so
│   ├── libformrender_service.so
│   └── libfms.so (依赖)
├── resources/
└── js/
```

## ETS/ANI 产物

### ANI 库

| 模块 | 产物 | 安装路径 |
|------|------|----------|
| form_agent_ani | libform_agent_ani.so | system/lib64/ |
| formHost_ani | libformHost_ani.so | system/lib64/ |
| formProvider_ani | libformProvider_ani.so | system/lib64/ |
| formBindingData_ani | libformBindingData_ani.so | system/lib64/ |
| formobserver_ani | libformobserver_ani.so | system/lib64/ |

### ABC 文件

| 模块 | 产物 | 安装路径 |
|------|------|----------|
| formAgent | framework/formagent.abc | system/ |
| formHost | framework/formHost.abc | system/ |
| formProvider | framework/formProvider.abc | system/ |
| formBindingData | framework/form_binding_data.abc | system/ |
| formInfo | framework/formInfo.abc | system/ |
| formError | framework/formError.abc | system/ |
| formObserver | framework/form_observer_abc.abc | system/ |

## C/JSI 产物

### CJ FFI 库

| 模块 | 产物 |
|------|------|
| cj_formBindingData_ffi | libcj_formBindingData_ffi.so |
| cj_formProvider_ffi | libcj_formProvider_ffi.so |

## 运行时加载关系

```
系统启动
    │
    ▼
FormMgrService (SA 403)
    │
    ├── Load libfms.so
    │   ├── Load libform_manager.so
    │   │   ├── Load IDL 生成代码
    │   │   └── Load libhilog.so
    │   │
    │   └── Load libipc.so
    │
    └── Register SA
            │
            ▼
    FormRenderService (HAP)
            │
            ├── Load libformrender.so
            │   └── Load libform_manager.so
            │
            └── Load libformrender_service.so
                    │
                    └── Load libfms.so

应用进程
    │
    ▼
    Load N-API 模块
            │
            ├── Load libformprovider.so
            │   └── Load libfmskit_native.so
            │       └── Load libform_manager.so
            │
            ├── Load libformhost.so
            │   └── Load libfmskit_native.so
            │
            └── Load libformagent.so
                └── Load libfmskit_native.so
```

## 配置产物

### form_config.xml

- **路径**: `out/{device}/system/etc/form/form_config.xml`
- **功能**: Form 框架配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<form_config>
    <!-- Form 配置项 -->
</form_config>
```
