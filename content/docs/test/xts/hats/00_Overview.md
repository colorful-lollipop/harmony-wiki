# 项目概览

## 项目定位

HATS（Hardware Abstract Test Suite，硬件抽象测试套件）是 OpenHarmony XTS（X Test Suite）生态认证测试套件的重要组成部分。

### 核心目标

1. **HAL 兼容性验证**：帮助终端设备厂商尽早发现 HAL 层软件与 OpenHarmony 的不兼容性
2. **全生命周期保障**：确保软件在整个开发过程中满足 OpenHarmony 的兼容性要求
3. **硬件抽象标准化**：提供标准化的硬件接口测试框架

### 项目边界

| 范围 | 说明 |
|------|------|
| **在范围内** | HAL/HDI 层接口测试、驱动兼容性测试、系统能力测试 |
| **不在范围内** | 应用层测试、N-API JS 接口测试、单元测试（使用 xts_acts） |

> **证据**：`/Volumes/lexar/code/d/work/oh/test/xts/hats/README.md:4-8` - "XTS子系统是OpenHarmony生态认证测试套件的集合，当前包括hats（hardware abstract test suite), HAL兼容性测试套件"

---

## 核心能力

### 支持的系统类型

| 系统类型 | 内存要求 | 处理器 | 开发语言 | 测试框架 |
|----------|----------|--------|----------|----------|
| **Mini 系统** | ≥128 KiB | ARM Cortex-M, RISC-V 32-bit | C | hctest |
| **Small 系统** | ≥1 MiB | ARM Cortex-A | C++ | hcpptest |
| **Standard 系统** | ≥128 MiB | ARM Cortex-A | C++/JS | HJSUnit, hcpptest |

### 用例级别（Test Level）

| 级别 | 名称 | 说明 |
|------|------|------|
| Level0 | 冒烟 | 验证关键功能基本可用 |
| Level1 | 基本 | 验证常见输入下的功能 |
| Level2 | 重要 | 验证常规输入和异常情况 |
| Level3 | 一般 | 验证全部功能和极端输入 |
| Level4 | 生僻 | 验证极端异常条件 |

### 用例粒度（Test Size）

| 粒度 | 被测对象 | 环境要求 |
|------|----------|----------|
| LargeTest | 业务功能/全场景特性 | 真实设备环境 |
| MediumTest | 模块/子系统集成 | 真实单设备 |
| SmallTest | 模块/类/函数 | 开发者环境（可 MOCK） |

---

## 运行环境

### 硬件要求

| 系统类型 | 最低内存 | 典型设备 |
|----------|----------|----------|
| Mini | 128 KiB | 连接模组、传感器、穿戴设备 |
| Small | 1 MiB | IP Camera、电子猫眼、路由器 |
| Standard | 128 MiB | 高端显示屏、智能电视 |

### 软件依赖

| 依赖项 | 版本 | 说明 |
|--------|------|------|
| OpenHarmony | 4.0+ | 目标测试系统 |
| GoogleTest | Latest | C++ 测试框架 |
| hctest/hcpptest | Latest | OpenHarmony 测试框架 |

---

## 目录结构

```
/test/xts/hats/
├── ai/                    # AI/NNRT 测试
│   └── nnrt/             # 神经网络运行时测试
├── distributedhardware/   # 分布式硬件测试
├── hdf/                  # 硬件驱动框架测试（20+ 模块）
├── kernel/               # 内核/系统调用测试
├── powermgr/             # 电源管理测试
├── startup/              # 启动测试
├── telephony/            # 电信/RIL 测试
├── testtools/            # 测试工具配置
├── useriam/              # 用户身份认证测试
├── BUILD.gn             # 主构建配置
├── bundle.json          # 组件配置
└── README.md            # 项目说明
```

### 子系统统计

| 子系统 | BUILD.gn 文件数 | C/C++ 文件数 | 主要测试对象 |
|--------|-----------------|--------------|--------------|
| hdf | 154 | 393 | 硬件驱动接口 |
| kernel | 307 | 172 | 系统调用 |
| useriam | 10 | 23 | 用户认证 |
| ai | 9 | 16 | 神经网络运行时 |
| powermgr | 13 | 9 | 电源管理 |
| telephony | 4 | 10 | 无线接口层 |
| distributedhardware | 3 | 6 | 分布式相机 |
| startup | 3 | 2 | 分区启动 |
| testtools | 1 | 0 | 工具配置 |
| **总计** | **505** | **631** | - |

---

## 关键概念

### HDI（Hardware Driver Interface）

HDI 是 OpenHarmony 硬件驱动的标准接口层，位于 HAL 之上，为上层服务提供统一的硬件访问接口。

```
应用层 → 服务层 → HDI → HAL → 硬件
         ↑                    ↑
      HATS 测试覆盖范围
```

### 测试模式

HATS 采用 **HDI 服务测试模式**：

1. **HDI Service 测试**：直接调用 HDI 接口验证驱动实现
2. **HDI Adapter 测试**：测试适配层逻辑
3. **配置测试**：基于 HCS（Hardware Configuration Source）的 JSON 配置验证

### 组件配置（bundle.json）

```json
{
  "name": "@ohos/hats",
  "version": "4.0",
  "component": {
    "name": "hats",
    "subsystem": "xts",
    "features": [
      "hats_rich",
      "hats_nnrt",
      "hats_drivers_peripheral_power_wakeup_cause_path",
      "hats_drivers_peripheral_battery_pc_macro_isolation"
    ],
    "adapted_system_type": ["mini", "small", "standard"]
  }
}
```

---

## 快速开始

### 编译测试

```bash
# 标准系统编译
./build.sh product_name=hispark_taurus_standard suite=hats system_size=standard

# 输出目录
out/hispark_taurus/suites/hats/testcases
```

### 运行测试

```bash
# NFS 方式挂载到设备
mount 192.168.1.10:/nfs /nfs nfs
cd /nfs/hats
./run.bat
```

### 查看测试报告

```
suites/hats/reports/summary_report.html
```

---

## 相关文档

| 文档 | 路径 | 说明 |
|------|------|------|
| 架构说明 | [01_Architecture.md](./01_Architecture.md) | 系统架构详解 |
| 子系统 | [02_Modules.md](./02_Modules.md) | 各模块详解 |
| 构建系统 | [04_Build.md](./04_Build.md) | GN 构建配置 |
| 安全评审 | [05_Security.md](./05_Security.md) | 安全风险分析 |
