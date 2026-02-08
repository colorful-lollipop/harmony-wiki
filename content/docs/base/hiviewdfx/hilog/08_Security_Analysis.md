# HiLog 安全风险分析

> 生成时间: 2026-02-06
> 相关证据: 多个源文件（详见证据索引）

---

## 目的

本文档基于代码证据分析 HiLog 模块的安全风险、攻击面、可被利用点，并提供修复建议。

## 适用范围

涵盖 HiLog 模块的所有安全机制和潜在漏洞（hilogd 服务、N-API、通信机制、存储等）。

---

## 威胁模型

### 攻击面图

```
外部输入（恶意应用/工具）
    ↓
┌─────────────────────────────────────────────────┐
│              HiLog 安全边界                     │
├─────────────────────────────────────────────────┤
│  攻击面                                      │
│  ├─ N-API 接口（JS API）                     │
│  ├─ NDK/C API                            │
│  ├─ Unix Domain Socket 通信                  │
│  ├─ hilogd 服务（control/output/input sockets） │
│  ├─ 日志落盘（文件系统）                  │
│  ├─ 正则表达式过滤（DoS）                  │
│  ├─ 流控机制（资源耗尽）                │
│  ├─ 缓冲区管理（内存耗尽）                │
│  └─ 压缩库（内存破坏）                    │
├─────────────────────────────────────────────────┤
│  防护机制                                      │
├─────────────────────────────────────────────────┤
│  ├─ UID 基础访问控制                      │
│  ├─ Socket 凭证传递（SO_PASSCRED）          │
│  ├─ 隐私保护（%{public}/%{private}）       │
│  ├─ Domain 范围验证                        │
│  ├─ PID 过滤权限检查                       │
│  ├─ 缓冲区大小限制                          │
│  ├─ 文件大小和数量限制                      │
│  └─ 流控配额                              │
└─────────────────────────────────────────────────┘
    ↓
敏感操作（读取日志、配置系统、落盘文件）
```

---

## 攻击面分析

### 1. 日志注入攻击

**风险**: 恶意应用通过格式化字符串注入恶意内容

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| N-API 参数解析 | `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:275-334` | `ParseNapiValue()`, `ParseLogContent()` | 参数类型转换和格式化 |
| N-API 缓冲区 | `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:298-334` | MAX_NUMBER=100 | 最多 100 个参数 |

**可利用路径**:
```
恶意应用 → HiLog.debug(0xD002900, "MyTag", "User: %{public}s", payload)
```

其中 payload 包含特殊字符或格式化指令。

**影响**: 信息泄露、日志污染、可能触发解析错误

**修复建议**:
1. 对格式化字符串进行严格验证
2. 限制特殊字符和格式化指令
3. 添加输入长度限制
4. 记录异常输入并拒绝

**状态**: ⚠️ 中等风险 - 存在但影响有限

---

### 2. 隐私数据泄露

**风险**: NDK/C++ API 不受隐私保护限制

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| N-API 隐私控制 | `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:52-54` | `IsPrivateModeEnable()` | 仅对 JS API 生效 |
| C API | `interfaces/native/innerkits/include/hilog/log_c.h` | `HiLogPrint()` | 无隐私检查 |

**可利用路径**:
```
恶意 C++ 应用 → HiLogPrint(LOG_APP, ..., "password: %{private}s", password)
```

由于隐私模式仅对 JS API 有效，C/C++ API 会直接输出明文密码。

**影响**: 敏感信息泄露到日志文件（/data/log/hilog/*.gz），可能被攻击者读取

**修复建议**:
1. 将隐私保护扩展到所有 API 层（C/C++ API）
2. 实施全局隐私策略（系统级别强制隐私模式）
3. 对 NDK/C++ API 的隐私标记也进行验证
4. 记录隐私违规并告警

**状态**: 🔴 高风险 - 敏感数据可直接泄露

---

### 3. 缓冲区溢出 DoS

**风险**: 恶意应用持续写入日志，耗尽系统内存

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| 缓冲区最大大小 | `frameworks/libhilog/include/hilog_common.h:36` | `MAX_BUFFER_SIZE = 16 * 1024 * 1024` | 每类型最多 16MB |
| 缓冲区管理 | `services/hilogd/log_buffer.cpp` | `HilogBuffer::Insert()` | 无大小限制检查，可增长到最大值 |

**可利用路径**:
```
恶意应用 → for (i = 0; i < 100000; i++) {
    HiLog.debug(domain, tag, "DoS payload %d", i);
}
```

导致：
1. 内存耗尽（系统崩溃）
2. 影响 hilogd 响应性
3. 影响其他系统服务

**影响**:
- 内存耗尽导致系统崩溃
- 服务拒绝
- 设备无响应

**修复建议**:
1. 添加每个进程的写入配额限制
2. 添加每秒写入速率限制
3. 监控并限制总内存使用
4. 在缓冲区满时提前拒绝新日志

**状态**: 🔴 高风险 - 可导致系统崩溃

---

### 4. 日志落盘路径遍历

**风险**: 恶意控制命令利用参数注入，在任意位置创建日志文件

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| 落盘路径硬编码 | `frameworks/libhilog/include/hilog_common.h:29` | `HILOG_FILE_DIR = "/data/log/hilog/"` | 固定路径 |
| 文件名生成 | `services/hilogd/log_persister_rotator.cpp` | `GetPersistFileName()` | 使用格式化字符串拼接 |

**可利用路径**:
```
恶意工具 → hilog -w start -l 8M -f "../../../etc/passwd"
```

可能创建文件到：
- `/data/log/hilog/../../../etc/passwd`
- `/system/etc/passwd`（如果路径遍历成功）

**影响**: 文件系统损坏、敏感文件泄露

**修复建议**:
1. 验证和规范化所有路径参数
2. 禁止 `..` 和绝对路径
3. 使用安全的文件名生成机制
4. 限制可写入路径范围到 `/data/log/hilog/`

**状态**: 🔴 高风险 - 可导致文件系统破坏

---

### 5. 流控规避

**风险**: 应用通过创建多个 domain 绕过单 domain 配额限制

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| Domain 流控 | `services/hilogd/flow_control.cpp` | `DomainInfo` 结构 | 基于 domain 的配额 |
| 配额检查 | `services/hilogd/flow_control.cpp` | `CheckDomainQuota()` | 返回 allow/drop/drop |

**可利用路径**:
```
恶意应用 → for (int d = 0xD000000; d < 0xD000005; d++) {
    HiLog.debug(d, tag, "DoS payload %d", ...);
}
```

每个 domain 有独立配额，创建多个 domain 可绕过限制。

**影响**:
- 资源耗尽（大量日志写入）
- 系统 DoS

**修复建议**:
1. 实施 per-PID 流控，限制每个进程的总写入量
2. 限制单个进程可使用的 domain 数量
3. 监控并限制总日志写入速率
4. 将 domain 与进程 UID 绑定

**状态**: 🟡 中等风险 - 可绕过配额限制

---

### 6. 权限升级

**风险**: compromised 进程通过 hilogd 获得更高权限

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| Socket 权限 | `services/hilogd/etc/hilogd.cfg` | hilogControl: 0660 | 只有 log 组可写 |
| UID 检查 | `services/hilogd/service_controller.cpp` | `CheckOutputRqst()` | 仅检查 socket UID |
| 无 AccessToken 验证 | - | - | 未集成现代 OpenHarmony 安全框架 |

**可利用路径**:
```
恶意应用（ compromised ）→ hilog -P <victim_pid> -t all
```

如果应用进程被入侵，可能查看其他应用的日志。

**影响**: 敏感信息泄露、隐私侵犯

**修复建议**:
1. 集成 AccessToken 验证，补充 UID 检查
2. 对 PID 过滤操作进行更严格的权限检查
3. 记录并告警可疑的访问模式
4. 考虑添加审计日志

**状态**: 🟡 中等风险 - 当前 UID 检查不够充分

---

### 7. 正则表达式 DoS

**风险**: 复杂正则表达式导致高 CPU 使用，阻塞 hilogd 线程

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| 正则过滤 | `services/hilogd/service_controller.cpp` | `LogFilter::regex` | 支持 regex 过滤 |
| 正则长度限制 | `frameworks/libhilog/include/hilog_common.h:30` | `MAX_REGEX_STR_LEN = 128` | 无复杂度限制 |

**可利用路径**:
```
恶意工具 → hilog -e "((a+){100,})+((b+){100,})"
```

可能导致：
1. ReDoS（正则拒绝服务）
2. CPU 占用 100%
3. hilogd 无响应

**影响**:
- 服务拒绝
- 系统无响应
- 正常日志查询受阻

**修复建议**:
1. 添加正则复杂度限制
2. 实施正则匹配超时
3. 使用受限的正则引擎（避免 catastrophic backtracking）
4. 限制正则长度到合理范围

**状态**: 🟡 中等风险 - 可导致 DoS

---

### 8. 压缩库漏洞

**风险**: 恶意日志触发压缩库的内存破坏漏洞

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| zlib 使用 | `services/hilogd/log_compress.cpp` | `LogCompress::CompressBuffer()` | 使用 zlib 压缩 |
| zstd 使用 | `services/hilogd/log_compress.cpp` | `LogCompress::CompressBuffer()` | 使用 zstd 压缩 |
| 压缩缓冲区 | `frameworks/libhilog/include/hilog_common.h:37` | `MAX_PERSISTER_BUFFER_SIZE = 64KB` | 固定大小 |

**可利用路径**:
```
恶意应用 → HiLog.debug(domain, tag, " crafted exploit %{public}s", long_malicious_payload)
```

如果存在缓冲区溢出或其他内存破坏漏洞。

**影响**:
- 内存破坏
- 代码执行
- 系统崩溃

**修复建议**:
1. 保持 zlib/zstd 库更新到最新稳定版本
2. 启用并执行压缩库的 fuzz 测试
3. 添加输入长度验证
4. 使用安全的内存管理实践

**状态**: 🟡 中等风险 - 依赖第三方库安全性

---

### 9. 域验证不足

**风险**: 系统 service 可滥用应用 domain，导致日志污染

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| Domain 范围验证 | `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:130-131` | 仅验证数值范围 | 未验证调用者 UID |
| 系统 Domain 范围 | `interfaces/native/innerkits/include/hilog/log_c.h:27-33` | 0xD000000 - 0xD0FFFFF | 保留给系统服务 |

**可利用路径**:
```
恶意系统服务 → HiLogPrint(LOG_CORE, 0xD002900, "MyTag", "spoofed log")
```

系统 service 可以使用应用 domain 范围，混淆和污染日志。

**影响**:
- 日志混淆和污染
- 域验证失效
- 追踪困难

**修复建议**:
1. 验证调用者 UID 和 domain 范围的一致性
2. 对系统服务实施 domain 白名单
3. 记录系统 service 的 domain 使用情况
4. 添加审计日志

**状态**: 🟡 中等风险 - 可能导致日志污染

---

### 10. 日志截断导致信息丢失

**风险**: 应用错误地截断敏感数据导致不完整日志

| 证据 | 文件路径 | 行号/符号 | 说明 |
|------|---------|----------|------|
| N-API 参数限制 | `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:34-35` | MAX_NUMBER=100 | 最多 100 个参数 |
| 单条日志最大长度 | `README_zh.md:167` | 约 3500 字节 | 文档说明 |

**可利用路径**:
```
正常应用 → HiLog.debug(domain, tag, "API Key: %{public}s, very_long_key_string_here...")
```

如果超长，被截断为 "API Key: <private>"，丢失实际值。

**影响**:
- 不完整日志记录
- 调试困难
- 可能暴露部分敏感信息

**修复建议**:
1. 文档化长度限制并添加警告
2. 建议拆分长日志为多条
3. 记录截断事件
4. 提供日志长度查询 API

**状态**: 🟢 低风险 - 信息丢失但不直接导致安全问题

---

## 现有安全机制

### UID 基础访问控制

| 机制 | 描述 | 有效性 |
|------|------|--------|
| ROOT/SHELL/HIVIEW/PROFILER UID 检查 | 限制非特权用户按 PID 过滤 | ✅ 有效 |
| 普通 PID 自动过滤 | 非特权用户只能看到自己的日志 | ✅ 有效 |
| Socket 权限 | 控制写/读权限 | ✅ 有效 |

**证据**: `services/hilogd/service_controller.cpp:444-498`

### Socket 凭证传递

| 机制 | 描述 | 有效性 |
|------|------|--------|
| SO_PASSCRED / SO_PEERCRED | 从内核获取真实 PID/UID | ✅ 有效 - 防止伪造 |
| `__RECV_MSG_WITH_UCRED_` | hilogd 使用凭证 | ✅ 有效 |

**证据**:
- `services/hilogd/BUILD.gn:48`
- `frameworks/libhilog/socket/dgram_socket_server.cpp:33-64`
- `frameworks/libhilog/socket/seq_packet_socket_server.cpp:62-65`

### 隐私保护

| 机制 | 描述 | 局限性 |
|------|------|--------|
| `%{public}/%{private}` 标记 | JS API 隐私控制 | 🟡 仅 JS 层 |
| 隐私模式参数 | 通过 param 控制开关 | 🟡 全局但可能未生效到所有 API |

**证据**:
- `interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:52-54`

### 流控保护

| 机制 | 描述 | 有效性 |
|------|------|--------|
| 进程级配额 | 每进程每秒最多 50K | 🟡 可被 domain 绕过 |
| Domain 级配额 | 每个 domain 有独立配额 | 🟡 可创建多 domain 绕过 |
| 配额开关 | 可通过 `-Q` 命令控制 | ✅ 有效 |

**证据**: `services/hilogd/flow_control.cpp`, `README_zh.md:115-123`

---

## 优先修复建议

### 高优先级（1-3 个月）

1. **扩展隐私保护到所有 API 层**
   - C/C++ API 也应支持隐私标记
   - 实施全局隐私策略

2. **集成 AccessToken 验证**
   - 补充 UID 检查
   - 验证调用者权限

3. **添加 per-PID 流控**
   - 限制每个进程的总写入量
   - 防止通过多 domain 绕过配额

4. **修复日志落盘路径遍历**
   - 验证和规范化所有路径参数
   - 禁止 `..` 和绝对路径

### 中优先级（3-6 个月）

5. **添加正则复杂度限制**
   - 限制正则表达式复杂度
   - 实施匹配超时

6. **记录和告警可疑访问**
   - 添加审计日志
   - 监控异常访问模式

7. **文档化日志长度限制**
   - 添加长度警告
   - 建议日志拆分

### 低优先级（持续改进）

8. **实施输入验证**
   - 对所有输入参数进行严格验证
   - 限制特殊字符

9. **添加监控和指标**
   - 监控日志写入速率
   - 添加资源使用告警

10. **保持依赖库更新**
   - 定期更新 zlib/zstd
   - 执行安全审计

---

## 风险评估总结

| 风险 | 级别 | 优先级 | 说明 |
|------|--------|--------|------|
| 隐私数据泄露 | 🔴 高 | 高 | NDK/C++ API 无隐私保护 |
| 缓冲区溢出 DoS | 🔴 高 | 高 | 无写入配额限制 |
| 日志落盘路径遍历 | 🔴 高 | 高 | 路径参数未充分验证 |
| 权限升级 | 🟡 中 | 中 | 未集成 AccessToken |
| 流控规避 | 🟡 中 | 中 | 可通过多 domain 绕过 |
| 正则表达式 DoS | 🟡 中 | 中 | 无复杂度限制 |
| 压缩库漏洞 | 🟡 中 | 中 | 依赖第三方库 |
| 域验证不足 | 🟡 中 | 中 | 系统 service 可滥用 |
| 日志注入 | ⚠️ 低 | 低 | 存在但影响有限 |
| 日志截断 | 🟢 低 | 低 | 信息丢失 |

---

## 相关跳转链接

- [架构说明](03_Architecture.md)
- [调用链图](appendix/Callgraphs.md)

---

## 证据索引

| 主题 | 文件路径 | 行号/符号 |
|------|---------|----------|
| 隐私控制（JS） | interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:52-54 | IsPrivateModeEnable() |
| 缓冲区大小限制 | frameworks/libhilog/include/hilog_common.h:36 | MAX_BUFFER_SIZE |
| Domain 范围 | interfaces/native/innerkits/include/hilog/log_c.h:27-33 | DOMAIN_APP_MIN/MAX, DOMAIN_OS_MIN/MAX |
| UID 权限检查 | services/hilogd/service_controller.cpp:444-498 | CheckOutputRqst() |
| 落盘路径 | frameworks/libhilog/include/hilog_common.h:29 | HILOG_FILE_DIR |
| 流控实现 | services/hilogd/flow_control.cpp | 全文 | DomainInfo, CheckDomainQuota() |
| 正则长度限制 | frameworks/libhilog/include/hilog_common.h:30 | MAX_REGEX_STR_LEN |
| N-API 参数限制 | interfaces/js/kits/napi/src/hilog/src/hilog_napi_base.cpp:34-35 | MIN_NUMBER, MAX_NUMBER |
