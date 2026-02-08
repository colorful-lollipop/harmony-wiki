# 配置说明

## 4.1 配置概述

测试框架使用 XML 格式的配置文件，所有配置文件位于 `config/` 目录下。

### 配置文件列表

| 文件 | 说明 | 位置 |
|------|------|------|
| `user_config.xml` | 用户配置 | `config/user_config.xml` |
| `framework_config.xml` | 框架配置 | `config/framework_config.xml` |
| `build_config.xml` | 构建配置 | `config/build_config.xml` |
| `filter_config.xml` | 过滤配置 | `config/filter_config.xml` |
| `fuzz_config.xml` | Fuzz 配置 | `config/fuzz_config.xml` |

## 4.2 用户配置 (user_config.xml)

### 4.2.1 配置结构

```xml
<?xml version="1.0" encoding="utf-8"?>
<user_config>
  <build>...</build>           <!-- 构建配置 -->
  <environment>...</environment> <!-- 环境配置 -->
  <test_cases>...</test_cases> <!-- 用例路径 -->
  <coverage>...</coverage>      <!-- 覆盖率配置 -->
  <NFS>...</NFS>              <!-- NFS 挂载 -->
</user_config>
```

### 4.2.2 构建配置 (build)

| 元素 | 属性 | 默认值 | 说明 |
|------|------|--------|------|
| `<example>` | - | `false` | 是否编译示例用例 |
| `<version>` | - | `false` | 是否编译版本 |
| `<testcase>` | - | `false` | 是否编译测试用例 |
| `<parameter>` | - | - | 编译参数 |
| | `<target_cpu>` | 空 | 目标 CPU (arm64) |

**证据**: `config/user_config.xml:17-28`

### 4.2.3 环境配置 (environment)

#### HDC 设备配置

```xml
<device type="usb-hdc" label="ohos">
  <info ip="" port="" sn="" alias="" />
</device>
```

| 属性 | 说明 |
|------|------|
| `type` | 设备类型 (`usb-hdc`) |
| `label` | 设备标签 |
| `<ip>` | 设备 IP 地址 |
| `<port>` | HDC 端口 |
| `<sn>` | 设备序列号 |

#### 串口设备配置

```xml
<device type="com" label="ipcamera">
  <serial>
    <com></com>
    <type>cmd</type>
    <baud_rate>115200</baud_rate>
    <data_bits>8</data_bits>
    <stop_bits>1</stop_bits>
    <timeout>1</timeout>
  </serial>
</device>
```

| 属性 | 默认值 | 说明 |
|------|--------|------|
| `baud_rate` | `115200` | 波特率 |
| `data_bits` | `8` | 数据位 |
| `stop_bits` | `1` | 停止位 |
| `timeout` | `1` | 超时时间(秒) |

**证据**: `config/user_config.xml:29-45`

### 4.2.4 测试用例路径 (test_cases)

```xml
<test_cases>
  <dir>/home/test/out/release/tests</dir>
</test_cases>
```

### 4.2.5 覆盖率配置 (coverage)

```xml
<coverage>
  <outpath></outpath>
</coverage>
```

### 4.2.6 NFS 配置 (NFS)

```xml
<NFS>
  <host_dir>D:\nfs</host_dir>
  <mnt_cmd></mnt_cmd>
  <board_dir>user</board_dir>
</NFS>
```

| 元素 | 说明 |
|------|------|
| `<host_dir>` | 主机 NFS 目录 |
| `<board_dir>` | 板端目录 |
| `<mnt_cmd>` | 挂载命令 |

## 4.3 框架配置 (framework_config.xml)

### 4.3.1 产品形态 (productform)

```xml
<productform>
  <option name="rk3568" />
  <option name="ipcamera_hispark_aries" />
  <option name="ipcamera_hispark_taurus" />
  <option name="wifiiot_hispark_pegasus" />
</productform>
```

**证据**: `config/framework_config.xml:17-22`

### 4.3.2 测试类型 (test_category)

| 类型 | 描述 | 超时(秒) |
|------|------|---------|
| `UT` | 单元测试 | 300 |
| `ACTS` | 活动测试 | 300 |
| `HATS` | 硬件抽象测试 | 300 |
| `HITS` | HIT 测试 | 300 |
| `MST` | 模块测试 | 300 |
| `ST` | 系统测试 | 300 |
| `PERF` | 性能测试 | 900 |
| `SEC` | 安全测试 | 900 |
| `FUZZ` | 模糊测试 | 900 |
| `RELI` | 可靠性测试 | 900 |
| `DST` | 分布式测试 | 900 |
| `BENCHMARK` | 基准测试 | 300 |
| `ARKTSTDD` | ArkTS TDD | 900 |

**证据**: `config/framework_config.xml:23-65`

## 4.4 构建配置 (build_config.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<build>
  <!-- 构建模板配置 -->
</build>
```

**证据**: `config/build_config.xml`

## 4.5 过滤配置 (filter_config.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<filter>
  <!-- 用例过滤规则 -->
</filter>
```

**证据**: `config/filter_config.xml`

## 4.6 Fuzz 配置 (fuzz_config.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<fuzz_config>
  <!-- Fuzz 测试参数 -->
</fuzz_config>
```

**证据**: `config/fuzz_config.xml`

## 4.7 配置使用流程

```
1. 修改 user_config.xml
   └─→ 设置设备信息、NFS 路径

2. (可选) 修改 framework_config.xml
   └─→ 添加新产品形态

3. 执行 ./start.sh
   └─→ 框架自动读取配置

4. 运行测试
   └─→ 根据配置执行
```

## 4.8 配置验证

### 配置检查点

| 检查项 | 说明 |
|-------|------|
| 设备连接 | HDC 连接或串口连接 |
| NFS 挂载 | 串口设备必需 |
| 权限 | 文件/目录读写权限 |
| 网络 | 设备 IP 可达 |

### 常见配置错误

| 错误 | 原因 | 解决方法 |
|------|------|---------|
| 设备不识别 | IP/端口错误 | 检查 user_config.xml |
| NFS 挂载失败 | 目录不存在 | 创建目录并配置 |
| 用例编译失败 | GN 配置错误 | 检查 BUILD.gn |
| 覆盖率无数据 | 未开启覆盖率 | 添加 `-cov` 参数 |

## 4.9 相关文档

- [06_Usage_Guide.md](06_Usage_Guide.md) - 使用指南
- [05_Build_System.md](05_Build_System.md) - 构建系统
- [README_zh.md](../README_zh.md) - 原始中文文档
