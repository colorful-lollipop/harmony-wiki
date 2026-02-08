# DCTS 项目概览

## 项目定位

**DCTS**（Distributed Compatibility Test Suite，分布式兼容性测试套件）是 OpenHarmony XTS（X Test Suite）测试子系统的重要组成部分，用于验证设备间的分布式场景兼容性。

### 核心价值

- **早期发现问题**：帮助设备厂商尽早检测分布式场景的不兼容性
- **持续兼容性保障**：确保软件在整个开发过程中与 OpenHarmony 保持兼容
- **多系统支持**：覆盖 Standard、Small、Mini 三种系统类型

## 适用范围

### 支持的系统类型

| 系统类型 | 内存要求 | 处理器 | 测试框架 |
|----------|----------|--------|----------|
| **Mini 系统** | ≥ 128 KiB | ARM Cortex-M, RISC-V | HCTest (C) |
| **Small 系统** | ≥ 1 MiB | ARM Cortex-A | HCPPTest (C++) |
| **Standard 系统** | ≥ 128 MiB | ARM Cortex-A | HJSUnit / HCPPTest |

> **当前状态**：DCTS 主要支持 Standard 系统，Mini/Small 系统支持正在进行中。

### 典型应用场景

- 智能家居设备互联
- 可穿戴设备协同
- 分布式数据同步
- 跨设备任务流转

## 核心能力

### 测试类型覆盖

**证据**: `README.md:56-78`

| 类型 | 说明 | 测试框架 |
|------|------|----------|
| **功能测试** | 验证服务功能的正确性 | HJSUnit / HCPPTest / HCTest |
| **性能测试** | 测试处理能力指标（QPS、FPS）| HJSUnit / HCPPTest |
| **可靠性测试** | 稳定性、压力、故障注入 | HJSUnit / HCPPTest |
| **安全测试** | 防御安全威胁能力 | HJSUnit / HCPPTest |
| **兼容性测试** | 向前/向后兼容性验证 | HJSUnit / HCPPTest |

### 测试粒度

**证据**: `README.md:113-143`

| 粒度 | 测试对象 | 环境 | TestType 注解 |
|------|----------|------|--------------|
| **LargeTest** | 服务功能、全场景特性 | 接近真实设备 | TestType.LARGE |
| **MediumTest** | 模块、集成后功能 | 单设备实际使用 | TestType.MEDIUM |
| **SmallTest** | 模块、类、函数 | 本地 PC，大量 Mock | TestType.SMALL |

**测试框架使用示例**：

```javascript
// 证据：distributedhardware/distributeddevicejstest/entry/src/ohosTest/js/test/distributedDevice.test.js:16
import { describe, beforeAll, beforeEach, afterEach, afterAll, it, expect, TestType, Size, Level } from '@ohos/hypium'

export default function distributedDeviceManager() {
    describe('distributedDeviceManager', function () {
        // LargeTest：全场景测试
        it('SUB_DH_DeviceManager_Dcts_0100', TestType.FUNCTION | Size.LARGETEST | Level.LEVEL0, ...)

        // MediumTest：模块集成测试
        it('SUB_DH_DeviceManager_Dcts_0200', TestType.FUNCTION | Size.MEDIUMTEST | Level.LEVEL1, ...)

        // SmallTest：单元测试
        it('SUB_DH_DeviceManager_Dcts_0300', TestType.FUNCTION | Size.SMALLTEST | Level.LEVEL2, ...)
    })
}
```

## 运行环境

### 开发环境要求

- **操作系统**: Linux (推荐 Ubuntu 20.04+)
- **构建工具**: hb (OpenHarmony 构建工具) + GN + Ninja
- **Python 版本**: 3.8+
- **Node.js 版本**: 14+ (JS 测试)

### 依赖组件

**证据**: `bundle.json:61-30`

```json
{
  "deps": {
    "components": [
      "access_token",      // 权限管理
      "c_utils",            // C 工具库
      "dsoftbus",           // 分布式软总线
      "hilog",              // 日志
      "ipc",                // 进程间通信
      "samgr",              // 系统能力管理
      "wifi"                // 无线网络
    ]
  }
}
```

**说明**：
- `access_token`: 提供 `ohos.permission.*` 权限申请能力
- `dsoftbus`: 分布式通信底层依赖
- `ipc`: 进程间通信机制
- `samgr`: 系统能力管理和查询

### 目标设备

- 标准系统设备（TV、手机、平板等）
- 小型系统设备（智能摄像头、路由器等）
- 微型系统设备（传感器、可穿戴等）

## 关键概念

### 测试级别 (Level)

**证据**: `README.md:74-108`

| 级别 | 名称 | 范围 | Level 注解 |
|------|------|------|-----------|
| Level0 | Smoke | 基本功能验证，最常见输入 | Level.LEVEL0 |
| Level1 | Basic | 基本功能验证，普通输入 | Level.LEVEL1 |
| Level2 | Major | 功能验证，错误处理 | Level.LEVEL2 |
| Level3 | Regular | 所有功能，全面输入组合 | Level.LEVEL3 |
| Level4 | Rare | 极端条件，异常输入组合 | Level.LEVEL4 |

**测试用例示例**：

```javascript
// 证据：filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:3701
it('test_filefs_write_file_000', Level.LEVEL0, async function (done) {
    // Smoke 测试：基本文件写入功能
    let fpath = await getDistributedFilePath(tcNumber);
    let file = fs.openSync(fpath, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
    fs.writeSync(file.fd, DISTRIBUTED_FILE_CONTENT);
    fs.closeSync(file);
    done();
})

// 证据：communication/dsoftbus_rpcets/rpcclient/entry/src/ohosTest/ets/test/RpcRequestEtsUnit.test.ets:2117-2200
it("SUB_DSoftbus_RPC_API_NEW_MessageSequence_0520", Level.LEVEL3, async () => {
    // Regular 测试：边界值测试（溢出行为）
    data.writeByte(128);   // 超过最大值
    expect(result.reply.readByte()).assertEqual(-128);  // 验证回绕
})
```

### 关键术语

| 术语 | 说明 | 相关 API |
|------|------|----------|
| **HAP** | Harmony Ability Package，应用安装包 | `.hap` 文件，Test.json |
| **FA** | Feature Ability，特征能力（旧模式） | `@ohos.ability.featureAbility` |
| **Stage** | 舞台模式，OpenHarmony 应用开发新范式 | `UIAbility` |
| **RPC** | Remote Procedure Call，远程过程调用 | `@ohos.rpc` |
| **IPC** | Inter-Process Communication，进程间通信 | `@ohos.ipc` |
| **SA** | System Ability，系统能力 | 系统服务接口 |

**API 调用示例**：

```javascript
// 证据：filemanagement/fileio/client/entry/src/ohosTest/js/test/FileioJsUnit.test.js:16-20
import rpc from "@ohos.rpc";
import fs from '@ohos.file.fs';
import securityLabel from '@ohos.file.securityLabel';
import featureAbility from "@ohos.ability.featureAbility";

// Feature Ability (FA) 调用
let context = featureAbility.getContext();

// RPC 连接
let connection = rpc.connectAbility(...);

// 文件系统操作
let file = fs.openSync(path, mode);
securityLabel.setSecurityLabelSync(path, "s0");
```

## 项目边界

### 包含内容

```
test/xts/dcts/
├── ability/          # 分布式能力测试
├── communication/    # 通信测试
├── distributeddatamgr/  # 数据管理测试
├── distributedhardware/ # 硬件设备测试
├── filemanagement/   # 文件管理测试
├── multimedia/       # 多媒体测试
├── testtools/        # 测试工具
└── common/           # 公共模块
```

### 不包含内容

- **测试套件本身**：HCTest、HCPPTest、Hypium 等测试框架（位于单独仓库）
- **被测系统 API**：由 OpenHarmony 各子系统提供
- **设备侧运行时**：运行在目标设备上的测试执行器

## 相关文档

- [目录结构详解](02_Directory_Structure.md)
- [架构设计](03_Architecture.md)
- [模块说明](04_Modules.md)
- [构建系统](05_Build_System.md)
