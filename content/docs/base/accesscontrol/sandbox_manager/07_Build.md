# 构建与产物 (Build)

> Sandbox Manager GN 构建配置、编译产物与 Feature 开关详解

---

## 7.1 构建系统概述

| 属性 | 值 |
|-----|-----|
| **构建系统** | GN (Generate Ninja) |
| **构建入口** | `//base/accesscontrol/sandbox_manager/BUILD.gn` |
| **构建命令** | `./build.sh --product-name {product} --build-target sandbox_manager` |
| **组件路径** | `//base/accesscontrol/sandbox_manager` |

---

## 7.2 GN Targets 清单

### 主构建目标

| Target | 类型 | 路径 | 职责 |
|--------|------|------|------|
| `sandbox_manager_service` | executable | `services/sandbox_manager:sandbox_manager_service` | 服务可执行文件 |
| `sandbox_manager_sa_profile_standard` | sa_profile | `services/sandbox_manager/main/sa_profile:sandbox_manager_sa_profile_standard` | SA 配置文件 |
| `libsandbox_manager_sdk` | static/shared library | `frameworks/inner_api/sandbox_manager:libsandbox_manager_sdk` | SDK 静态/动态库 |

**证据来源**：`bundle.json:52-70`

```json
{
  "build": {
    "group_type": {
      "service_group": [
        "//base/accesscontrol/sandbox_manager/services/sandbox_manager:sandbox_manager_service",
        "//base/accesscontrol/sandbox_manager/services/sandbox_manager/main/sa_profile:sandbox_manager_sa_profile_standard"
      ]
    },
    "inner_kits": [
      {
        "name": "//base/accesscontrol/sandbox_manager/frameworks/inner_api/sandbox_manager:libsandbox_manager_sdk",
        "header": {
          "header_base": "//base/accesscontrol/sandbox_manager/interfaces/inner_api/sandbox_manager/include",
          "header_files": [
            "sandbox_manager_kit.h",
            "sandbox_manager_err_code.h",
            "policy_info.h"
          ]
        }
      }
    ]
  }
}
```

### 测试构建目标

| Target | 类型 | 路径 | 描述 |
|--------|------|------|------|
| `sandbox_manager_build_module_test` | test group | `BUILD.gn:17` | 单元测试组 |
| `sandbox_manager_build_fuzz_test` | test group | `BUILD.gn:29` | Fuzz 测试组 |
| `unittest` | unittest | `frameworks/inner_api/sandbox_manager/test:unittest` | 框架层测试 |
| `unittest` | unittest | `services/sandbox_manager/test:unittest` | 服务层测试 |
| `fuzztest` | fuzztest | `test/fuzztest/innerkits/sandbox_manager:fuzztest` | Fuzz 测试 |

**证据来源**：`BUILD.gn:17-38`

```gn
group("sandbox_manager_build_module_test") {
  testonly = true
  deps = []
  if (is_standard_system) {
    deps += [
      "frameworks/inner_api/sandbox_manager/test:unittest",
      "frameworks/test:unittest",
      "services/sandbox_manager/test:unittest",
    ]
  }
}

group("sandbox_manager_build_fuzz_test") {
  testonly = true
  deps = []
  if (is_standard_system) {
    deps += [
      "test/fuzztest/innerkits/sandbox_manager:fuzztest",
      "test/fuzztest/services/sandbox_manager:fuzztest",
    ]
  }
}
```

---

## 7.3 Feature 开关

### 可配置 Feature

| Feature | 默认值 | 类型 | 描述 |
|---------|-------|------|------|
| `sandbox_manager_feature_coverage` | - | bool | 代码覆盖率支持 |
| `sandbox_manager_process_resident` | `false` | bool | 进程常驻模式 |
| `sandbox_manager_with_dec` | `false` | bool | DEC (Data Execution Prevention) 支持 |
| `sandbox_manager_dec_ext` | `false` | bool | DEC 扩展支持 |

**证据来源**：`sandbox_manager.gni:18-22`

```gni
declare_args() {
  sandbox_manager_process_resident = false
  sandbox_manager_with_dec = false
  sandbox_manager_dec_ext = false
}
```

### Feature 详细说明

#### sandbox_manager_process_resident

**描述**：控制服务是否为常驻进程。

**影响**：
- `true`：服务启动后一直运行，不自动退出
- `false`：服务空闲一段时间后自动卸载（默认 3 分钟）

**配置位置**：
```cpp
// 证据来源：services/sandbox_manager/main/cpp/src/service/sandbox_manager_service.cpp

#ifdef NOT_RESIDENT
    // 非常驻模式：空闲后卸载
    SandboxManagerService::DelayUnloadService();
#endif
```

#### sandbox_manager_with_dec

**描述**：是否启用 DEC (Data Execution Prevention) 支持。

**影响**：
- `true`：启用 DEC 相关功能
- `false`：禁用 DEC 功能

**证据来源**：`bundle.json:19`

```json
"features": [
  "sandbox_manager_feature_coverage",
  "sandbox_manager_process_resident",
  "sandbox_manager_with_dec",
  "sandbox_manager_dec_ext"
]
```

#### sandbox_manager_dec_ext

**描述**：是否启用 DEC 扩展支持。

**依赖**：
- 依赖于 `sandbox_manager_with_dec`

---

## 7.4 编译产物

### 产物清单

| 产物类型 | 路径模式 | 说明 |
|---------|---------|------|
| **SDK 静态库** | `out/.../libsandbox_manager_sdk.a` | 供其他模块静态链接 |
| **SDK 动态库** | `out/.../libsandbox_manager_sdk.z.so` | 供其他模块动态链接 |
| **服务可执行** | `out/.../sandbox_manager_service` | SystemAbility 可执行文件 |
| **SA 配置文件** | `out/.../sandbox_manager_sa_profile.xml` | SA 注册配置 |

### 输出目录结构

```
out/
└── product_name/
    └── ...
        └── system/
            └── bin/
                └── sandbox_manager_service    # 服务可执行文件
            └── etc/
                └── sandbox_manager/
                    └── sandbox_manager_sa_profile.xml
            └── lib/
                ├── libsandbox_manager_sdk.z.so  # 动态库
                └── libsandbox_manager_sdk.a     # 静态库
```

---

## 7.5 构建配置示例

### 标准构建

```bash
# 构建 sandbox_manager 模块
./build.sh --product-name rk3568 --build-target sandbox_manager

# 构建所有测试
./build.sh --product-name rk3568 --build-target sandbox_manager_build_module_test
./build.sh --product-name rk3568 --build-target sandbox_manager_build_fuzz_test
```

### 带 Feature 构建

```bash
# 启用 DEC 支持
./build.sh --product-name rk3568 \
    --build-target sandbox_manager \
    --gn-args sandbox_manager_with_dec=true
```

---

## 7.6 依赖配置

### 组件依赖

| 组件 | 用途 |
|-----|------|
| `ability_base` | Ability 框架基础 |
| `access_token` | 访问令牌与权限管理 |
| `appspawn` | 应用生成服务 |
| `bundle_framework` | Bundle 管理框架 |
| `cJSON` | JSON 解析 |
| `c_utils` | C 工具库 |
| `common_event_service` | 公共事件服务 |
| `config_policy` | 配置策略 |
| `data_share` | 数据共享 |
| `eventhandler` | 事件处理 |
| `hilog` | 日志系统 |
| `hisysevent` | HiSysEvent 事件 |
| `ipc` | 进程间通信 |
| `media_library` | 媒体库 |
| `os_account` | 账户管理 |
| `relational_store` | 关系型数据库 |
| `safwk` | System Ability Framework |
| `samgr` | System Ability Manager |

**证据来源**：`bundle.json:29-48`

```json
"deps": {
  "components": [
    "ability_base",
    "access_token",
    "appspawn",
    "bundle_framework",
    "cJSON",
    "c_utils",
    "common_event_service",
    "config_policy",
    "data_share",
    "eventhandler",
    "hilog",
    "hisysevent",
    "ipc",
    "media_library",
    "os_account",
    "relational_store",
    "safwk",
    "samgr"
  ],
  "third_party": []
}
```

---

## 7.7 资源占用

| 资源类型 | 大小限制 |
|---------|---------|
| **ROM** | 10000 KB (10 MB) |
| **RAM** | 5000 KB (5 MB) |

**证据来源**：`bundle.json:23-24`

```json
"rom": "10000KB",
"ram": "5000KB",
```

---

## 7.8 构建问题排查

### 常见问题

#### Q1: 编译报错 "undefined reference"

**问题**：链接错误，找不到依赖库

**解决方案**：
```bash
# 确保依赖组件已构建
./build.sh --product-name {product} --build-target access_token
./build.sh --product-name {product} --build-target relational_store
```

#### Q2: SA 注册失败

**问题**：服务启动时报 SA 注册错误

**解决方案**：
- 检查 `sandbox_manager_sa_profile.xml` 路径是否正确
- 确认 SA ID 与代码中一致

#### Q3: 找不到头文件

**问题**：编译时找不到 `sandbox_manager_kit.h`

**解决方案**：
- 确认 inner_kits 配置正确
- 检查 include path 是否包含 `interfaces/inner_api/sandbox_manager/include`

---

*文档更新时间: 2025-02-07*
