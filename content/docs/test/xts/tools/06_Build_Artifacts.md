# 06_Build_Artifacts.md

## 目的

本文档说明 XTS Tools 仓库的编译产物，包括类型、安装路径与运行时加载关系。

## 适用范围

- 仅涉及 tools/ 仓库构建产物
- 不涉及 acts 测试用例产物（位于 /test/xts/acts）

---

## 产物清单

| 产物类型 | 示例名称 | 生成位置 | 安装路径 | 运行时加载方式 | 证据 |
|---------|---------|---------|----------|----------------|------|
| 静态库 (.a) | libhctest.a | out/xxx/obj/tools/lite/hctest/... | 链接至镜像 | 静态链接 | README.md:345-351 |
| 可执行文件 (.bin) | ActsDemoTest.bin | out/xxx/suites/acts/ | suites/acts/ 目录 | NFS 挂载执行 | README.md:477-483 |
| HAP 包 | xxx.hap | out/xxx/ | HAP 安装目录 | 应用加载 | README.md:645-649 |
| 可执行文件 (工具) | QueryMainStandard | out/xxx/bin/ | bin/ 或系统路径 | 直接调用 | glob 发现 QueryMainStandard.cpp |
| 校验和工具 | checksum | out/xxx/bin/checksum | bin/ 或系统路径 | 直接调用 | `lite/checksum/BUILD.gn:15` |

---

## 安装路径

### Mini 系统

- 静态库
  - 链接至系统镜像，无独立安装路径
  - 证据：README.md:345-351

### Small/Standard 系统（C++）

- 可执行文件
  - 归档至 suites/acts/ 目录
  - 通过 NFS 挂载执行
  - 证据：README.md:477-483

### Standard 系统（JavaScript）

- HAP 包
  - 安装至 OpenHarmony HAP 目录
  - 应用加载执行
  - 证据：README.md:645-649

### 工具链

- 查询工具
  - 安装至 bin/ 或系统路径
  - 证据：`others/query/BUILD.gn`、`lite/others/query/BUILD.gn`

- 校验和工具
  - 安装至 bin/ 或系统路径（条件编译：仅在 liteos_a 或 linux 内核）
  - 证据：`lite/checksum/BUILD.gn:14-22`
  - 证据：glob 发现 checksum/BUILD.gn

---

## 运行时加载关系

### 静态库加载

- Mini 系统
  - 静态库链接至镜像，系统上电时加载
  - 证据：README.md:345-351

### 可执行文件加载

- Small/Standard 系统（C++）
  - 通过 NFS 挂载 suites/acts/ 目录
  - 执行 .bin 文件
  - 证据：README.md:485-510

- 工具链
  - 直接调用可执行文件
  - 证据：QueryMainStandard.cpp

### HAP 加载

- Standard 系统（JavaScript）
  - 应用加载 HAP 包
  - HJSUnit 框架加载测试用例
  - 证据：README.md:511-649

---

## 构建产物与依赖

- 静态库
  - 依赖：Unity/Googletest 框架（开源）
  - 证据：README.md:256、374

- 可执行文件
  - 依赖：静态库 + GN 目标
  - 证据：BUILD.gn deps/public_deps

- HAP 包
  - 依赖：deccjsunit 框架（内部）
  - 证据：README.md:513

---

## 相关跳转

- [00_Overview.md](00_Overview.md)
- [05_GN_Targets.md](05_GN_Targets.md)
- [02_Architecture.md](02_Architecture.md)
