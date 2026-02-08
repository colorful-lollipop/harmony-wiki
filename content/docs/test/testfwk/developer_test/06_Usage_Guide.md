# 使用指南

## 6.1 环境准备

### 6.1.1 Python 环境依赖

```bash
# Ubuntu/Debian
sudo apt-get install libreadline-dev
pip3 install setuptools
pip3 install paramiko
pip3 install rsa
pip3 install pyserial
```

### 6.1.2 版本要求

| 组件 | 最低版本 |
|------|---------|
| Python | 3.7.5 |
| Paramiko | 2.7.1 |
| Setuptools | 40.8.0 |
| RSA | 4.0 |
| NFS | V4 |
| pySerial | 3.3 |

## 6.2 快速开始

### 6.2.1 启动测试框架

```bash
# Linux
./start.sh

# Windows
start.bat
```

### 6.2.2 选择产品形态

启动后系统会提示选择产品形态，根据实际开发板选择。

支持的设备形态：
- `rk3568` - 标准设备
- `ipcamera_hispark_aries` - IPCamera Aries
- `ipcamera_hispark_taurus` - IPCamera Taurus
- `wifiiot_hispark_pegasus` - WiFi IoT Pegasus

### 6.2.3 运行测试

```bash
# 运行所有单元测试
run -t UT

# 运行指定部件测试
run -t UT -tp PartName

# 运行指定模块测试
run -t UT -tp PartName -tm TestModuleName

# 运行指定测试套
run -t UT -tp PartName -tm TestModuleName -ts CalculatorSubTest

# 运行指定测试用例
run -t UT -ts CalculatorSubTest -tc CalculatorSubTest.integer_sub_001
```

## 6.3 命令参考

### 6.3.1 show 命令

| 命令 | 说明 |
|------|------|
| `show productlist` | 查询支持的产品形态 |
| `show typelist` | 查询支持的测试类型 |
| `show subsystemlist` | 查询支持的子系统 |
| `show modulelist` | 查询支持的模块 |

### 6.3.2 run 命令参数

```bash
run [-h] [-p PRODUCTFORM] [-t [TESTTYPE [TESTTYPE ...]]]
    [-ss SUBSYSTEM] [-tm TESTMODULE] [-ts TESTSUIT]
    [-tc TESTCASE] [-tl TESTLEVEL] [-cov COVERAGE]
    [-ra random] [-pd partdeps] [--repeat REPEAT]
    [--hl] [--rh RUNHISTORY] [--retry]
```

| 参数 | 说明 |
|------|------|
| `-p, --productform` | 指定产品形态 |
| `-t, --testtype` | 指定测试类型 (UT, MST, ST, PERF, FUZZ, etc.) |
| `-ss, --subsystem` | 指定子系统 |
| `-tm, --testmodule` | 指定模块 |
| `-ts, --testsuite` | 指定测试套 |
| `-tc, --testcase` | 指定测试用例 |
| `-tl, --testlevel` | 指定测试级别 |
| `-cov, --coverage` | 覆盖率执行参数 |
| `-ra, --random` | 用例乱序执行 |
| `-pd, --partdeps` | 二级依赖部件执行 |
| `--repeat` | 用例执行次数 |
| `-hl` | 显示历史记录 |
| `-rh` | 执行历史记录 |
| `--retry` | 复测失败用例 |

### 6.3.3 其他命令

| 命令 | 说明 |
|------|------|
| `help` | 显示帮助信息 |
| `gen` | 生成代码 |
| `version` | 显示版本 |
| `quit` | 退出框架 |

## 6.4 测试类型详细说明

### 6.4.1 UT (单元测试)

```bash
# C++ 单元测试
run -t UT -tp PartName -tm ModuleName -ts TestSuite

# JS 单元测试
run -t UT -ss subsystem -ts ActsXXXTest
```

### 6.4.2 PERF (性能测试)

```bash
run -t PERF -tp PartName -tm ModuleName -ts PerfTestSuite
```

### 6.4.3 FUZZ (模糊测试)

```bash
run -t FUZZ -tp PartName -tm ModuleName -ts FuzzTestSuite
```

### 6.4.4 ACTS (活动测试)

```bash
# 运行所有 ACTS 测试
run -t ACTS

# 运行指定子系统
run -t ACTS -ss arkui

# 运行指定测试套
run -t ACTS -ss arkui -ts ActsAceEtsTest

# 运行指定用例
run -t ACTS -ss arkui -ts ActsAceEtsTest -ta class:alphabetIndexerTest#alphabetIndexerTest001
```

### 6.4.5 DST (分布式测试)

```bash
run -t DST -tp PartName -tm ModuleName -ts DistributedTestSuite
```

## 6.5 配置说明

### 6.5.1 user_config.xml 配置

#### HDC 设备配置

```xml
<device type="usb-hdc">
  <ip>192.168.1.100</ip>
  <port>9111</port>
  <sn></sn>
</device>
```

#### 串口设备配置

```xml
<device type="com" label="ipcamera">
  <serial>
    <com>COM1</com>
    <type>cmd</type>
    <baud_rate>115200</baud_rate>
    <data_bits>8</data_bits>
    <stop_bits>1</stop_bits>
    <timeout>1</timeout>
  </serial>
</device>
```

#### NFS 配置

```xml
<NFS>
  <host_dir>D:\nfs</host_dir>
  <board_dir>user</board_dir>
</NFS>
```

### 6.5.2 用例路径配置

```xml
<test_cases>
  <dir>/home/test/out/release/tests</dir>
</test_cases>
```

## 6.6 测试报告

### 6.6.1 报告位置

```
reports/xxxx_xx_xx_xx_xx_xx/
├── result/                     # 测试结果
│   └── ...
├── log/                       # 测试日志
│   └── plan_log_xxxx_xx_xx_xx_xx_xx.log
├── summary_report.html         # 汇总报告
├── details_report.html         # 详细报告
└── platform_log_xxxx_xx_xx_xx_xx_xx.log  # 框架日志
```

### 6.6.2 最新报告

```
reports/latest/
└── [指向最新报告的软链接]
```

## 6.7 覆盖率

### 6.7.1 覆盖率执行

```bash
run -t UT -tp PartName -cov coverage
```

### 6.7.2 覆盖率要求

编译时需添加 `--coverage` 标志：

```gn
ldflags = [ "--coverage" ]
cflags = [ "--coverage" ]
cflags_cc = [ "--coverage" ]
```

### 6.7.3 覆盖率报告

| 类型 | 路径 |
|------|------|
| 代码覆盖率 | `localCoverage/codeCoverage/results/coverage/reports/cxx/html` |
| 接口覆盖率 | `localCoverage/interfaceCoverage/results/coverage/interface_kits/html` |

## 6.8 常见问题

### Q1: 设备无法连接

**解决方法**：
1. 检查 HDC 工具是否安装
2. 检查设备 IP 和端口配置
3. 检查网络连通性

### Q2: 用例编译失败

**解决方法**：
1. 检查 BUILD.gn 语法
2. 检查依赖配置
3. 查看编译日志

### Q3: 测试用例无结果

**解决方法**：
1. 检查用例路径配置
2. 检查设备端用例是否推送成功
3. 查看设备日志

### Q4: 覆盖率无数据

**解决方法**：
1. 确认编译时添加了 `--coverage`
2. 确认执行时使用了 `-cov coverage`
3. 检查覆盖率输出路径

## 6.9 最佳实践

### 6.9.1 用例开发规范

1. **命名规范**
   - 测试套名称：大驼峰风格
   - 测试用例名称：`[功能]_[编号]`，编号 3 位数字

2. **用例结构**
   ```cpp
   class TestSuiteName : public testing::Test {
       static void SetUpTestCase(void);
       static void TearDownTestCase(void);
       void SetUp();
       void TearDown();
   };

   HWTEST_F(TestSuiteName, test_case_001, TestSize.Level1)
   ```

3. **用例注释**
   ```cpp
   /**
    * @tc.name: test_case_001
    * @tc.desc: 测试用例描述
    * @tc.type: FUNC
    * @tc.require: issueI56WJ7
    */
   ```

### 6.9.2 配置管理

1. 使用版本控制管理 `user_config.xml`
2. 不同环境使用不同配置文件
3. 敏感信息勿提交到版本控制

## 6.10 相关文档

- [01_Overview.md](01_Overview.md) - 项目概览
- [02_Architecture.md](02_Architecture.md) - 系统架构
- [04_Configuration.md](04_Configuration.md) - 配置说明
- [05_Build_System.md](05_Build_System.md) - 构建系统
- [07_Examples.md](07_Examples.md) - 示例说明
- [README_zh.md](../README_zh.md) - 原始中文文档
