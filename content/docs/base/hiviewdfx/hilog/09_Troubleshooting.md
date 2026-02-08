# HiLog 常见问题与定位

> 生成时间: 2026-02-06
> 相关证据: `README_zh.md:239-274`

---

## 目的

本文档描述 HiLog 模块的常见问题、定位方法、调试技巧。

## 适用范围

涵盖日志未打印、日志丢失、性能问题、配置问题等。

---

## 日志未打印问题

### 问题 1：日志级别不匹配

**现象**: 设置了日志级别但日志未打印

**原因分析**:
1. 全局日志级别设置过低（`hilog -b D` 但应用打印 INFO）
2. 应用日志级别被过滤（domain 流控或进程流控）
3. 日志被隐私过滤掉（`%{private}` 标记）

**定位步骤**:
```
1. 检查全局日志级别
   hilog -b I   # 查询当前全局级别
   hilog -b D -D <domain> -L I  # 确认 domain 是否被过滤

2. 检查 domain 流控状态
   hilog -Q domainon/domainoff  # 查询 domain 流控是否开启

3. 检查进程流控状态
   hilog -Q pidon/pidoff  # 查询进程流控是否开启

4. 检查隐私模式
   hilog -p on  # 确认隐私模式是否开启
```

**修复建议**:
- 调整日志级别以匹配应用
- 关闭流控：`hilog -Q pidoff domainoff`
- 关闭隐私模式：`hilog -p off`

**证据**: `README_zh.md:98-110`

---

### 问题 2：日志超过配额被丢弃

**现象**: 日志中出现 `LOGLIMIT` 提示，日志被丢弃

**示例输出**:
```
04-24 17:02:50.167  2650  2650 W A01B01/LOGLIMIT: ==com.ohos.sceneboard LOGS OVER PROC QUOTA, 46 DROPPED==
```

**原因分析**:
- 进程日志写入速率超过配额（默认 50K/秒）
- Domain 流控配额已用完
- 系统保护防止日志淹没

**定位步骤**:
```
1. 查找 LOGLIMIT 关键字
   hilog | grep "LOGLIMIT"

2. 检查被丢弃的进程
   hilog -P <pid> -L W

3. 查询流控状态
   hilog -Q pidon/pidoff domainon/domainoff

4. 检查统计信息
   hilog -s -D <domain>
```

**修复建议**:
- **临时关闭流控**（仅调试）：`hilog -Q pidoff domainoff`
- **降低日志频率**：减少非必要日志
- **应用日志分级**：DEBUG 日志在发布版本中关闭
- **增大配额**：通过 param 配置（如果需要）

**证据**: `README_zh.md:115-123`, `services/hilogd/flow_control.cpp`

---

### 问题 3：缓冲区满导致日志丢失

**现象**: 日志中出现 `Slow reader` 提示，日志被老化丢失

**示例输出**:
```
04-24 17:02:19.315     0     0 I C00000/HiLog: ========Slow reader missed log lines: 209
```

**原因分析**:
- 日志写入速率超过缓冲区读取速率
- hilog 工具读取速度慢于日志写入速度
- 缓冲区大小不足

**定位步骤**:
```
1. 查找 Slow reader 关键字
   hilog | grep "Slow reader"

2. 检查缓冲区大小
   hilog -g   # 查询当前缓冲区大小
   hilog -G <size>   # 增大缓冲区

3. 检查日志统计
   hilog -s   # 查询统计数据

4. 减少过滤条件
   # 减少不必要的类型、domain、tag 过滤
```

**修复建议**:
- **增大缓冲区大小**：`hilog -G 8M` 或更大
- **减小日志写入速率**：减少非关键日志
- **优化过滤条件**：精确匹配需要的日志
- **使用阻塞模式**：`hilog -x` 不持续读取，减少负载

**证据**: `README_zh.md:241-247`

---

## 性能问题

### 问题 4：日志写入影响应用性能

**现象**: 应用频繁打印日志导致性能下降

**原因分析**:
- 日志写入涉及：
  - Unix Domain Socket 通信
  - 参数格式化
  - 内存分配
  - 系统调用

- 高频日志（DEBUG 级别）特别影响性能

**定位步骤**:
```
1. 使用性能分析工具
   # 使用系统性能监控工具

2. 识别高频日志点
   hilog -s -D <domain>   # 查询统计数据
   hilog -T <tag> -L D   # 查询特定 DEBUG 日志

3. 检查流控状态
   # 频繁被丢弃说明写入过快

4. 优化日志调用
   - 移除调试日志
   - 使用条件编译减少日志调用
   - 使用异步日志（如支持）
```

**修复建议**:
- **使用日志级别分级**：DEBUG 仅在开发时启用
- **减少日志参数**：合并多个参数到一条日志
- **使用日志宏控制**：通过编译时宏开关日志
- **异步日志**：如果支持，使用异步方式减少阻塞

---

### 问题 5：hilogd 响应慢

**现象**: hilog 工具查询响应缓慢

**原因分析**:
- 缓冲区数据量大
- 过滤条件复杂（正则、多类型、多 domain）
- hilogd CPU 占用高
- 多个并发查询

**定位步骤**:
```
1. 检查 hilogd 线程状态
   # 查看进程列表和线程数

2. 检查缓冲区状态
   hilog -g   # 查询缓冲区大小和使用情况

3. 检查流控统计
   hilog -s   # 查询流控和统计信息

4. 优化查询条件
   # 使用更精确的过滤（不要用正则）
   # 减少 time range
   # 使用单一类型过滤
```

**修复建议**:
- **优化过滤条件**：避免正则，使用精确匹配
- **减少并发查询**：避免同时运行多个 hilog 实例
- **增大缓冲区**：减少缓冲区满的情况

---

## 配置问题

### 问题 6：日志落盘失败

**现象**: 执行 `hilog -w start` 后日志未落盘

**可能原因**:
- 磁盘空间不足
- 权限问题
- hilogd 配置问题
- 压缩库未安装

**定位步骤**:
```
1. 检查磁盘空间
   df -h /data/log/hilog

2. 检查落盘任务状态
   hilog -w query   # 查询落盘任务

3. 查看 hilogd 日志
   # 查找错误信息

4. 检查权限
   ls -la /data/log/hilog

5. 重启 hilogd
   # 如果 hilogd 异常，重启服务
```

**修复建议**:
- **检查磁盘空间**：确保 `/data/log/hilog/` 有足够空间
- **检查权限**：确保 logd:log 组有写入权限
- **验证压缩支持**：确认 zlib/zstd 正常工作
- **重启服务**：`killall hilogd` 然后 init 启动

---

### 问题 7：隐私模式不生效

**现象**: 发布版本中敏感数据仍然显示

**可能原因**:
- 使用 NDK/C++ API（不受隐私控制）
- 隐私模式参数未设置
- 应用绕过隐私机制

**定位步骤**:
```
1. 检查隐私模式状态
   hilog -p   # 查询当前状态

2. 检查日志内容
   # 确认是否包含明文敏感信息

3. 检查应用类型
   # 确认是否使用 N-API

4. 查看实现代码
   # 检查是否使用了 %{private} 标记
```

**修复建议**:
- **确认隐私模式**：`hilog -p on`
- **使用 N-API**：优先使用受保护的 JS API
- **手动标记敏感数据**：使用 `%{private}s` 标记
- **检查应用代码**：确保正确使用隐私标记

**证据**: `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:52-54`

---

## 调试技巧

### 技巧 1：精确过滤日志

```
# 按类型和级别过滤
hilog -t app -L I

# 按 tag 过滤
hilog -T MyTag

# 按 domain 过滤
hilog -D 0xD002900

# 正则过滤（小心使用）
hilog -e "error.*failed"

# 排除过滤
hilog -t ^core  # 排除 core 类型
```

### 技巧 2：查看统计信息

```
hilog -s -D <domain>
hilog -s -D 0xD002900
hilog -s   # 查看所有统计
```

### 技巧 3：控制日志输出格式

```
hilog -v time    # 显示本地时间
hilog -v epoch    # 显示 Unix 时间戳
hilog -v color    # 彩色输出
hilog -v nsec     # 显示纳秒精度
hilog -v year     # 显示年份
```

### 技巧 4：查询缓冲区信息

```
hilog -g                # 查询所有类型缓冲区大小
hilog -g -t app        # 查询 app 类型缓冲区大小
hilog -G <size> -t app  # 设置 app 类型缓冲区大小
```

---

## 关键日志关键字

| 关键字 | 含义 | 处理建议 |
|---------|------|----------|
| `LOGLIMIT` | 日志超配额被丢弃 | 关闭流控或减少日志频率 |
| `Slow reader` | 缓冲区满导致日志丢失 | 增大缓冲区或优化读取 |
| `over quota` | domain 配额用完 | 增大 domain 配额或减少该域日志 |
| `no permission` | 无权限操作 | 检查 UID/GID 权限 |
| `file not found` | 文件不存在 | 检查路径或权限 |
| `invalid argument` | 参数错误 | 检查命令语法 |

---

## 相关跳转链接

- [N-API 接口](04_NAPI_Interface.md)
- [内部 API](05_Internal_API.md)
- [架构说明](03_Architecture.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| 日志格式 | README_zh.md:73-78 | 日期时间 进程号 线程号... |
| LOGLIMIT 说明 | README_zh.md:115-123 | 进程/domain 流控 |
| Slow reader 说明 | README_zh.md:241-247 | 缓冲区满导致丢失 |
| 流控开关 | README_zh.md:171-223 | -Q pidon/pidoff/domainon/domainoff |
| 隐私开关 | README_zh.md:125-133 | -p on/off |
| 缓冲区大小 | README_zh.md:219-223 | -g/-G 命令 |
