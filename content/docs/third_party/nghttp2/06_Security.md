# 安全风险分析

## 概述

本文档分析 nghttp2 v1.66.0 在 OpenHarmony 中的安全风险，包括已知 CVE、潜在攻击面以及缓解措施。

---

## 已知 CVE

### 历史 CVE 检查

**TODO(需确认)**: 需要查询以下信息：

1. nghttp2 v1.66.0 是否存在已知 CVE
2. 该版本是否修复了历史 CVE
3. OH 中使用的版本是否存在未修复漏洞

**查询建议**:
- NVD (National Vulnerability Database): https://nvd.nist.gov/
- GitHub Security Advisories: https://github.com/nghttp2/nghttp2/security
- nghttp2 官方公告

### 历史 CVE 示例（供参考）

以下为 nghttp2 历史上曾出现的 CVE 类型（**不代表当前版本存在**）：

| CVE ID | 影响版本 | 类型 | 描述 |
|--------|----------|------|------|
| CVE-2019-XXXX | < 1.39.0 | 拒绝服务 | 特定输入导致无限循环 |
| CVE-2020-XXXX | < 1.41.0 | 内存问题 | HPACK 解码器越界读取 |
| CVE-2023-XXXX | < 1.52.0 | 拒绝服务 | 控制流绕过 |

**注意**: 上述 CVE 仅为示例，v1.66.0 可能已修复。

---

## 攻击面分析

### 直接攻击面

nghttp2 作为网络协议库，攻击面主要在：

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| **HTTP/2 帧解析** | 中 | 处理来自网络的帧数据 |
| **HPACK 解码** | 中 | 解码头压缩数据 |
| **流管理** | 低 | 流状态机处理 |

### 间接攻击面

通过 curl 使用时的攻击面：

```
攻击者 -> HTTP/2 服务器 -> curl -> nghttp2
```

| 场景 | 风险等级 | 说明 |
|------|----------|------|
| 恶意 HTTP/2 响应 | 中 | 服务器返回恶意构造的数据 |
| 中间人攻击 | 低 | TLS 提供保护 |
| 拒绝服务 | 低 | 流控制和速率限制缓解 |

---

## 内置安全机制

### HPACK 安全

| 机制 | 实现 | 作用 |
|------|------|------|
| **动态表大小限制** | `NGHTTP2_DEFAULT_HEADER_TABLE_SIZE` | 防止内存耗尽 |
| **最大头列表大小** | 可配置选项 | 防止大头部攻击 |
| **严格验证** | RFC 7540 合规检查 | 防止协议违规 |

**相关 API**:
```c
// 设置最大动态表大小
nghttp2_option_set_max_deflate_dynamic_table_size(option, size);

// 设置最大头列表大小
nghttp2_option_set_max_header_list_size(option, size);
```

### HTTP/2 流控制

| 机制 | 默认大小 | 作用 |
|------|----------|------|
| **连接窗口** | 65535 字节 | 防止单一连接耗尽内存 |
| **流窗口** | 65535 字节 | 防止单一流耗尽内存 |
| **窗口更新** | 自动/手动 | 接收方控制流量 |

**相关 API**:
```c
// 设置初始连接窗口大小
nghttp2_option_set_peer_max_concurrent_streams(option, num);

// 消费数据（更新窗口）
nghttp2_session_consume(session, stream_id, size);
```

### 速率限制 (v1.66.0 新增)

```c
// 速率限制结构
nghttp2_ratelim rl;

// 初始化
nghttp2_ratelim_init(&rl, burst, rate);

// 更新限制
nghttp2_ratelim_update(&rl, t, n);
```

**作用**: 防止快速发送大量数据导致拒绝服务。

### 其他安全特性

| 特性 | 说明 |
|------|------|
| **严格的 HTTP 语义检查** | `nghttp2_http.c` 验证头字段合规性 |
| **最大并发流限制** | 防止资源耗尽 |
| **Ping 频率限制** | 防止 Ping 洪水攻击 |
| **Settings 限制** | 防止 Settings 洪水攻击 |

---

## OH 特定的安全考虑

### Patch 引入的风险

**Patch**: `src/http-parser.patch`

| 评估项 | 结论 |
|--------|------|
| **影响范围** | 仅示例代码 |
| **OH 使用** | 否（不构建 nghttpx） |
| **风险等级** | 极低 |
| **建议** | 如确认不使用 nghttpx，可考虑删除此 patch |

### 构建配置安全

```gn
# BUILD.gn 中的安全相关配置

# 1. 符号隐藏
version_script = "libnghttp2_shared.map"
# 作用：仅导出必要符号，减少攻击面

# 2. 栈保护
branch_protector_ret = "pac_ret"
# 作用：ARM Pointer Authentication，防止 ROP 攻击

# 3. 编译警告
cflags = [ "-Wno-deprecated-declarations" ]
# 注意：这隐藏了废弃声明警告，但不影响运行时安全
```

---

## 安全升级策略

### 定期检查清单

| 检查项 | 频率 | 方法 |
|--------|------|------|
| CVE 公告 | 每月 | 订阅 nghttp2 安全通告 |
| 版本更新 | 每季度 | 检查上游发布 |
| 安全测试 | 每次升级 | fuzz 测试、压力测试 |

### 升级优先级

| 情况 | 优先级 | 行动 |
|------|--------|------|
| 远程代码执行 CVE | 紧急 | 立即升级 |
| 拒绝服务 CVE | 高 | 尽快升级 |
| 信息泄露 CVE | 中 | 计划升级 |
| 功能更新 | 低 | 按需升级 |

### 升级前检查

升级 nghttp2 版本前：

1. **安全公告检查**
   - [ ] 检查新版本修复的 CVE
   - [ ] 评估当前版本的风险

2. **兼容性验证**
   - [ ] curl 功能测试
   - [ ] HTTP/2 协议合规性测试
   - [ ] 性能基准测试

3. **模糊测试 (Fuzzing)**
   ```bash
   # 使用上游 fuzz 测试
   cd tests/fuzz
   # 运行 fuzz 测试...
   ```

---

## 安全最佳实践

### 使用 nghttp2 时的安全建议

#### 1. 限制资源使用

```c
nghttp2_option *option;
nghttp2_option_new(&option);

// 限制最大并发流
nghttp2_option_set_peer_max_concurrent_streams(option, 100);

// 限制最大头列表大小
nghttp2_option_set_max_header_list_size(option, 16384);

// 创建会话时应用选项
nghttp2_session_client_new2(&session, callbacks, user_data, option);
nghttp2_option_del(option);
```

#### 2. 验证输入数据

```c
// 回调中验证数据
static int on_header_callback(nghttp2_session *session,
                               const nghttp2_frame *frame,
                               const uint8_t *name, size_t namelen,
                               const uint8_t *value, size_t valuelen,
                               uint8_t flags, void *user_data) {
    // 验证头字段长度
    if (namelen > MAX_HEADER_NAME_LEN || 
        valuelen > MAX_HEADER_VALUE_LEN) {
        return NGHTTP2_ERR_TEMPORAL_CALLBACK_FAILURE;
    }
    return 0;
}
```

#### 3. 设置超时

```c
// 使用非阻塞 I/O + 超时
// 防止慢速攻击
struct timeval timeout = {30, 0};  // 30秒超时
setsockopt(fd, SOL_SOCKET, SO_RCVTIMEO, &timeout, sizeof(timeout));
```

### 系统级安全建议

| 层级 | 建议 |
|------|------|
| **沙箱** | 在沙箱中运行网络服务 |
| **防火墙** | 限制不必要的出站连接 |
| **监控** | 监控异常流量模式 |
| **更新** | 及时应用安全更新 |

---

## 漏洞响应

### 发现漏洞时的流程

1. **评估影响**
   - 漏洞类型（RCE、DoS、信息泄露）
   - 触发条件
   - 影响范围

2. **临时缓解**
   - 禁用受影响功能
   - 增加访问限制
   - 监控异常行为

3. **修复**
   - 应用官方补丁
   - 或升级到修复版本

4. **验证**
   - 确认漏洞已修复
   - 回归测试

### 上报漏洞

如发现 nghttp2 安全问题：
- **上游**: https://github.com/nghttp2/nghttp2/security
- **OpenHarmony**: 遵循 OH 安全响应流程

---

## 总结

### 当前安全状态

| 评估项 | 状态 | 说明 |
|--------|------|------|
| 已知 CVE | 待确认 | 需查询 v1.66.0 的 CVE 状态 |
| 内置保护 | 良好 | HPACK 安全、流控制、速率限制 |
| Patch 风险 | 极低 | 仅影响示例代码 |
| 升级难度 | 低 | 无 OH 特定代码修改 |

### 安全建议

1. **短期**
   - [ ] 确认 v1.66.0 的 CVE 状态
   - [ ] 评估是否使用 nghttpx（决定是否保留 http-parser.patch）

2. **中期**
   - [ ] 建立定期 CVE 检查机制
   - [ ] 添加模糊测试到 CI

3. **长期**
   - [ ] 考虑 HTTP/3 (ngtcp2) 作为未来升级方向
   - [ ] 关注 QUIC/HTTP3 标准化进展

### 风险等级

**总体风险**: 🟢 低

- 库本身设计良好，有内置安全机制
- 只有 curl 使用，攻击面有限
- 无 OH 特定代码引入新风险
- 版本较新 (v1.66.0)，通常包含最新安全修复
