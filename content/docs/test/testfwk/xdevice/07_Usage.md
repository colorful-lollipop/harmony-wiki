# XDevice 使用指南

## 1. 安装

### 1.1 环境要求

| 依赖 | 版本要求 |
|------|---------|
| Python | >= 3.7.5 |
| pySerial | >= 3.3 |
| Paramiko | >= 2.7.1 |
| RSA | >= 4.0 |

### 1.2 安装步骤

```bash
# 1. 安装 XDevice 主包
cd /path/to/xdevice
python setup.py sdist
pip install dist/xdevice-*.tar.gz

# 2. 安装 ohos 插件（必需）
cd plugins/ohos
python setup.py sdist
pip install dist/xdevice-ohos-*.tar.gz

# 3. 安装 devicetest 插件（可选）
cd plugins/devicetest
python setup.py sdist
pip install dist/xdevice-devicetest-*.tar.gz
```

### 1.3 验证安装

```bash
xdevice --help
```

---

## 2. 配置

### 2.1 编辑 user_config.xml

```bash
# 复制模板配置文件
cp config/user_config.xml user_config.xml
# 根据实际环境编辑配置
```

### 2.2 配置设备连接

#### USB-HDC 设备

```xml
<device type="usb-hdc" label="ohos">
    <info ip="" port="" sn="" alias=""/>
</device>
```

#### 串口设备

```xml
<device type="com" label="wifiiot">
    <serial>
        <com>COM20</com>
        <type>cmd</type>
        <baud_rate>115200</baud_rate>
    </serial>
</device>
```

---

## 3. 命令

### 3.1 help 命令

```bash
# 显示帮助
xdevice help

# 显示 run 命令帮助
xdevice help run

# 显示 list 命令帮助
xdevice help list
```

### 3.2 list 命令

```bash
# 列出设备
xdevice list

# 列出历史任务
xdevice list history

# 列出特定任务详情
xdevice list <task_id>
```

### 3.3 run 命令

#### 基本用法

```bash
# 运行 ACTS 测试套件
xdevice run acts -l module1;module2

# 运行 HIT 测试套件
xdevice run hits -l module1

# 运行 SSTS 测试套件
xdevice run ssts -l module1
```

#### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-l` | 测试模块列表 | `-l module1;module2` |
| `-sn` | 指定设备 SN | `-sn 123456` |
| `-c` | 配置文件 | `-c user_config.xml` |
| `-rp` | 报告路径 | `-rp ./reports` |
| `--retry` | 重试失败用例 | `--retry --session xxx` |
| `--reboot-per-module` | 模块执行前重启 | `--reboot-per-module` |

---

## 4. 执行测试

### 4.1 准备测试环境

```bash
# 1. 确保设备已连接
xdevice list

# 2. 检查设备状态
xdevice list devices
```

### 4.2 执行第一个测试

```bash
# 使用默认配置执行
xdevice run acts -l <module_name>

# 指定配置文件
xdevice run acts -c user_config.xml -l <module_name>

# 指定设备
xdevice run acts -sn <device_sn> -l <module_name>
```

### 4.3 监控执行

- 控制台实时输出日志
- 报告实时更新

---

## 5. 查看结果

### 5.1 报告结构

```
reports/
├── result/                    # XML 结果
│   └── module_name.xml
├── log/                       # 日志
│   ├── task_log.log
│   └── device_*.log
├── summary_report.html        # HTML 汇总报告
├── summary_data_report.xml    # 数据报告
├── task_info.record          # 任务记录
└── summary.ini               # 摘要
```

### 5.2 查看报告

```bash
# 使用浏览器打开 HTML 报告
open reports/summary_report.html
```

---

## 6. 常见操作

### 6.1 重试失败用例

```bash
# 使用上次的任务记录重试
xdevice run --retry --session <report_dir>
```

### 6.2 分布式执行

```xml
<!-- user_config.xml 配置 -->
<cluster>
    <enable>true</enable>
    <service_mode>controller</service_mode>
    <control_service_url>http://127.0.0.1:8000</control_service_url>
</cluster>
```

### 6.3 调试模式

```bash
# 开启调试日志
xdevice run acts -l <module> --log-level DEBUG
```

---

## 相关文档

- [配置说明](05_Configuration.md)
- [架构说明](02_Architecture.md)
- [故障排查](08_Troubleshooting.md)
