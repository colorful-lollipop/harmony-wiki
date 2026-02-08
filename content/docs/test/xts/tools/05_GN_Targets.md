# 05_GN_Targets.md

## 目的

本文档梳理 XTS Tools 仓库的 GN 目标（targets），包括类型、依赖、产物与编译开关。帮助开发者理解构建系统。

## 适用范围

- 仅涉及 tools/ 仓库内的 BUILD.gn/.gni 文件
- 不涉及上层 acts 测试用例构建

---

## 关键 BUILD.gn/.gni 文件

| 文件路径 | 类型 | 作用 | 证据 |
|---------|------|------|------|
| tools/lite/BUILD.gn | GN 构建文件 | 轻量级系统测试框架顶层构建 | glob 发现 |
| tools/lite/hctest/BUILD.gn | GN 构建文件 | HCTest 框架目标 | glob 发现 |
| tools/lite/hcpptest/BUILD.gn | GN 构建文件 | HCPPTest 框架目标 | glob 发现 |
| tools/lite/checksum/BUILD.gn | GN 构建文件 | 校验和工具目标 | glob 发现 |
| tools/lite/others/query/BUILD.gn | GN 构建文件 | 查询工具目标 | glob 发现 |
| tools/others/query/BUILD.gn | GN 构建文件 | 标准系统查询工具目标 | glob 发现 |
| tools/build/suite.gni | GN 模板 | 测试套件模板 | glob 发现 |

---

## 主要 Targets（按模块分组）

### 测试框架模块

- HCTest 目标（`lite/hctest/BUILD.gn`）
  - target 类型：`static_library`
  - target 名称：`hctest`
  - sources：3 个 C 文件：`unity.c`、`hctest.c`、`hctest_service.c`
  - deps：空（deps = []）
  - public：`unity.h`、`hctest.h`
  - 输出名：`libhctest.a`（推断）
  - 证据：`lite/hctest/BUILD.gn`

- HCPPTest 目标（`lite/hcpptest/BUILD.gn`）
  - target 类型：`static_library`（4 个目标：hcpptest、hcpptest_main、gmock、gmock_main）
  - target 名称：`hcpptest`、`hcpptest_main`、`gmock`、`gmock_main`
  - sources：约40 个 C++ 文件（Google Test/Mock 源文件）
  - deps：`hcpptest_main` → `hcpptest`；`gmock` → `hcpptest`；`gmock_main` → `gmock`、`hcpptest`
  - configs：`hcpptest_config`、`hcpptest_private_config`、`gmock_config`、`gmock_private_config`
  - 输出名：`libhcpptest.a`、`libhcpptest_main.a`、`libgmock.a`、`libgmock_main.a`（推断）
  - 证据：`lite/hcpptest/BUILD.gn`

- HJSUnit 目标
  - HJSUnit 本身不直接由 GN 构建（JS 框架），但 HAP 包由 GN 编译
  - target 类型：`ohos_hap_suite`（模板定义在 `build/suite.gni`）
  - 模板名称：`ohos_hap_suite`、`ohos_js_hap_suite`、`ohos_js_app_suite`
  - 输出名：`.hap`

---

## Target 类型与输出名

| 模块 | Target 类型 | 输出名 | 证据 |
|------|------------|--------|------|
| HCTest | static_library | libhctest.a（推断） | `lite/hctest/BUILD.gn` |
| HCPPTest | static_library | libhcpptest.a、libgmock.a 等（推断） | `lite/hcpptest/BUILD.gn` |
| HJSUnit | ohos_hap_suite（模板） | .hap | `build/suite.gni` |
| 校验和工具 | executable | checksum | `lite/checksum/BUILD.gn` |
| 查询工具 | ohos_executable | queryStandard、querySmall | `others/query/BUILD.gn`、`lite/others/query/BUILD.gn` |

---

## 关键 deps/public_deps

- HCTest
  - deps：无外部依赖（deps = []）
  - public_deps：无（未在 BUILD.gn 中声明）
  - 证据：`lite/hctest/BUILD.gn:36`「deps = []」
  - 说明：HCTest 直接链接 Unity 源码，不通过 deps 引用

- HCPPTest
  - deps：
    - `hcpptest_main` → `hcpptest`（`:hcpptest`）
    - `gmock` → `hcpptest`（`:hcpptest`）
    - `gmock_main` → `gmock`, `hcpptest`（`:gmock`, `:hcpptest`）
  - public_deps：
    - `hcpptest_main`: [":hcpptest"]
    - `gmock_main`: [":gmock", ":hcpptest"]
  - 证据：`lite/hcpptest/BUILD.gn:86, 134, 139-142`

- HJSUnit
  - deps：deccjsunit 框架（内部）
  - public_deps：不适用（JS 框架不由 GN 直接管理）
  - 证据：README.md:513

---

## 关键 defines 与 configs

- defines
  - HCTest：`UNITY_INCLUDE_CONFIG_H`
  - HCPPTest：`GTEST_HAS_CLONE=0`
  - 证据：`lite/hctest/BUILD.gn:34`、`lite/hcpptest/BUILD.gn:81`

- configs
  - HCTest：条件编译 `if (board_toolchain_type != "iccarm") { cflags = ["-Wno-error"] }`
  - HCPPTest：
    - `hcpptest_config`：include_dirs、ldflags、cflags_cc、cflags
    - `hcpptest_private_config`：include_dirs、ldflags、cflags_cc、cflags
    - `gmock_config`：include_dirs、cflags_cc
    - `gmock_private_config`：include_dirs
  - 证据：`lite/hctest/BUILD.gn:38-40`、`lite/hcpptest/BUILD.gn:14-31, 89-105`

---

## Target ↔ 最终产物映射

| Target 名称 | 输出类型 | 预期输出路径 | 安装路径 | 运行时加载方式 | 证据 |
|------------|---------|--------------|----------|----------------|------|
| hctest | static_library | out/xxx/obj/tools/lite/hctest/.../libhctest.a | 链接至镜像 | 静态链接 | `lite/hctest/BUILD.gn`；README.md:345-351 |
| hcpptest | static_library | out/xxx/obj/tools/lite/hcpptest/.../libhcpptest.a | 链接至 .bin | 静态链接 | `lite/hcpptest/BUILD.gn` |
| checksum | executable | out/xxx/bin/checksum | bin/ 或系统路径 | 直接调用 | `lite/checksum/BUILD.gn` |
| queryStandard | ohos_executable | out/xxx/.../queryStandard | 系统路径 | 直接调用 | `others/query/BUILD.gn` |
| querySmall | executable | out/xxx/suites/.../querySmall.bin | suites/acts/ 目录 | NFS 挂载执行 | `lite/others/query/BUILD.gn` |
| ohos_hap_suite | ohos_hap | out/xxx/.../xxx.hap | HAP 安装目录 | 应用加载 | `build/suite.gni`；README.md:645-649 |

---

## 编译开关

- 测试类型开关：Function/Performance/Power/Reliability/Security 等
  - 证据：README.md:147-211

- 测试粒度开关：SmallTest/MediumTest/LargeTest
  - 证据：README.md:112-144

- 测试级别开关：Level0-Level4
  - 证据：README.md:63-109

**说明**：编译开关主要用于测试用例级别控制，由测试框架宏定义实现，非 GN 目标层配置。

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [01_Directory_Structure.md](01_Directory_Structure.md)
- [06_Build_Artifacts.md](06_Build_Artifacts.md)
