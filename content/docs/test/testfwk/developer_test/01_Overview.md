# 项目概览

## 1.1 项目定位

**Developer Test Framework** 是 OpenHarmony 测试子系统的核心组件，为开发者提供一套完整的**自测试执行框架**。开发者可根据测试需求开发相关测试用例，在开发阶段提前发现缺陷，大幅提高代码质量。

### 核心价值

- **用例开发支持**: 支持 C++/JS/ArkTS 多种语言的测试用例开发
- **自动化执行**: 支持测试用例的自动化编译、执行、结果收集
- **多设备适配**: 支持标准设备、Lite 设备等多种硬件形态
- **多类型测试**: 覆盖单元测试、性能测试、模糊测试、分布式测试等

### 目标用户

| 用户类型 | 使用场景 |
|---------|---------|
| 应用开发者 | 验证应用功能的正确性 |
| 系统开发者 | 验证系统模块的正确性和性能 |
| 测试工程师 | 执行回归测试、完整性测试 |

## 1.2 核心能力

### 测试类型支持

| 测试类型 | 说明 | 配置文件 |
|---------|------|---------|
| **UT** (单元测试) | 验证单个函数/模块功能 | unittest |
| **MST** (模块测试) | 验证模块间交互 | moduletest |
| **ST** (系统测试) | 验证系统级功能 | systemtest |
| **PERF** (性能测试) | 性能基准测试 | performance |
| **FUZZ** (模糊测试) | 安全性/稳定性测试 | fuzztest |
| **RELI** (可靠性测试) | 长时间运行稳定性 | reliability |
| **DST** (分布式测试) | 分布式场景测试 | distributedtest |
| **BENCHMARK** | 性能基准对比 | benchmark |
| **ACTS** | 活动组件测试 | actstest |
| **HATS** | 硬件抽象测试 | hatstest |
| **ARKTSTDD** | ArkTS TDD 测试 | arktstdd |

### 设备支持

| 设备类型 | 连接方式 | 说明 |
|---------|---------|------|
| 标准设备 (rk3568) | HDC | 支持完整系统功能 |
| IPCamera (hispark_aries) | HDC/串口 | 支持 Lite 系统 |
| IPCamera (hispark_taurus) | HDC/串口 | 支持 Lite 系统 |
| WiFi IoT (hispark_pegasus) | HDC/串口 | 轻量设备 |

### 编程语言支持

| 语言 | 测试框架 | 示例 |
|------|---------|------|
| **C++** | gtest + hwext | calculator, detector |
| **JavaScript** | deccjsunit | app_info |
| **ArkTS** | hypium | stagetest |

## 1.3 运行环境

### 系统要求

| 组件 | 版本要求 | 说明 |
|------|---------|------|
| Python | 3.7.5+ | 运行环境 |
| Paramiko | 2.7.1+ | SSH 库 |
| Setuptools | 40.8.0+ | 包管理 |
| RSA | 4.0+ | 加密支持 |
| NFS | V4+ | 串口设备挂载 |
| pySerial | 3.3+ | 串口通信 |

### 操作系统

- **Windows**: Windows 10+
- **Linux**: Ubuntu 18.04+

### 硬件要求

- 开发板运行 OpenHarmony 系统
- 主机与开发板网络互通
- HDC 工具或串口连接

### 依赖组件

```text
testfwk_xdevice/          # 测试调度框架（同级依赖）
├── xdevice/              # 核心调度
└── ...
```

## 1.4 关键概念

### 产品形态 (Product Form)

产品形态指不同的硬件设备配置，每个产品形态对应特定的编译配置和测试目标。

支持的设备形态（来自 `config/framework_config.xml:17-22`）:
- `rk3568` - 标准设备
- `ipcamera_hispark_aries` - IPCamera Aries
- `ipcamera_hispark_taurus` - IPCamera Taurus  
- `wifiiot_hispark_pegasus` - WiFi IoT Pegasus

### 测试套 (Test Suite)

测试套是测试用例的集合，通常对应一个模块或功能点。

```cpp
// 示例：C++ 测试套
class CalculatorSubTest : public testing::Test {
    // 测试套 Setup/Teardown
    static void SetUpTestCase(void);
    static void TearDownTestCase(void);
    void SetUp();
    void TearDown();
};

// 测试用例
HWTEST_F(CalculatorSubTest, integer_sub_001, TestSize.Level1)
```

### 测试级别 (Test Level)

| 级别 | 说明 | 门禁 |
|------|------|------|
| Level0 | 门禁用例 | ✅ |
| Level1 | 重要功能 | ❌ |
| Level2 | 一般功能 | ❌ |
| Level3 | 次要功能 | ❌ |
| Level4 | 边缘功能 | ❌ |

### 覆盖率 (Coverage)

| 覆盖率类型 | 说明 |
|-----------|------|
| 代码覆盖率 | C/C++ 代码行覆盖、分支覆盖 |
| 接口覆盖率 | API 接口调用覆盖 |

## 1.5 版本信息

| 版本 | 发布日期 | 主要特性 |
|------|---------|---------|
| 3.2.1.0 | - | 增加 ACTS 测试能力 |
| 3.2.2.0 | - | 增加任务统计、复测能力 |
| 3.2.3.0 | - | 增加覆盖率执行 |

## 1.6 相关文档

- [02_Architecture.md](02_Architecture.md) - 系统架构
- [03_Directory_Structure.md](03_Directory_Structure.md) - 目录结构
- [06_Usage_Guide.md](06_Usage_Guide.md) - 使用指南
- [README_zh.md](../README_zh.md) - 原始中文文档
