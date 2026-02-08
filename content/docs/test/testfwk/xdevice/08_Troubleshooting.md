# XDevice 故障排查

## 1. 安装问题

### 问题 1：Python 版本不兼容

**错误信息**：
```
AttributeError: module 'sys' has no attribute 'version_info'
```

**解决方法**：
```bash
# 检查 Python 版本
python --version

# 确保 Python >= 3.7.5
python3 --version
```

---

### 问题 2：依赖安装失败

**错误信息**：
```
ERROR: Could not find a version that satisfies the requirement pyserial
```

**解决方法**：
```bash
# 更新 pip
pip install --upgrade pip

# 手动安装依赖
pip install pyserial>=3.3
pip install paramiko>=2.7.1
pip install rsa>=4.0
```

---

## 2. 配置问题

### 问题 3：配置文件不存在

**错误信息**：
```
Code_0103001: user_config.xml does not exist
```

**解决方法**：
```bash
# 确保配置文件存在
ls -la user_config.xml

# 从模板复制
cp config/user_config.xml .
```

---

### 问题 4：XML 解析失败

**错误信息**：
```
Code_0103002: Parsing the user_config.xml failed
```

**解决方法**：
```bash
# 验证 XML 格式
python -c "import xml.etree.ElementTree as ET; ET.parse('user_config.xml')"

# 检查特殊字符
```

---

### 问题 5：设备别名重复

**错误信息**：
```
Code_0103004: Find duplicate sn config, configuration incorrect
```

**解决方法**：
```xml
<!-- user_config.xml 中确保设备别名唯一 -->
<device type="usb-hdc" label="ohos">
    <info alias="device1" .../>
</device>
<device type="usb-hdc" label="ohos">
    <info alias="device2" .../>  <!-- 避免重复别名 -->
</device>
```

---

## 3. 设备连接问题

### 问题 6：设备未识别

**错误信息**：
```
No device found matching criteria
```

**解决方法**：
```bash
# 1. 检查 HDC 是否可用
hdc list targets -v

# 2. 检查设备状态
xdevice list

# 3. 重启 HDC 服务
hdc kill
hdc start
```

---

### 问题 7：串口连接失败

**错误信息**：
```
SerialException: could not open port COM20
```

**解决方法**：
```bash
# 1. 检查串口是否被占用
# Windows
mode

# 2. 检查串口权限
# Linux
ls -la /dev/tty*

# 3. 确认串口正确
dmesg | grep tty
```

---

### 问题 8：SSH 连接失败

**错误信息**：
```
Authentication failed for NFS server
```

**解决方法**：
```bash
# 1. 检查 SSH 密钥
ssh -i /path/to/key user@host

# 2. 验证凭据
# 确保 user_config.xml 中密码正确

# 3. 检查服务器可达性
ping <nfs_ip>
```

---

## 4. 执行问题

### 问题 9：测试用例未发现

**错误信息**：
```
No test case found
```

**解决方法**：
```bash
# 1. 检查用例路径
xdevice run acts -tcpath ./testcases -l <module>

# 2. 验证 JSON 配置
python -c "import json; json.load(open('acts.json'))"

# 3. 检查用例格式
```

---

### 问题 10：驱动执行失败

**错误信息**：
```
Driver execution failed: CppTestDriver not found
```

**解决方法**：
```bash
# 1. 检查 ohos 插件是否安装
pip list | grep xdevice-ohos

# 2. 重新安装插件
pip install -e plugins/ohos/

# 3. 检查驱动类型
xdevice run acts -l <module> --dry-run
```

---

### 问题 11：并发数设置无效

**错误信息**：
```
Max driver threads limit exceeded
```

**解决方法**：
```bash
# 检查配置文件中的并发设置
# user_config.xml 中确保配置正确

# 使用命令行参数覆盖
xdevice run acts -l <module> --max-driver-threads 5
```

---

## 5. 报告问题

### 问题 12：报告生成失败

**错误信息**：
```
Report generation failed
```

**解决方法**：
```bash
# 1. 检查报告路径权限
mkdir -p reports
chmod 755 reports

# 2. 检查磁盘空间
df -h

# 3. 清理旧报告
rm -rf reports/*
```

---

### 问题 13：报告加密密钥丢失

**错误信息**：
```
Decryption failed: Private key not found
```

**解决方法**：
```bash
# 1. 检查密钥文件
ls -la config/

# 2. 重新生成密钥
python -c "from xdevice._core.report.encrypt import generate_key_file; generate_key_file()"

# 3. 注意：重新生成密钥后旧报告将无法解密
```

---

## 6. 集群问题

### 问题 14：集群控制器启动失败

**错误信息**：
```
FastAPI server failed to start
```

**解决方法**：
```bash
# 1. 检查端口是否被占用
lsof -i :8000

# 2. 检查 Python 版本 (需要 3.10+)
python --version

# 3. 安装可选依赖
pip install fastapi uvicorn sqlmodel
```

---

### 问题 15：Worker 无法连接

**错误信息**：
```
Worker connection timeout
```

**解决方法**：
```bash
# 1. 检查控制器 URL
# 确保 user_config.xml 中 control_service_url 正确

# 2. 检查网络连通性
curl http://127.0.0.1:8000/health

# 3. 检查防火墙设置
```

---

## 7. 调试技巧

### 7.1 启用详细日志

```bash
# DEBUG 级别日志
xdevice run acts -l <module> --log-level DEBUG
```

### 7.2 干运行模式

```bash
# 不实际执行，仅显示计划
xdevice run acts -l <module> --dry-run
```

### 7.3 检查设备日志

```bash
# 查看设备日志
xdevice list
# 在 reports/log/ 目录下查找 device_*.log
```

---

## 常用检查命令

| 命令 | 用途 |
|------|------|
| `xdevice --help` | 查看帮助 |
| `xdevice list` | 列出设备 |
| `xdevice help run` | 查看 run 命令帮助 |
| `hdc list targets -v` | 列出 HDC 设备 |
| `pip list \| grep xdevice` | 检查安装 |

---

## 相关文档

- [使用指南](07_Usage.md)
- [配置说明](05_Configuration.md)
- [安全评审](06_Security.md)
