# 安全风险评审

本文档分析 OpenHarmony build 仓库的安全风险，包括攻击面、可被利用点和修复建议。

## 评估范围

**检查范围**:
- 构建脚本 (`build_scripts/`)
- hb 构建工具 (`hb/`)
- 工具脚本 (`scripts/`)
- 打包脚本 (`ohos/`)
- 配置文件 (`.gni`, `.json`)

**未覆盖范围**:
- 测试代码 (`test/`)
- 第三方库

## 攻击面分析

### 1. 输入验证攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|---------|------|
| 命令行参数 | 中 | 用户传入的构建参数 |
| 配置文件解析 | 中 | bundle.json, config.json |
| 路径处理 | 高 | 文件系统路径拼接 |
| 环境变量 | 中 | PATH, LD_LIBRARY_PATH 等 |

### 2. 代码执行攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|---------|------|
| Python 脚本执行 | 高 | 构建过程中执行大量 Python 脚本 |
| Shell 命令执行 | 高 | build.sh, env_setup.sh |
| GN 代码生成 | 中 | GN 模板展开 |
| Ninja 构建执行 | 中 | 编译器调用 |

### 3. 文件系统攻击面

| 攻击面 | 风险等级 | 说明 |
|--------|---------|------|
| 文件读写 | 高 | 构建过程读写大量文件 |
| 目录遍历 | 高 | 路径拼接可能导致越界访问 |
| 符号链接 | 中 | 符号链接劫持 |
| 临时文件 | 中 | 临时文件安全 |

## 可被利用点

### 风险 1: 路径遍历漏洞

**证据**:
```python
# build_scripts/build.py:83
python_relative_dir = gn_helpers.ReadFile(...)
# 未验证路径内容直接拼接
```

**触发路径**:
1. 攻击者控制 `.gn` 文件内容
2. `build.py` 读取并解析
3. 路径拼接后执行

**影响**: 可能导致任意文件读取或执行

**修复建议**:
```python
# 建议添加路径验证
import os
if not os.path.normpath(path).startswith(allowed_base):
    raise ValueError("Path traversal detected")
```

### 风险 2: 命令注入漏洞

**证据**:
```python
# hb/services/gn.py:102-133
# GN 参数直接拼接执行
gn_gen_cmd = [self.exec, 'gen', '--args={}'.format(' '.join(self._convert_args()))]
```

**触发路径**:
1. 用户传入 `--gn-args` 参数
2. 参数未充分过滤直接传递给 GN
3. 可能导致命令注入

**影响**: 执行任意系统命令

**修复建议**:
```python
# 使用列表传递参数，避免 shell 解释
subprocess.run(cmd_list, shell=False)
# 对参数进行白名单验证
```

### 风险 3: Python 代码执行

**证据**:
```python
# scripts/ 目录下大量脚本使用 exec/eval
# gn_helpers.py 中的 exec_script 调用
```

**触发路径**:
```gn
# BUILD.gn 中可执行任意 Python 脚本
result = exec_script("my_script.py", [arg1, arg2], "value")
```

**影响**: 执行任意 Python 代码

**修复建议**:
- 限制 `exec_script` 只能执行白名单内的脚本
- 脚本路径使用绝对路径
- 验证脚本签名

### 风险 4: 敏感信息泄露

**证据**:
```gni
# ohos_var.gni:392-404
default_key_alias = "OpenHarmony Application Release"
default_signature_algorithm = "SHA256withECDSA"
default_keystore_path = "//developtools/hapsigner/dist/OpenHarmony.p12"
default_hap_private_key_path = "123456"
default_keystore_password = "123456"
```

**影响**: 默认密码和密钥路径暴露

**修复建议**:
- 移除硬编码密码
- 使用环境变量或安全存储
- 强制用户设置自己的密钥

### 风险 5: 不安全的临时文件

**证据**:
```python
# scripts/ 目录多处使用临时文件
# 未使用 mkstemp 等安全 API
```

**影响**: 临时文件可能被预测或竞争条件攻击

**修复建议**:
```python
import tempfile
# 使用安全的临时文件创建方式
with tempfile.NamedTemporaryFile(mode='w', delete=False) as f:
    f.write(data)
```

### 风险 6: 环境变量注入

**证据**:
```python
# hb/services/ninja.py:41-76
# 环境变量直接传递给 Ninja
env = os.environ.copy()
# 过滤逻辑可能不完整
```

**触发路径**:
1. 攻击者设置恶意环境变量
2. 构建过程读取并传递
3. 影响编译器行为

**影响**: 可能导致编译器执行恶意操作

**修复建议**:
- 使用白名单机制过滤环境变量
- 验证环境变量值

### 风险 7: 文件权限问题

**证据**:
```python
# scripts/copy_ex.py
# 文件复制未保留或正确设置权限
```

**影响**: 敏感文件可能被其他用户读取

**修复建议**:
- 明确设置文件权限
- 使用 umask 限制默认权限

### 风险 8: 依赖混淆攻击

**证据**:
```python
# hb/services/preloader.py
# 从多个源加载 Python 模块
```

**影响**: 可能加载恶意模块

**修复建议**:
- 验证模块路径
- 使用绝对路径导入
- 模块签名验证

### 风险 9: 不安全的反序列化

**证据**:
```python
# 多处使用 json.load 解析用户输入
# 未验证 JSON 内容
```

**影响**: 可能导致拒绝服务或代码执行

**修复建议**:
```python
import json
# 限制解析深度
data = json.load(f, parse_constant=lambda x: None)
```

### 风险 10: 日志注入

**证据**:
```python
# hb/util/log_util.py
# 日志直接记录用户输入
```

**影响**: 日志伪造、日志文件污染

**修复建议**:
- 对日志内容进行过滤
- 使用结构化日志

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│  不信任区域                                                   │
│  ├── 用户输入 (命令行参数、配置文件)                          │
│  ├── 环境变量                                                │
│  ├── 外部仓库代码                                            │
│  └── 网络资源                                                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (验证/过滤)
┌─────────────────────────────────────────────────────────────┐
│  半信任区域                                                  │
│  ├── 构建脚本 (build.sh, build.py)                          │
│  ├── hb 工具                                                 │
│  └── GN 配置                                                 │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼ (沙箱/限制)
┌─────────────────────────────────────────────────────────────┐
│  信任区域                                                    │
│  ├── 编译器 (clang)                                          │
│  ├── 系统库                                                  │
│  └── 内核                                                    │
└─────────────────────────────────────────────────────────────┘
```

## 修复优先级

| 优先级 | 风险 | 建议修复时间 |
|--------|------|-------------|
| P0 | 路径遍历、命令注入 | 立即 |
| P1 | Python 代码执行、敏感信息泄露 | 1周内 |
| P2 | 临时文件、环境变量注入 | 1月内 |
| P3 | 文件权限、依赖混淆 | 3月内 |

## 安全最佳实践

### 对于开发者

1. **输入验证**
   - 所有用户输入都需要验证
   - 使用白名单而非黑名单
   - 验证路径、文件名、参数

2. **安全编码**
   - 避免使用 `eval()`, `exec()`
   - 使用 `subprocess.run()` 而非 `os.system()`
   - 使用安全的临时文件 API

3. **权限控制**
   - 最小权限原则
   - 敏感文件设置正确权限
   - 避免以 root 运行构建

4. **审计日志**
   - 记录关键操作
   - 监控异常行为
   - 定期审计构建日志

### 对于用户

1. **环境安全**
   - 在隔离环境构建
   - 使用容器或虚拟机
   - 定期更新构建工具

2. **源码验证**
   - 验证源码完整性
   - 使用可信的源码镜像
   - 检查源码签名

3. **构建隔离**
   - 使用 `chroot` 或容器
   - 限制网络访问
   - 监控文件系统访问

## 相关配置

### 安全相关 GN 参数

```gn
# 启用安全编译选项
is_asan = true          # Address Sanitizer
is_ubsan = true         # Undefined Behavior Sanitizer
is_cfi = true           # Control Flow Integrity

# 安全标志
enforce_selinux = true  # 强制 SELinux
```

### 安全编译标志

```gn
# //build/config/compiler/BUILD.gn
# 已启用的安全标志
cflags += [ "-fstack-protector-strong" ]  # 栈保护
ldflags += [ "-Wl,-z,relro" ]             # RELRO
ldflags += [ "-Wl,-z,now" ]               # 立即绑定
ldflags += [ "-Wl,-z,noexecstack" ]       # 不可执行栈
```

---

*文档生成时间: 2025-02-06*

**免责声明**: 本安全评审基于代码静态分析，实际风险可能需要进一步验证。
