# 问题定位 (Troubleshooting)

> bundle_tool 常见构建、运行、调试问题与解决方案

## 构建问题

### 问题 1: GN 依赖缺失

**错误信息**:
```
error: dependencies not found: xxx
```

**原因**: 缺少必要的子系统或组件依赖

**解决方案**:
```bash
# 检查子系统配置
hb set

# 确保相关子系统已包含
# - ability_runtime
# - bundle_framework
# - ipc
# - samgr
```

**证据来源**: `bundle.json:19-41`

---

### 问题 2: 编译链接失败

**错误信息**:
```
undefined reference to xxx
```

**原因**: 缺少 external_deps 或 public_external_deps

**解决方案**:
检查 `frameworks/BUILD.gn` 中是否包含必要依赖:
```gn
external_deps = [
    "bundle_framework:appexecfwk_base",
    "bundle_framework:appexecfwk_core",
    "ipc:ipc_core",
    # ... 其他依赖
]
```

---

### 问题 3: Sanitizer 编译错误

**错误信息**:
```
sanitizer: unknown option '-fsanitize=cfi'
```

**原因**: 目标平台不支持某些 sanitizer 选项

**解决方案**:
在 `frameworks/BUILD.gn` 中条件编译:
```gn
if (target_cpu != "arm") {
  sanitize = {
    cfi = true
    # ...
  }
}
```

**证据来源**: `frameworks/BUILD.gn:32-39`

---

## 运行问题

### 问题 4: 命令无法识别

**错误信息**:
```
error: unknown option xxx
```

**原因**: 参数拼写错误或不支持的选项

**解决方案**:
```bash
# 查看帮助
bm help

# 或查看具体命令帮助
bm install -h
```

**证据来源**: `bundle_command.h:28-205`

---

### 问题 5: 权限不足

**错误信息**:
```
error: permission denied
```

**原因**: 缺少必要的系统权限

**解决方案**:
```bash
# 确保在 root 版本运行
# 或检查权限配置

# enable/disable 需要 root
bm enable -n com.example.app  # 需要 root

# clean 在 user 版本需要开发者模式
bm clean -c -n com.example.app
```

---

### 问题 6: 安装失败

**错误信息**:
```
error: failed to install bundle.
```

**可能原因**:
1. HAP 文件损坏
2. 签名验证失败
3. 权限不足
4. 磁盘空间不足

**排查步骤**:
```bash
# 1. 检查 HAP 文件
ls -la /path/to/app.hap

# 2. 检查签名
# bundle_tool 不直接显示签名信息
# 通过 BundleManagerService 日志

# 3. 检查磁盘空间
df -h

# 4. 查看日志
hilog | grep BMSTool
```

---

### 问题 7: 卸载失败

**错误信息**:
```
error: failed to uninstall bundle.
```

**可能原因**:
1. 包名不存在
2. 应用正在运行
3. 系统应用禁止卸载

**排查步骤**:
```bash
# 1. 检查包是否存在
bm dump -n com.example.app

# 2. 强制停止应用后卸载
# 通过其他工具停止应用

# 3. 检查是否为系统应用
# 系统应用可能无法卸载
```

---

## 调试方法

### 方法 1: 日志查看

**日志标签**:
```
APP_LOG_TAG = "BMSTool"
LOG_DOMAIN = 0xD001123
```

**查看日志**:
```bash
# 过滤 bundle_tool 日志
hilog | grep BMSTool

# 或查看所有日志级别
hilog | grep -E "BMSTool|BundleManager"
```

**证据来源**: `BUILD.gn:24-25`

---

### 方法 2: 调试日志

**启用调试**:
```bash
# 查看详细输出
bm install -p /path/to/app.hap 2>&1 | verbose
```

---

### 方法 3: 返回码分析

**错误码映射**:
```cpp
// bundle_command.h:340
ErrCode TransformErrCode(const int32_t resultCode);
```

**常见错误码**:
| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 非 0 | 失败（见具体消息） |

**证据来源**: `bundle_command.h:340`

---

## 常见场景

### 场景 1: 安装 HAP 失败

**问题**: `bm install -p app.hap` 失败

**排查步骤**:
```bash
# 1. 检查文件存在
ls -la app.hap

# 2. 检查 HAP 格式
file app.hap

# 3. 检查签名
# 需要查看 BundleManagerService 日志

# 4. 尝试覆盖安装
bm install -p app.hap -r
```

---

### 场景 2: 查询不到应用

**问题**: `bm dump -n app` 返回空

**排查步骤**:
```bash
# 1. 检查包名是否正确
bm dump -a

# 2. 检查用户 ID
# 可能在其他用户下
bm dump -n app -u 0

# 3. 检查应用是否已安装
bm dump -a | grep app
```

---

### 场景 3: 清理缓存失败

**问题**: `bm clean -c -n app` 失败

**排查步骤**:
```bash
# 1. 检查权限
# user 版本需要开发者模式
bm clean -c -n app

# 2. 检查应用是否存在
bm dump -n app

# 3. 检查应用索引
# 分身应用可能需要指定 -i
bm clean -c -n app -i 0
```

---

### 场景 4: 快速修复失败

**问题**: `bm quickfix -a -f patch.hqf` 失败

**排查步骤**:
```bash
# 1. 检查 HQF 文件
ls -la patch.hqf

# 2. 查询现有补丁
bm quickfix -q -b com.example.app

# 3. 查看调试信息
bm quickfix -a -f patch.hqf -d
```

---

## 问题报告模板

当报告问题时，请包含以下信息:

```markdown
## 问题描述
[简要描述问题]

## 复现步骤
1. [步骤1]
2. [步骤2]
3. [...]

## 实际结果
[描述实际发生的情况]

## 期望结果
[描述期望发生的情况]

## 环境信息
- 设备型号: [设备信息]
- 系统版本: [OpenHarmony 版本]
- bm 版本: [版本信息]

## 日志
```
[粘贴相关日志]
```

## 其他信息
[任何其他有用信息]
```

---

## 相关文档

- [02_Command_Reference.md](./02_Command_Reference.md) - 命令参考
- [05_Security.md](./05_Security.md) - 安全评审
