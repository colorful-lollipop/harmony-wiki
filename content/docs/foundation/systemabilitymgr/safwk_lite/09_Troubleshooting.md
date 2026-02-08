# 故障排查

## 编译问题

### 问题 1：feature 开关未生效

**现象**:
```bash
error: undefined reference to 'xxx'
```

**原因**:
对应的 feature 开关未启用，导致依赖未包含。

**解决**:
```bash
# 在构建命令中启用 feature
hb build safwk_lite -D safwk_lite_feature_enable_dtbschedmgr=true
```

或在 `productdefine/common/products/rk3568.json` 中添加：
```json
{
  "board_hdf_command": [
    "--Cflags=safwk_lite_feature_enable_dtbschedmgr=true"
  ]
}
```

---

### 问题 2：samgr_lite 依赖缺失

**现象**:
```bash
error: 'samgr_lite.h' file not found
```

**原因**:
samgr_lite 子系统未正确拉取或 include 路径错误。

**解决**:
```bash
# 1. 拉取 samgr_lite 依赖
repo sync systemabilitymgr/samgr_lite

# 2. 清理并重新构建
hb clean
hb build safwk_lite
```

---

### 问题 3：hilog 链接失败

**现象**:
```bash
error: cannot find -lhilog_shared
```

**原因**:
hilog_lite 子模块未编译。

**解决**:
```bash
# 确保 hilog_lite 被编译
hb build hilog_lite
hb build safwk_lite
```

---

## 运行问题

### 问题 4：foundation 进程启动失败

**现象**:
```bash
# 进程启动后立即退出
$ ./foundation
Segmentation fault
```

**排查步骤**:

1. 检查日志
```bash
# 启用调试日志
DEBUG_SERVICES_SAFWK_LITE=true ./foundation
```

2. 检查 Samgr 初始化
```bash
# 使用 gdb 调试
gdb ./foundation
(gdb) break SAMGR_Bootstrap
(gdb) run
```

3. 检查依赖库
```bash
# 查看缺失库
ldd ./foundation
```

---

### 问题 5：服务未注册

**现象**:
```
[Samgr] Service XXX not found
```

**原因**:
feature 未启用或子服务初始化失败。

**排查步骤**:

1. 确认 feature 状态
```bash
# 在 BUILD.gn 中确认
grep "safwk_lite_feature_enable" BUILD.gn
```

2. 检查子服务日志
```bash
# 查看 dmesg 或 hilog
hilog | grep -E "abilityms|bundlems"
```

---

### 问题 6：IPC 通信失败

**现象**:
```
[IPC] Send request failed: errno XXX
```

**排查步骤**:

1. 检查权限配置
```bash
# 确认 PMS 配置正确
cat /etc/access_token.cfg
```

2. 检查 IPC_AUTH
```bash
# 确认 ipc_auth 服务运行
ps | grep ipc_auth
```

---

## 调试技巧

### 启用详细日志

**方法 1**: 编译时定义
```gn
# BUILD.gn 中添加
cflags = [ "-DDEBUG_SERVICES_SAFWK_LITE" ]
```

**方法 2**: 运行参数
```bash
DEBUG_SERVICES_SAFWK_LITE=1 ./foundation
```

### 使用 GDB 调试

```bash
# 启动调试
arm-linux-gnueabi-gdb ./foundation

# 设置断点
(gdb) break OHOS_SystemInit
(gdb) break SAMGR_Bootstrap

# 运行
(gdb) run

# 查看调用栈
(gdb) backtrace
```

### 分析 Crash

```bash
# 1. 获取 core dump
ulimit -c unlimited
./foundation
# 等待 crash
gdb ./foundation core
(gdb) bt  # backtrace
```

### 检查进程状态

```bash
# 查看进程信息
cat /proc/<pid>/status

# 查看内存使用
cat /proc/<pid>/maps

# 查看文件描述符
ls -la /proc/<pid>/fd/
```

## 常见错误码

| 错误码 | 含义 | 排查方向 |
|--------|------|---------|
| -1 | pause 返回错误 | 检查信号处理器 |
| -2 | 参数错误 | 检查 argv |
| -3 | 初始化失败 | 检查 Samgr 日志 |
| SIGSEGV | 段错误 | 使用 gdb 调试 |

## 定位路径

| 场景 | 日志路径 | 工具 |
|------|----------|------|
| 启动问题 | `/var/log/safwk.log` | `cat`, `tail` |
| IPC 问题 | `hilog` | `hilog`, `grep` |
| 崩溃问题 | `core dump` | `gdb` |
| 性能问题 | `perf` | `perf top` |
