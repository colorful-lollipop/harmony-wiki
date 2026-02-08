# XDevice 配置说明

## 1. 配置架构

### 1.1 配置类型

| 类型 | 格式 | 说明 |
|------|------|------|
| 用户配置 | XML | `user_config.xml` - 环境、设备、日志配置 |
| 模块配置 | JSON | `.json` 文件 - 测试套件定义 |
| 命令参数 | - | 命令行 `-c`, `-env` 等 |

### 1.2 配置加载优先级

```
1. 命令行 -c 参数指定
2. ./config/user_config.xml
3. $XDEVICE_CONFIG/user_config.xml
4. 内置默认配置
```

---

## 2. user_config.xml 详解

### 2.1 完整配置结构

**文件**: `config/user_config.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<user_config>
    <!-- ==================== 设备环境配置 ==================== -->
    <environment>
        <!-- 标准系统设备 (USB-HDC 连接) -->
        <device type="usb-hdc" label="ohos">
            <info ip="" port="" sn="" alias=""/>
        </device>
        
        <!-- 轻量系统设备 (串口连接) -->
        <device type="com" label="wifiiot">
            <serial>
                <com></com>
                <type>cmd</type>
                <baud_rate>115200</baud_rate>
                <data_bits>8</data_bits>
                <stop_bits>1</stop_bits>
                <timeout>20</timeout>
            </serial>
            <serial>
                <com></com>
                <type>deploy</type>
                <baud_rate>115200</baud_rate>
            </serial>
        </device>
        
        <!-- 小型系统设备 -->
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
    </environment>
    
    <!-- ==================== 测试用例配置 ==================== -->
    <testcases>
        <dir></dir>
        <server label="NfsServer">
            <ip></ip>
            <port></port>
            <dir></dir>
            <username></username>
            <password></password>
            <remote></remote>
        </server>
    </testcases>
    
    <!-- ==================== 资源文件配置 ==================== -->
    <resource>
        <dir></dir>
        <web_resource>
            <enable>FALSE</enable>
            <url></url>
        </web_resource>
    </resource>
    
    <!-- ==================== 设备日志配置 ==================== -->
    <devicelog>
        <enable>ON</enable>
        <loglevel>INFO</loglevel>
        <dir></dir>
        <clear>TRUE</clear>
        <hdc>FALSE</hdc>
        <suitecaselog>ON</suitecaselog>
    </devicelog>
    
    <!-- ==================== 全局日志级别 ==================== -->
    <loglevel>INFO</loglevel>
    
    <!-- ==================== 集群配置 ==================== -->
    <cluster>
        <enable>false</enable>
        <service_mode>controller</service_mode>
        <service_port></service_port>
        <control_service_url>http://127.0.0.1:8000</control_service_url>
    </cluster>
</user_config>
```

### 2.2 设备配置详解

#### 2.2.1 USB-HDC 设备

```xml
<device type="usb-hdc" label="ohos">
    <info ip="" port="" sn="" alias=""/>
</device>
```

| 属性 | 说明 |
|------|------|
| `type` | 连接类型：`usb-hdc` |
| `label` | 设备标签：`ohos` |
| `ip` | 远程设备 IP（空表示本地） |
| `port` | 远程设备端口 |
| `sn` | 设备序列号（空表示所有设备） |
| `alias` | 设备别名 |

#### 2.2.2 串口设备

```xml
<device type="com" label="wifiiot">
    <serial>
        <com></com>
        <type>cmd|deploy</type>
        <baud_rate>115200</baud_rate>
        <data_bits>8</data_bits>
        <stop_bits>1</stop_bits>
        <timeout>20</timeout>
    </serial>
</device>
```

| 属性 | 说明 | 默认值 |
|------|------|-------|
| `com` | 串口号 | - |
| `type` | 串口类型：`cmd` 或 `deploy` | - |
| `baud_rate` | 波特率 | 115200 |
| `data_bits` | 数据位 | 8 |
| `stop_bits` | 停止位 | 1 |
| `timeout` | 超时（秒） | 20 |

---

## 3. JSON 模块配置

### 3.1 ACTS 配置示例

**文件**: `config/acts.json`

```json
{
    "description": "Configuration for ACTS Tests",
    "environment": [
        {
            "type": "device",
            "label": "phone"
        }
    ],
    "driver": {
        "type": "CppTest",
        "xml-output": false,
        "rerun": false
    },
    "kits": [
        {
            "type": "ShellKit",
            "run-command": [
                "remount",
                "mkdir /data/data/resource"
            ],
            "teardown-command": [
                "remount",
                "rm -rf /data/data/resource"
            ]
        }
    ]
}
```

### 3.2 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `description` | string | 测试套件描述 |
| `environment` | array | 运行环境配置 |
| `driver` | object | 测试驱动配置 |
| `kits` | array | 测试工具包配置 |
| `subsystem` | string | 子系统名称 |
| `part` | string | 部件名称 |

---

## 4. 命令行参数

### 4.1 run 命令参数

```bash
xdevice run [options] action task
```

| 参数 | 说明 | 示例 |
|------|------|------|
| `-l, --testlist` | 测试列表 | `-l module1;module2` |
| `-tc, --testcase` | 测试用例 | `-tc TestClass.testMethod` |
| `-tf, --testfile` | 测试文件 | `-tf test.json` |
| `-c, --config` | 配置文件 | `-c user_config.xml` |
| `-env, --environment` | XML 环境字符串 | `-env "<environment>...</environment>"` |
| `-sn, --device_sn` | 设备序列号 | `-sn 123456` |
| `-rp, --report_path` | 报告路径 | `-rp ./reports` |
| `-respath` | 资源路径 | `-respath ./resource` |
| `-tcpath` | 用例路径 | `-tcpath ./testcases` |
| `--retry` | 重试失败用例 | `--retry --session xxx` |
| `--reboot-per-module` | 模块执行前重启 | `--reboot-per-module` |

---

## 5. 配置验证

### 5.1 验证规则

| 规则 | 说明 |
|------|------|
| 设备别名唯一 | 不允许重复的设备别名 |
| SN 格式验证 | TCP/IP 设备需符合 IP:PORT 格式 |
| 串口配置完整 | cmd 和 deploy 串口需配置完整 |

### 5.2 错误码

| 错误码 | 说明 |
|--------|------|
| Code_0103001 | user_config.xml 不存在 |
| Code_0103002 | XML 解析失败 |
| Code_0103004 | 设备别名重复 |
| Code_0101027 | JSON 文件不存在 |
| Code_0101029 | 配置项应为字典 |

---

## 相关文档

- [使用指南](07_Usage.md)
- [架构说明](02_Architecture.md)
