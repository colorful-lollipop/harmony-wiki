# 故障排查

## 1. 常见问题

### 1.1 策略加载失败

**问题**: 系统启动时 LoadPolicy() 返回失败

**可能原因**:

| 原因 | 排查方法 | 解决方案 |
|------|----------|----------|
| policy.31 文件不存在 | 检查 `/etc/selinux/targeted/policy/policy.31` | 重新编译策略 |
| policy.31 格式错误 | 使用 `checkpolicy` 手动检查 | 修复策略语法 |
| 权限不足 | 检查文件权限 | chmod 644 |
| selinuxfs 未挂载 | 检查 `/sys/fs/selinux` | 确保内核支持 |

**日志查看**:

```bash
# 查看内核日志
dmesg | grep -i selinux

# 查看 hilog
hilog | grep -i selinux
```

### 1.2 AVC 拒绝访问

**问题**: 正常功能被 SELinux 拒绝

**日志示例**:

```
audit: type=1400 audit(1502458430.566:4): avc: denied { open } 
  for pid=1658 comm="setenforce" path="/sys/fs/selinux/enforce"
  scontext=u:r:hdcd:s0 tcontext=u:object_r:selinuxfs:s0 tclass=file permissive=1
```

**字段说明** (README.md:66-78):

| 字段 | 含义 | 示例值 |
|------|------|--------|
| { operation } | 被拒绝的操作 | { open } |
| pid | 访问进程 ID | 1658 |
| comm | 进程名 | setenforce |
| path | 目标路径 | /sys/fs/selinux/enforce |
| scontext | 源安全上下文 | u:r:hdcd:s0 |
| tcontext | 目标安全上下文 | u:object_r:selinuxfs:s0 |
| tclass | 目标类型 | file |
| permissive | 宽容模式标志 | 1 (0=拦截) |

**解决步骤**:

1. 确认 permissive 模式
2. 分析访问需求
3. 添加对应的 allow 规则
4. 重新编译策略

**示例**:

```te
# 允许 hdcd 访问 selinuxfs 的文件
allow hdcd selinuxfs:file open;
```

### 1.3 restorecon 失败

**问题**: Restorecon() 返回错误

**可能原因**:

| 原因 | 排查方法 |
|------|----------|
| 路径不存在 | 检查路径是否有效 |
| 权限不足 | 检查文件权限 |
| 文件系统不支持 | 确认文件系统类型 |
| 符号链接循环 | 检查链接 |

**调试方法**:

```bash
# 详细输出
restorecon -v <path>

# 递归恢复
restorecon -R <path>

# 强制恢复（跳过 skipacl 检查）
restorecon -F <path>
```

### 1.4 参数检查失败

**问题**: SetParamCheck() 返回拒绝

**排查步骤**:

1. 检查参数是否存在
2. 确认调用者上下文
3. 验证参数权限配置

**代码调试**:

```c
// 开启详细日志
SetInitSelinuxLog();

// 检查参数
const char *label = GetParamLabel("sys.test");
if (label == NULL) {
    // 参数未配置
}
```

### 1.5 服务检查失败

**问题**: ServiceChecker 返回拒绝

**排查步骤**:

1. 检查服务是否已注册
2. 确认调用者权限
3. 验证 service_contexts 配置

### 1.6 HAP 上下文恢复失败

**问题**: HapFileRestorecon() 返回错误

**日志查看**:

```bash
# 查看 HAP restorecon 日志
hilog | grep -i hap_restorecon
```

**常见原因**:

| 原因 | 解决方案 |
|------|----------|
| JSON 配置格式错误 | 检查 sehap_contexts 文件 |
| 权限不足 | 检查文件权限 |
| 路径不存在 | 验证应用数据路径 |

## 2. 调试工具

### 2.1 命令行工具

| 工具 | 路径 | 用途 |
|------|------|------|
| getenforce | system/bin | 获取 SELinux 模式 |
| setenforce | system/bin | 设置 SELinux 模式 |
| load_policy | system/bin | 加载策略 |
| restorecon | system/bin | 恢复文件标签 |

### 2.2 查看标签

```bash
# 查看文件标签
ls -lZ <path>

# 查看进程标签
ps -eZ

# 查看链接源文件标签
ls -lLZ <path>

# 查看单个文件标签
getfilecon <path>
```

### 2.3 设置标签

```bash
# 设置文件标签
setfilecon <context> <path>

# 强制设置
setfilecon -f <path> <context>
```

### 2.4 AVC 日志分析

```bash
# 实时查看 AVC 日志
cat /proc/kmsg | grep avc

# 或使用 auditd
auditctl -D  # 删除所有规则
auditctl -a always,exit -F arch=b64 -S open -F pid=<pid>
```

## 3. 日志解读

### 3.1 日志来源

| 日志类型 | 位置 | 说明 |
|----------|------|------|
| 内核日志 | /proc/kmsg | SELinux 决策日志 |
| hilog | hilog | 用户空间日志 |
| audit | audit subsystem | 审计日志 |

### 3.2 日志级别

| 级别 | 说明 |
|------|------|
| ERROR | 错误 |
| WARN | 警告 |
| INFO | 信息 |
| DEBUG | 调试 |

### 3.3 hilog 过滤

```bash
# 过滤 SELinux 相关日志
hilog | grep -E "SELinux|selinux|restorecon|policy"

# 过滤 AVC 日志
hilog | grep "avc:"
```

## 4. 开发调试技巧

### 4.1 临时 permissive 模式

```bash
# 设置宽容模式（重启后失效）
setenforce 0

# 验证模式
getenforce
# 输出: Permissive
```

### 4.2 单个域 permissive

```bash
# 设置特定域为 permissive
semodule -D
semodule -B
# 或修改策略添加：
permissive <domain_name>;
```

### 4.3 策略增量加载

```bash
# 不重启加载策略
load_policy -c /etc/selinux/targeted/policy/policy.31
```

### 4.4 本地策略测试

```bash
# 1. 生成本地策略模块
checkmodule -M -m -o local.mod local.te

# 2. 打包模块
semodule_package -o local.pp -m local.mod

# 3. 安装模块
semodule -i local.pp

# 4. 启用模块
semodule -e local
```

## 5. 编译问题

### 5.1 策略编译失败

**问题**: build_policy.py 执行失败

**排查**:

```bash
# 1. 检查 Python 依赖
python3 scripts/build_policy.py --help

# 2. 检查策略文件语法
checkpolicy -c 31 -o /dev/null sepolicy/*.te

# 3. 查看详细错误
python3 -v scripts/build_policy.py
```

### 5.2 上下文编译失败

**问题**: build_contexts.py 执行失败

**排查**:

```bash
# 1. 检查 policy.31 是否存在
ls -la out/policy.31

# 2. 手动编译测试
sefcontext_compile -o file_contexts.bin file_contexts
```

### 5.3 依赖缺失

**问题**: GN 编译提示缺少依赖

**解决**:

```bash
# 1. 检查组件定义
./build.sh --product-name=rk3568 --info

# 2. 确认依赖已编译
hb build -p <dependency_part>
```

## 6. 性能问题

### 6.1 restorecon 慢

**优化方法**:

```c
// 使用并行恢复
RestoreconRecurseParallel(path, 8);  // 8 线程
```

**建议**:

| 场景 | 建议线程数 |
|------|------------|
| SSD | CPU 核心数 |
| HDD | 4-8 |
| NVMe | 最大 |

### 6.2 策略加载慢

**优化方法**:

1. 使用预编译的 policy.31
2. 减少策略复杂度
3. 使用 policydb 优化工具

## 7. 常见错误码

### 7.1 libselinux 错误码

| 错误码 | 含义 |
|--------|------|
| 0 | 成功 |
| -1 | 通用错误 |
| -2 | 无效参数 |
| -ENOENT | 文件不存在 |
| -EACCES | 权限拒绝 |

### 7.2 AVC 拒绝码

| 拒绝原因 | 说明 |
|----------|------|
| avc: denied | 访问被拒绝 |
| avc: granted | 访问被允许 |

## 8. 相关文档

| 文档 | 链接 |
|------|------|
| 项目概述 | [01_Overview.md](01_Overview.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| API 接口 | [03_API.md](03_API.md) |
| 构建系统 | [04_Build.md](04_Build.md) |
| 安全评审 | [05_Security.md](05_Security.md) |
