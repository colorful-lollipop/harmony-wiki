# 02_Architecture - 架构设计

## 整体架构

**advanced_ui_component** 采用分层架构设计，从上到下依次为：

1. **ArkTS 组件层** - UI 定义与声明式构建
2. **ABC 字节码层** - 编译产物与运行时代码
3. **N-API 绑定层** - 模块注册与系统桥接
4. **系统 API 层** - OpenHarmony 基础能力

```
┌──────────────────────────────────────────────────────────────────┐
│                         应用层 (ArkTS)                            │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │   Navigation      │  │     Search       │  │    Tabs      │  │
│  │   Component      │  │   Component      │  │  Component   │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │    Web Component │  │   Launch Comp    │  │  Dialog Comp │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
└───────────────────────────────┬────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                      ABC 字节码层                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │   ArkTS 源码 ──► es2abc ──► .abc 字节码 ──► objcopy ──►   │ │
│  │                     ↑                                       │ │
│  │              编译器: es2abc                                │ │
│  │              工具: objcopy (生成 C 数组)                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└───────────────────────────────┬────────────────────────────────┘
                                │ 嵌入到 .so
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                     N-API 绑定层 (.cpp)                           │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  1. 模块注册 (napi_module_register)                        │  │
│  │  2. ABC 导出 (GetABCCode 函数)                             │  │
│  │  3. JS API 实现 (checkUrl, silentInstall, 等)              │  │
│  │  4. 参数校验 (napi_typeof, napi_get_value_string_utf8)      │  │
│  │  5. 异步回调 (uv_work_t, napi_ref)                         │  │
│  └────────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬────────────────────────────────┘
                                │ N-API 调用
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                       系统 API 层                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Ability   │  │  Bundle     │  │      Window Manager      │  │
│  │    Kit     │  │   Manager   │  │                         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  API Policy│  │     HSP     │  │        IPC              │  │
│  │   Adapter  │  │  Installer  │  │                         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

**证据来源**: `BUILD.gn` 文件分析, N-API 源码分析

---

## 模块结构

### 组件目录模式

```
component_name/
├── interfaces/          # N-API 绑定与类型定义
│   ├── *.cpp           # N-API 模块实现
│   ├── *.h             # 头文件 (可选)
│   ├── *.js            # ArkTS 入口 (编译用)
│   ├── include/        # C++ 头文件 (可选)
│   └── BUILD.gn        # 构建配置
├── source/             # ArkTS 组件实现
│   ├── *.ets           # ArkTS UI 定义
│   └── main/           # 组件逻辑 (可选)
└── BUILD.gn            # 组件根构建配置
```

### 组件分类

| 分类 | 组件 | 特点 |
|------|------|------|
| **原子化服务 UI** | Navigation, Search, Tabs, Web | 原子化服务场景专用 |
| **启动组件** | FullScreen, HalfScreen, InnerFullScreen | 应用启动体验 |
| **工具组件** | NavPushPathHelper, CustomAppBar | 导航与应用栏 |

---

## N-API 模块注册机制

### 通用注册流程

**证据来源**: `atomicservicenavigation/interfaces/atomicservicenavigation.cpp:36-51`

```cpp
// 1. 定义 N-API 模块
static napi_module ModuleNameModule = {
    .nm_version = 1,                    // API 版本
    .nm_flags = 0,                      // 标志位
    .nm_filename = nullptr,              // 源文件名 (可选)
    .nm_modname = "atomicservice.ComponentName",  // 模块名
    .nm_register_func = Init,            // 注册函数 (可选)
    .nm_priv = ((void*)0),              // 私有数据
    .reserved = { 0 },                  // 保留字段
};

// 2. 导出 ABC 字节码获取函数
extern "C" __attribute__((visibility("default")))
void NAPI_atomicservice_ComponentName_GetABCCode(const char **buf, int *buflen)
{
    if (buf != nullptr) {
        *buf = _binary_componentname_abc_start;
    }
    if (buflen != nullptr) {
        *buflen = _binary_componentname_abc_end - _binary_componentname_abc_start;
    }
}

// 3. 自动注册模块
extern "C" __attribute__((constructor)) void ComponentNameRegisterModule(void)
{
    napi_module_register(&ModuleNameModule);
}
```

### 模块命名规范

| 组件 | N-API 模块名 | ABC 符号名 |
|------|--------------|------------|
| Navigation | `atomicservice.AtomicServiceNavigation` | `NAPI_atomicservice_AtomicServiceNavigation_GetABCCode` |
| Search | `atomicservice.AtomicServiceSearch` | `NAPI_atomicservice_AtomicServiceSearch_GetABCCode` |
| Tabs | `atomicservice.AtomicServiceTabs` | `NAPI_atomicservice_AtomicServiceTabs_GetABCCode` |
| Web | `atomicservice.AtomicServiceWeb` | `NAPI_atomicservice_AtomicServiceWeb_GetABCCode` |
| NavPushPathHelper | `atomicservice.NavPushPathHelper` | `NAPI_atomicservice_NavPushPathHelper_GetABCCode` |

---

## ABC 字节码构建流程

**证据来源**: `atomicservicenavigation/interfaces/BUILD.gn:18-31`

```
┌─────────────────────────────────────────────────────────────────┐
│                    ABC 字节码构建流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ArkTS 源码 (.ets)                                              │
│        │                                                       │
│        ▼                                                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  es2abc 编译器                                            │   │
│  │  - 输入: .ets 文件                                       │   │
│  │  - 输出: .abc 字节码                                     │   │
│  │  - 工具: //build/config/components/ets_frontend/es2abc    │   │
│  └─────────────────────────────────────────────────────────┘   │
│        │                                                       │
│        ▼                                                       │
│  .abc 字节码文件                                                │
│        │                                                       │
│        ├──► [预览模式] objcopy ──► .c 源码 ──► 编译 ──► .o    │
│        │                                                       │
│        └──► [运行模式] objcopy ──► C 数组 ──► 链接 ──► .so   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 构建脚本配置

```gn
es2abc_gen_abc("gen_componentname_abc") {
  src_js = "componentname.js"              # ArkTS 入口
  dst_file = target_out_dir + "/componentname.abc"
  extra_args = [ "--module" ]              # 模块模式编译
}

gen_js_obj("componentname_abc") {
  input = get_label_info(":gen_componentname_abc", "target_out_dir") + "/componentname.abc"
  dep = ":gen_componentname_abc"
}
```

---

## 线程模型

### 同步 API

大多数组件只导出 ABC 字节码，不涉及线程操作：

```cpp
// 纯 ABC 导出模块 - 无线程操作
static napi_module Module = {
    .nm_register_func = nullptr,  // 无 JS API
    // ...
};
```

### 异步 API

NavPushPathHelper 使用 UV 工作队列实现异步操作：

**证据来源**: `navpushpathhelper/include/hsp_silentinstall_napi.h:52-54`

```cpp
// 异步工作回调
static void SendSuccessBackWork(uv_work_t *work, int statusIn);
static void SendFailBackWork(uv_work_t *work, int statusIn);

// 回调数据管理
struct CallbackData {
    napi_env env = nullptr;
    int32_t errCode = 0;
    std::string errorMessage;
    napi_ref successCallback = nullptr;
    napi_ref failCallback = nullptr;
};
```

```
┌─────────────────────────────────────────────────────────────────┐
│                      异步调用时序                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JS 调用 ──► N-API ──► uv_queue_work ──► 异步执行 ──► 回调     │
│                │             │              │            │      │
│                │             │              │            │      │
│                ▼             ▼              ▼            ▼      │
│           参数校验    工作队列提交     后台任务    回调通知     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 依赖方向

### 组件依赖关系

```
                    ┌────────────────────┐
                    │  NavPushPathHelper │
                    │  (依赖最多)        │
                    └─────────┬──────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   BundleMgr     │  │   AbilityKit    │  │    IPC/Samgr    │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 各组件 external_deps 依赖

| 组件 | 核心依赖 |
|------|----------|
| 通用组件 | `hilog:libhilog`, `napi:ace_napi` |
| NavPushPathHelper | 上述 + `ability_runtime`, `bundle_framework`, `ipc`, `samgr_proxy` |
| AtomServiceMenuBar | 上述 + `ace_engine:ace_ndk`, `ace_engine:ace_uicontent` |
| AtomServiceWeb | 上述 + 无额外依赖 |

**证据来源**: 各组件 `BUILD.gn` 文件

---

## 稳定性标注

### 稳定接口

| 接口 | 稳定性 | 证据 |
|------|--------|------|
| N-API 模块注册 | 稳定 | `napi_module_register` 为系统 API |
| ABC 字节码导出 | 稳定 | `GetABCCode` 函数符号固定 |
| ArkTS 组件 API | 稳定 | 遵循 ArkUI 组件规范 |

### 需要注意的接口

| 接口 | 说明 | 建议 |
|------|------|------|
| `checkUrl` | URL 策略校验 | 依赖系统 API 策略库 |
| `silentInstall` | HSP 静默安装 | 需要特定权限 |
| `dlopen` 路径 | 动态库加载路径 | 不可修改 |

---

## 附录: 组件调用链示例

### AtomServiceWeb checkUrl 调用链

```
┌─────────────────────────────────────────────────────────────────┐
│                    checkUrl 调用链                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JS: atomicservice.AtomicServiceWeb.checkUrl(...)               │
│       │                                                        │
│       ▼                                                        │
│  N-API: CheckUrl(env, info)                                     │
│       │  - 参数解析: bundleName, domainType, url                │
│       │  - 类型校验: napi_typeof                                │
│       │  - 字符串获取: napi_get_value_string_utf8               │
│       ▼                                                        │
│  ApiPolicyAdapter::CheckUrl()                                    │
│       │  - dlopen("/system/lib64/platformsdk/...")              │
│       │  - dlsym("CheckUrl")                                    │
│       ▼                                                        │
│  系统 API: libapipolicy_client.z.so                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**证据来源**: `atomicserviceweb/interfaces/atomicserviceweb.cpp:27-71`