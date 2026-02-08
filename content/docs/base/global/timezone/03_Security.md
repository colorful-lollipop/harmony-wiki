# 安全评审

## 威胁模型概述

本模块为**时区数据管理工具**，主要涉及以下数据流：

```
外部输入          内部处理          系统存储
    │                │                │
    ▼                ▼                ▼
┌─────────┐    ┌─────────────┐    ┌─────────────┐
│  IANA   │───▶│ download_    │───▶│   data/     │
│ Database│    │ iana.py      │    │   iana/     │
└─────────┘    └─────────────┘    └─────────────┘
                     │
                     ▼
              ┌─────────────┐    ┌─────────────┐
              │ compile.sh  │───▶│  prebuild/  │
              └─────────────┘    └─────────────┘
                                   │
                                   ▼
                            ┌─────────────┐
                            │ 系统部署     │
                            │ /etc/icu_   │
                            │ tzdata/     │
                            └─────────────┘
```

## 攻击面分析

| 攻击面 | 类型 | 风险等级 | 说明 |
|--------|------|----------|------|
| 网络下载 | 网络 | 中 | 从外部服务器下载数据 |
| 文件解压 | 文件 | 中 | 解压缩远程获取的 tar.gz 文件 |
| 文件写入 | 文件 | 中 | 写入时区数据到本地目录 |
| 版本号更新 | 文件 | 低 | 修改 version.txt |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      不可信区域                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           IANA Time Zone Database                   │   │
│  │           (data.iana.org)                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                                │
│                            ▼ HTTP/HTTPS                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              download_iana.py                       │   │
│  │              (下载与验证)                            │   │
│  └────────────────────┬────────────────────────────┘   │
│                       │                                    │
└───────────────────────┼────────────────────────────────────┘
                        │ 验证后数据
                        ▼
┌─────────────────────────────────────────────────────────────┐
│                      可信区域                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              compile.sh                              │   │
│  │              (编译生成)                               │   │
│  └────────────────────┬────────────────────────────┘   │
│                       │                                    │
│                       ▼                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              系统部署路径                             │   │
│  │              /etc/icu_tzdata/                        │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 可被利用点与修复建议

### 1. HTTP 错误处理不完整

**证据**: `tool/update_tool/download_iana.py:46-47`

```python
except error.HTTPError as http_error:
    return -1
```

**问题描述**:
- 仅捕获 `HTTPError`，可能遗漏其他网络错误
- 错误发生时静默返回，无详细日志
- 无重试机制

**可利用路径**:
1. 网络波动导致下载超时
2. 临时网络故障导致下载失败
3. DNS 劫持导致连接到恶意服务器

**影响**:
- 下载失败时无法区分具体原因
- 可能导致版本不一致

**修复建议**:
```python
except error.HTTPError as http_error:
    print(f'HTTP Error {http_error.code}: {http_error.reason}')
    # 添加重试逻辑
    for retry in range(3):
        time.sleep(2 ** retry)  # 指数退避
        # 重试下载
    return -1
except error.URLError as url_error:
    print(f'URL Error: {url_error.reason}')
    return -1
except ssl.SSLError as ssl_error:
    print(f'SSL Error: {ssl_error.reason}')
    return -1
```

**风险等级**: 低

---

### 2. SSL 证书验证配置不明确

**证据**: `tool/update_tool/download_iana.py:42`

```python
context = ssl.SSLContext()
```

**问题描述**:
- 创建 SSLContext 但未指定版本和验证级别
- 默认行为可能随 Python 版本变化

**可利用路径**:
1. 中间人攻击 (MITM)
2. 证书验证绕过

**影响**:
- 可能下载到被篡改的时区数据

**修复建议**:
```python
# 明确启用证书验证
context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)
context.check_hostname = True
context.verify_mode = ssl.CERT_REQUIRED
context.load_default_certs()
```

**风险等级**: 中

---

### 3. 路径遍历风险

**证据**: `tool/update_tool/download_iana.py:86`

```python
tar.extractall(save_path)
```

**问题描述**:
- 使用 `extractall()` 解压任意文件
- 恶意 tar 文件可能包含路径遍历文件

**可利用路径**:
1. 攻击者控制 IANA 服务器
2. 恶意 tar 文件覆盖系统文件

**影响**:
- 可能覆盖系统关键文件
- 权限提升风险

**修复建议**:
```python
def safe_extractall(tar, path):
    for member in tar.getmembers():
        member_path = os.path.join(path, member.name)
        if not member_path.startswith(os.path.abspath(path) + os.sep):
            raise Exception(f'Path traversal attempt: {member.name}')
        tar.extract(member, path)

safe_extractall(tar, save_path)
```

**风险等级**: 中

---

### 4. Shell 命令注入风险

**证据**: `tool/compile_tool/compile.sh:20-21`

```bash
make -C ${iana_path}
mv ${iana_path}/zic ${zic_path}
```

**问题描述**:
- 使用变量拼接命令
- 路径变量可能被恶意修改

**可利用路径**:
1. 环境变量被恶意设置
2. 符号链接攻击

**影响**:
- 可能执行意外命令
- 文件被移动到错误位置

**修复建议**:
```bash
# 使用引号包裹变量
make -C "${iana_path}"
mv "${iana_path}/zic" "${zic_path}"

# 添加路径验证
if [[ ! "${iana_path}" =~ ^/ ]]; then
    echo "Error: Invalid path"
    exit 1
fi
```

**风险等级**: 低

---

### 5. 版本文件竞争条件

**证据**: `tool/update_tool/download_iana.py:114`

```python
with os.fdopen(os.open(os.path.join(download_path, 'version.txt'), flags, modes), 'w') as file:
    file.write(new_version + '\n')
```

**问题描述**:
- 使用 `O_WRONLY | O_CREAT` 而非 `O_EXCL`
- 并发执行可能导致版本文件损坏

**可利用路径**:
1. 多个下载进程同时运行
2. 中途终止导致版本不一致

**影响**:
- 版本文件损坏
- 时区数据版本不一致

**修复建议**:
```python
flags = os.O_WRONLY | os.O_CREAT | os.O_EXCL  # 避免覆盖
# 或使用文件锁
import fcntl
with open(version_file, 'a') as f:
    fcntl.flock(f, fcntl.LOCK_EX)
    f.write(new_version + '\n')
    fcntl.flock(f, fcntl.LOCK_UN)
```

**风险等级**: 低

---

### 6. 文件权限过宽

**证据**: `tool/update_tool/download_iana.py:31-32`

```python
modes = stat.S_IWUSR | stat.S_IRUSR
```

**问题描述**:
- 新建文件权限为用户读写
- 建议限制其他用户访问

**可利用路径**:
1. 本地其他用户读取时区数据
2. 修改版本文件

**影响**:
- 信息泄露
- 数据篡改

**修复建议**:
```python
modes = stat.S_IWUSR | stat.S_IRUSR | stat.S_IRGRP  # 允许组读取
# 或
modes = stat.S_IWUSR | stat.S_IRUSR  # 保持当前设置，但确保目录权限正确
```

**风险等级**: 低

---

### 7. Make 编译信任风险

**证据**: `tool/compile_tool/compile.sh:20`

```bash
make -C ${iana_path}
```

**问题描述**:
- 执行外部 Makefile
- Makefile 可能包含恶意规则

**可利用路径**:
1. IANA 源代码中的 Makefile 被篡改
2. 恶意 Makefile 规则执行任意命令

**影响**:
- 执行恶意编译命令
- 编译产物被篡改

**修复建议**:
- 验证 Makefile 内容后再执行
- 使用受限的 Make 选项
- 考虑用更安全的方式编译 zic

**风险等级**: 中

---

## 风险等级汇总

| 编号 | 风险项 | 等级 | 状态 |
|------|--------|------|------|
| 1 | HTTP 错误处理不完整 | 低 | 建议修复 |
| 2 | SSL 证书验证配置不明确 | 中 | 建议修复 |
| 3 | 路径遍历风险 | 中 | 建议修复 |
| 4 | Shell 命令注入风险 | 低 | 建议修复 |
| 5 | 版本文件竞争条件 | 低 | 建议修复 |
| 6 | 文件权限过宽 | 低 | 建议修复 |
| 7 | Make 编译信任风险 | 中 | 建议修复 |

## 安全最佳实践建议

### 网络安全
1. **强制 HTTPS**: 确保所有下载使用 HTTPS
2. **证书验证**: 启用 SSL 证书验证
3. **完整性校验**: 下载后校验 SHA256 或 GPG 签名
4. **限速/限流**: 防止 DoS 攻击

### 文件安全
1. **路径验证**: 拒绝路径遍历尝试
2. **最小权限**: 文件权限遵循最小权限原则
3. **原子操作**: 使用 `O_EXCL` 避免竞争条件
4. **沙箱环境**: 在隔离环境中解压

### 运行安全
1. **输入验证**: 验证所有外部输入
2. **日志记录**: 记录关键操作便于审计
3. **错误处理**: 提供有意义的错误信息
4. **回滚机制**: 失败时恢复到安全状态

## 相关文档

- [架构说明](./01_Architecture.md)
- [构建指南](./02_Build.md)
- [使用指南](./04_Usage.md)
