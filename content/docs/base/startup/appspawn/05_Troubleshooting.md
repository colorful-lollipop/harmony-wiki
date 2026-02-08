# 常见问题与调试指南

## 目录

- [常见构建问题](#常见构建问题)
- [运行时问题](#运行时问题)
- [调试方法](#调试方法)
- [日志分析](#日志分析)
- [问题定位](#问题定位)

## 常见构建问题

### 问题1: GN 配置错误

**症状**: 构建失败，提示找不到配置变量

```
ERROR: Can't load input file: //base/startup/appspawn/appspawn.gni
```

**原因**: 未正确设置环境变量

**解决方案**:

```bash
# 1. 设置 OHOS_HOME
export OHOS_HOME=/path/to/openharmony

# 2. 使用 hb set 配置目标
hb set

# 3. 重新构建
hb build appspawn
```

### 问题2: 依赖缺失

**症状**: 构建链接失败，提示未定义符号

```
/usr/bin/ld: cannot find -lappspawn_client
```

**原因**: 依赖组件未构建

**解决方案**:

```bash
# 1. 先构建依赖组件
hb build ability_runtime
hb build ipc

# 2. 重新构建 appspawn
hb build appspawn
```

### 问题3: 版本不兼容

**症状**: 运行时崩溃，提示版本不匹配

```
[ERROR] Symbol version mismatch for libappspawn_client.so
```

**原因**: 代码版本与编译时不匹配

**解决方案**:

```bash
# 清理构建缓存后重新编译
hb clean
hb build appspawn
```

## 运行时问题

### 问题1: Socket 连接失败

**症状**: 客户端无法连接 appspawn

```
[E0001] Failed to connect /run/data/service/el1/startup/appspawn/AppSpawn
```

**排查步骤**:

```bash
# 1. 检查 appspawn 是否运行
ps aux | grep appspawn

# 2. 检查 socket 路径权限
ls -la /run/data/service/el1/startup/appspawn/

# 3. 检查 SELinux 上下文
ls -Z /run/data/service/el1/startup/appspawn/

# 4. 手动启动 appspawn
/system/bin/appspawn -mode appspawn
```

**常见原因**:

| 原因 | 解决方案 |
|------|---------|
| appspawn 未启动 | 检查 init 配置 |
| socket 权限错误 | 修改 cfg 文件 |
| SELinux 拒绝 | 设置正确上下文 |

### 问题2: 应用孵化失败

**症状**: AMS 调用 appspawn 但应用未启动

**日志定位**:

```bash
# 查看 appspawn 日志
hilog | grep appspawn

# 或查看串口日志
dmesg | grep appspawn
```

**常见错误码**:

| 错误码 | 含义 | 排查方向 |
|--------|------|---------|
| APPSPAWN_ARG_INVALID | 参数非法 | 检查消息内容 |
| APPSPAWN_MSG_INVALID | 消息格式错误 | 检查TLV格式 |
| APPSPAWN_SANDBOX_INVALID | 沙箱创建失败 | 检查沙箱配置 |
| APPSPAWN_CHILD_CRASH | 子进程崩溃 | 检查应用日志 |
| APPSPAWN_SPAWN_TIMEOUT | 孵化超时 | 检查系统负载 |

### 问题3: 沙箱配置错误

**症状**: 应用启动后无法访问数据目录

```
[E0002] Failed to mount sandbox for bundle com.example.app
```

**排查步骤**:

```bash
# 1. 检查沙箱配置文件
cat /system/etc/appdata-sandbox.json

# 2. 检查目标目录权限
ls -la /data/service/el1/

# 3. 检查 /mnt/sandbox 目录
ls -la /mnt/sandbox/
```

### 问题4: 权限设置失败

**症状**: 应用启动后 UID/GID 错误

```
[E0003] setuid(10000) failed: Operation not permitted
```

**排查步骤**:

```bash
# 1. 检查 appspawn 权限
getcap /system/bin/appspawn

# 2. 检查 SELinux 状态
getenforce

# 3. 检查 capability
cat /proc/self/status | grep Cap
```

## 调试方法

### 方法1: 增加日志级别

```bash
# 设置环境变量
export APPSPAWN_LOG_LEVEL=DEBUG

# 或修改代码中的日志级别
# 在 appspawn_service.c 中
#define APPSPAWN_LOG_LEVEL APPSPAWN_LOG_DEBUG
```

### 方法2: 使用 gdb 调试

```bash
# 1. 附加到进程
gdb /system/bin/appspawn
(gdb) attach <pid>

# 2. 设置断点
(gdb) break AppSpawnReqMsgCreate

# 3. 继续执行
(gdb) continue

# 4. 查看调用栈
(gdb) bt
```

### 方法3: strace 系统调用追踪

```bash
# 追踪所有系统调用
strace -f -o appspawn_strace.txt /system/bin/appspawn

# 只追踪特定调用
strace -e trace=mount,bind /system/bin/appspawn
```

### 方法4: ltrace 库调用追踪

```bash
# 追踪库函数调用
ltrace -f -o appspawn_ltrace.txt /system/bin/appspawn
```

### 方法5: 动态分析

```bash
# Address Sanitizer
./appspawn --enable_asan=1

# Valgrind
valgrind --leak-check=full /system/bin/appspawn
```

## 日志分析

### 日志标签

| 标签 | 模块 | 说明 |
|------|------|------|
| APP/Startup | appspawn | 主服务 |
| APP/Sandbox | sandbox | 沙箱模块 |
| APP/Common | common | 公共模块 |
| APP/HNP | hnp | HNP模块 |

### 日志级别

| 级别 | 值 | 说明 |
|------|-----|------|
| ERROR | 1 | 错误 |
| WARN | 2 | 警告 |
| INFO | 3 | 信息 |
| DEBUG | 4 | 调试 |
| VERBOSE | 5 | 详细 |

### 日志格式

```
[时间戳][级别][标签] 消息内容 (文件:行号)
```

**示例**:

```
2024-01-15 10:30:45.123 [E/APP/Startup] Failed to connect socket (appspawn_service.c:145)
2024-01-15 10:30:45.124 [I/APP/Sandbox] Create sandbox for bundle com.example.app (appspawn_sandbox.c:234)
```

### 常用日志过滤

```bash
# 只看appspawn错误
hilog | grep "APP/Startup" | grep "E"

# 查看孵化相关日志
hilog | grep "APP/Startup" | grep "spawn"

# 查看沙箱日志
hilog | grep "APP/Sandbox"

# 查看最近100行
hilog | tail -n 100 | grep appspawn
```

## 问题定位

### 问题模板

当报告问题时，请包含以下信息：

```markdown
## 问题描述
[简要描述问题]

## 环境信息
- 设备型号: 
- OS版本: 
- appspawn版本: 

## 重现步骤
1. [步骤1]
2. [步骤2]
3. [...]

## 预期行为
[期望的行为]

## 实际行为
[实际的行为]

## 日志
[相关日志片段]

## 分析
[初步分析]
```

### 常见问题快速定位

#### 应用无法启动

1. 检查 AMS → appspawn 是否正常通信
2. 检查消息格式是否正确
3. 检查 UID/GID 是否有效
4. 检查沙箱配置是否正确
5. 检查应用入口是否存在

#### 权限设置失败

1. 检查 appspawn 进程 capability
2. 检查 SELinux 状态
3. 检查目标 UID/GID 范围
4. 检查 setgroups 调用

#### 沙箱创建失败

1. 检查沙箱配置文件语法
2. 检查 /mnt/sandbox 目录权限
3. 检查 mount 系统调用权限
4. 检查 namespace 支持

### 调试技巧

#### 1. 消息抓包

```c
// 在 appspawn_service.c 中添加调试代码
void PrintMsg(const AppSpawnMsg *msg)
{
    printf("MsgType: %u\n", msg->msgType);
    printf("MsgLen: %u\n", msg->msgLen);
    printf("ProcessName: %s\n", msg->processName);
    // ... 其他字段
}
```

#### 2. Socket 调试

```bash
# 使用 socat 监控 socket
socat -t 100 -x UNIX-LISTEN:/tmp/appspawn_debug,fork,reuseaddr \
      UNIX-CONNECT:/run/data/service/el1/startup/appspawn/AppSpawn
```

#### 3. 子进程调试

```bash
# 设置调试模式
export APPSPAWN_DEBUG_CHILD=1

# 使用 strace 追踪子进程
strace -f -o child_strace.txt /system/bin/appspawn
```

### 性能问题排查

#### 孵化耗时分析

```bash
# 启用性能日志
export APPSPAWN_PROFILE=1

# 查看孵化时间
hilog | grep "spawn time"
```

#### 常见瓶颈

| 瓶颈 | 原因 | 解决方案 |
|------|------|---------|
| 消息解析慢 | TLV解析效率低 | 优化解析算法 |
| 沙箱创建慢 | mount 操作多 | 减少不必要的挂载 |
| fork 慢 | 系统资源不足 | 检查内存/进程数限制 |

### 崩溃分析

#### 获取 core dump

```bash
# 启用 core dump
ulimit -c unlimited
echo "/data/core.%p" > /proc/sys/kernel/core_pattern

# 重现问题后
gdb /system/bin/appspawn /data/core.<pid>
```

#### 分析调用栈

```gdb
# 查看调用栈
(gdb) bt

# 查看局部变量
(gdb) info locals

# 查看寄存器
(gdb) info registers
```

### 安全问题排查

#### SELinux 问题

```bash
# 查看拒绝日志
audit2allow -w -a

# 生成策略模块
audit2allow -M appspawn_fix
semodule -i appspawn_fix.pp
```

#### Capability 问题

```bash
# 查看进程 capability
getpcaps <pid>

# 检查文件 capability
getcap /system/bin/appspawn
```

## 相关资源

### 内部资源
- 代码路径: `base/startup/appspawn`
- API文档: `interfaces/innerkits/`
- 配置路径: `*.cfg`, `appdata-sandbox*.json`

### 外部资源
- OpenHarmony 官方文档
- Linux man pages (socket, namespaces, capabilities)
- SELinux 文档

### 工具
- hilog: 系统日志工具
- bm: 性能分析工具
- gdb: 调试器
- strace: 系统调用追踪
