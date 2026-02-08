# 07_Build - 构建与产物

本文档描述 SOC 统一调频部件的 GN 构建配置、编译产物和 Feature 开关。

## 构建概览

| 属性 | 值 |
|------|-----|
| **构建系统** | GN + Ninja |
| **构建入口** | `./build.sh` |
| **构建目标** | `soc_perf` |
| **编译产物** | `.so` 共享库、`.a` 静态库、XML 配置文件 |

## Feature 开关

### soc_perf_device_enable

**定义文件**：`soc_perf.gni:24-26`

```gni
declare_args() {
  soc_perf_device_enable = true
}
```

**类型**：`declare_args`（全局配置）

**默认值**：`true`

**说明**：控制整个 soc_perf 模块的编译启用/禁用

**使用方式**：

```bash
# 禁用 soc_perf 模块
./build.sh --product-name {product} --build-target soc_perf \
  --gn-args soc_perf_device_enable=false
```

## GN Targets 清单

### 根目录 BUILD.gn

| Target | 类型 | 条件 | 依赖 | 说明 |
|--------|------|------|------|------|
| `base_group_soc_perf_all` | group | `soc_perf_device_enable` | `profile:socperf_config` | 基础配置组 |
| `fwk_group_socperf_client_all` | group | `soc_perf_device_enable` | `interfaces/...:socperf_client` | 客户端库组 |
| `service_group_soc_perf_all` | group | `soc_perf_device_enable` | `sa_profile:socperf_sa_profile`, `services:socperf_server` | 服务组 |
| `test_soc_perf_all` | group | `soc_perf_device_enable` + testonly | `fuzztest:...`, `testutil:...`, `unittest:...` | 测试组 |

### services/BUILD.gn

| Target | 类型 | 产物 | 依赖 |
|--------|------|------|------|
| `socperf_server_config` | config | - | - |
| `socperf_server` | ohos_shared_library | `libsocperf_server.so` | 12 个 external_deps |
| `socperf_server_static` | ohos_static_library | `libsocperf_server_static.a` | 12 个 external_deps |

**Source Files**：

```gn
sources = [
    "$ socperf_services/core/src/socperf.cpp",
    "$ socperf_services/core/src/socperf_config.cpp",
    "$ socperf_services/core/src/socperf_thread_wrap.cpp",
    "$ socperf_services/dfx/src/socperf_hitrace_chain.cpp",
    "$ socperf_services/server/src/socperf_server.cpp",
]
```

**External Dependencies**：

| 依赖 | 用途 |
|------|------|
| `ipc:ipc_core` | IPC 通信 |
| `samgr:samgr_proxy` | 服务管理 |
| `safwk:safwk_core` | SA 框架 |
| `hilog:hilog` | 日志输出 |
| `ffrt:libffrt` | 异步任务 |
| `hitrace:hitrace` | 性能追踪 |
| `libxml2:libxml2` | XML 解析 |
| `utils:utils_base` | 基础工具 |
| `cjson:cjson` | JSON 处理 |
| `hisysevent:hisysevent` | 事件上报 |
| `access_token:access_token` | 权限管理 |
| `selinux_adapter:selinux_adapter` | SELinux 适配 |

### interfaces/BUILD.gn

| Target | 类型 | 产物 | 依赖 |
|--------|------|------|------|
| `socperf_client_interface` | idl_gen_interface | IDL 生成代码 | `ISocPerf.idl` |
| `socperf_client_public_config` | config | - | - |
| `socperf_client` | ohos_shared_library | `libsocperf_client.so` | `socperf_client_interface` |
| `socperf_stub` | ohos_source_set | - | `socperf_client_interface` |

### sa_profile/BUILD.gn

| Target | 类型 | 产物 | 说明 |
|--------|------|------|------|
| `socperf_sa_profile` | ohos_sa_profile | SA 配置文件 | `1906.json` |

### profile/BUILD.gn

| Target | 类型 | 产物 | 安装路径 |
|--------|------|------|----------|
| `socperf_resource_config` | ohos_prebuilt_etc | `socperf_resource_config.xml` | `etc/soc_perf/` |
| `socperf_boost_config` | ohos_prebuilt_etc | `socperf_boost_config.xml` | `etc/soc_perf/` |
| `socperf_config` | group | - | 聚合上述 |

## 编译产物

### 核心产物

| 产物文件 | 类型 | 来源 Target | 说明 |
|----------|------|--------------|------|
| `libsocperf_server.so` | 共享库（SA） | `services:socperf_server` | 系统服务主库 |
| `libsocperf_server_static.a` | 静态库 | `services:socperf_server_static` | 静态链接版本 |
| `libsocperf_client.so` | 共享库 | `interfaces/...:socperf_client` | 客户端接口库 |
| `1906.json` | SA 配置 | `sa_profile:socperf_sa_profile` | SA ID 注册 |
| `socperf_resource_config.xml` | 配置文件 | `profile:socperf_resource_config` | 资源定义 |
| `socperf_boost_config.xml` | 配置文件 | `profile:socperf_boost_config` | 提频配置 |

### 安装路径

| 产物 | 目标路径 |
|------|----------|
| `libsocperf_server.so` | `system/lib64/` |
| `libsocperf_client.so` | `system/lib64/` |
| `*.xml` | `system/etc/soc_perf/` |
| `1906.json` | `system/profile/` |

## 构建命令

### 标准构建

```bash
# 编译 32 位 ARM 系统
./build.sh --product-name {product} --ccache --build-target soc_perf

# 编译 64 位 ARM 系统
./build.sh --product-name {product} --ccache --target-cpu arm64 --build-target soc_perf
```

### 带测试构建

```bash
# 编译所有（包括测试）
./build.sh --product-name {product} --build-target soc_perf --build-target test_soc_perf_all
```

### 独立构建

```bash
# 仅构建 soc_perf 服务
./build.sh --product-name {product} --build-target socperf_server

# 仅构建客户端
./build.sh --product-name {product} --build-target socperf_client
```

## 依赖关系图

```
libsocperf_server.so
├── libsocperf_stub.a
│   └── ISocPerf 接口生成代码
├── libhilog.so
├── libhisysevent.so
├── libipc.so
├── libsamgr.so
├── libsafwk.so
├── libffrt.so
├── libhitrace.so
├── libxml2.so
├── libselinux_adapter.so
└── libcjson.so

libsocperf_client.so
├── ISocPerf 接口生成代码
├── libipc.so
└── libhilog.so
```

## 构建配置

### 产物类型标签

| 标签 | 说明 |
|------|------|
| `ohos_shared_library` | 动态共享库 |
| `ohos_static_library` | 静态库 |
| `ohos_prebuilt_etc` | 预编译配置文件 |
| `ohos_sa_profile` | SA 配置文件 |
| `ohos_source_set` | 源码集合 |
| `ohos_unittest` | 单元测试 |
| `ohos_fuzztest` | Fuzz 测试 |

### 构建组

| 组 | 包含 |
|----|------|
| `base_group` | 基础配置（XML 配置文件） |
| `fwk_group` | 框架层（客户端库） |
| `service_group` | 服务层（服务端 SA） |

---

## 相关文档

- 架构设计：[02_Architecture](02_Architecture.md)
- 接口文档：[04_Interface](04_Interface.md)
- 代码地图：[03_CodeMap](03_CodeMap.md)

---

*文档版本：v1.0*
*最后更新：2026-02-07*
