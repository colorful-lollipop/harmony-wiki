# 06 - 安全风险分析

## 6.1 安全概览

### 组件安全等级

| 组件 | 安全等级 | 原因 |
|-----|---------|------|
| cangjie-runtime | **高** | 核心运行时，处理内存管理、线程调度 |
| cangjie-std-* | **中** | 标准库，处理用户数据 |
| libboundscheck | **高** | 边界检查，防止缓冲区溢出 |
| libpcre2-8 | **中** | 正则表达式，历史上有 ReDoS 漏洞 |
| OpenSSL (系统) | **高** | 加密功能，频繁出现 CVE |

### 攻击面分析

```
┌─────────────────────────────────────────────────────────────┐
│                      攻击面分析                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  输入层                                                     │
│  ├── 网络输入 (std.net) ────────→ 解析漏洞、DoS             │
│  ├── 文件输入 (std.fs) ─────────→ 路径遍历、解析漏洞        │
│  ├── 用户输入 (std.console) ────→ 注入攻击                  │
│  └── 正则表达式 (std.regex) ────→ ReDoS                     │
│                                                             │
│  处理层                                                     │
│  ├── GC/内存管理 ───────────────→ UAF、Double Free          │
│  ├── 线程调度 ──────────────────→ 竞争条件、死锁            │
│  └── FFI/C 互操作 ──────────────→ 内存损坏                  │
│                                                             │
│  依赖层                                                     │
│  ├── OpenSSL ───────────────────→ 加密漏洞                  │
│  ├── PCRE2 ─────────────────────→ 正则漏洞                  │
│  └── Musl libc ─────────────────→ 系统调用漏洞              │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 6.2 CVE 状态分析

### 本仓库 CVE

**结论**：cangjie_runtime 作为华为自研项目，目前**无公开 CVE**。

### 依赖库 CVE 跟踪

#### OpenSSL

| 版本 | 状态 | CVE 风险 |
|-----|------|---------|
| OHOS 系统预装 | 由 OH 安全团队维护 | 高（需持续跟踪） |

**建议**：
- 关注 OH 安全公告
- 及时更新系统 OpenSSL
- 避免使用已弃用的 TLS 版本

#### PCRE2

| 版本 | 状态 | CVE 风险 |
|-----|------|---------|
| 10.40+ (绑定) | 当前版本 | 中 |

**历史 CVE**（参考）：
- CVE-2022-1587: PCRE2 堆缓冲区溢出
- CVE-2020-14155: PCRE 整数溢出

**缓解措施**：
- 使用 JIT 编译模式（性能优化 + 安全检查）
- 对用户输入的正则表达式进行复杂度限制
- 设置匹配超时

#### bounds_checking_function

| 版本 | 状态 | CVE 风险 |
|-----|------|---------|
| OpenHarmony-v6.0-Release | 华为维护 | 低 |

此库专门用于安全加固，本身无已知 CVE。

#### flatbuffers

| 版本 | 状态 | CVE 风险 |
|-----|------|---------|
| 绑定版本 | 当前版本 | 低 |

**历史 CVE**：
- CVE-2020-35864: 堆缓冲区溢出（已修复）

## 6.3 安全特性

### 已启用的安全机制

#### 1. 边界检查（bounds_checking_function）

```
功能：对字符串和内存操作进行边界检查
位置：libboundscheck.so
效果：防止缓冲区溢出
```

涉及函数：
- `memcpy_s`
- `strcpy_s`
- `strcat_s`
- `sprintf_s`
- 等安全函数

#### 2. CFI（Control Flow Integrity）

```gn
# runtime/runtime_config.gni
cflags = [
  "-flto",
  "-fsanitize=cfi",
  "-fno-sanitize=cfi-nvcall,cfi-icall",
]
```

**保护效果**：
- 防止控制流劫持攻击
- 确保间接调用目标合法

#### 3. PAC-RET（Pointer Authentication）

ARM64 特有：
```gn
"-mbranch-protection=pac-ret"
```

**保护效果**：
- 返回地址签名验证
- 防止 ROP 攻击

#### 4. GWP-ASAN

```gn
GWPASAN_SUPPORT_FLAG = 1  # 默认启用
```

**保护效果**：
- 检测 UAF（Use-After-Free）
- 检测堆缓冲区溢出
- 低开销（仅采样部分分配）

#### 5. FORTIFY_SOURCE

```gn
"-D_FORTIFY_SOURCE=2"
```

**保护效果**：
- 编译时缓冲区溢出检测
- 常用字符串/内存函数检查

#### 6. 沙箱隔离

OH 平台特性：
```
命名空间隔离：运行时与应用代码隔离
效果：限制攻击扩散范围
```

### 安全编译选项汇总

| 选项 | 说明 | 状态 |
|-----|------|------|
| `-D_FORTIFY_SOURCE=2` | 强化检查 | ✅ 启用 |
| `-fstack-protector-strong` | 栈保护 | ✅ 启用 |
| `-fsanitize=cfi` | 控制流完整性 | ✅ 启用 |
| `-mbranch-protection=pac-ret` | 指针认证 | ✅ ARM64 |
| GWP-ASAN | 堆错误检测 | ✅ 启用 |
| ASAN/HWASAN | 地址消毒 | ⚪ 调试可选 |

## 6.4 潜在风险

### 高风险

#### 1. 内存管理

**风险**：GC 或内存分配器漏洞

**场景**：
- UAF（Use-After-Free）
- Double Free
- 堆溢出

**缓解**：
- GWP-ASAN 采样检测
- 边界检查库
- 代码审计

#### 2. FFI 边界

**风险**：Cangjie ↔ C 互操作边界

**场景**：
- 类型混淆
- 生命周期管理错误
- 缓冲区溢出

**缓解**：
- 严格类型检查
- 边界检查
- 沙箱隔离

### 中风险

#### 3. 正则表达式（ReDoS）

**风险**：灾难性回溯导致 DoS

**场景**：
```cangjie
// 危险：可能导致指数级回溯
let pattern = Regex("(a+)+$")
pattern.matches("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!")  // ReDoS
```

**缓解**：
- 设置匹配超时
- 用户输入的正则预检查
- PCRE2 JIT 优化

#### 4. 网络输入

**风险**：协议解析漏洞

**场景**：
- HTTP 头注入
- URL 解析漏洞
- 响应解析错误

**缓解**：
- 输入验证
- 使用安全的解析库
- 协议合规检查

### 低风险

#### 5. 第三方依赖

**风险**：依赖库 CVE

**缓解**：
- 及时更新依赖
- 监控安全公告
- 使用系统提供的 OpenSSL

## 6.5 安全升级建议

### 例行维护

| 频率 | 任务 |
|-----|------|
| 每周 | 检查 OH 安全公告 |
| 每月 | 检查依赖库 CVE |
| 每季度 | 安全代码审计 |
| 每版本 | 完整安全测试 |

### 升级检查清单

#### 运行时升级

- [ ] 新版本是否修复已知安全漏洞
- [ ] GC/内存管理相关变更是否经过安全审查
- [ ] FFI 边界是否保持兼容
- [ ] 安全编译选项是否仍然启用

#### 标准库升级

- [ ] 新增 API 是否有安全风险
- [ ] 修改的 API 是否影响边界检查
- [ ] 网络/文件相关 API 是否经过 fuzz 测试

#### 依赖升级

- [ ] PCRE2 版本是否有 CVE
- [ ] OpenSSL 是否需要系统更新
- [ ] bounds_checking_function 是否有更新

### 应急响应

#### CVE 发现时

1. **评估**：确认影响范围和严重程度
2. **隔离**：如可能，临时禁用受影响功能
3. **修复**：
   - 自有代码：快速修复
   - 依赖库：等待上游或临时补丁
4. **验证**：修复后完整测试
5. **发布**：更新版本并公告

#### 安全事件响应联系人

```
维护者: zhangbin1@huawei.com
安全团队: OpenHarmony 安全响应中心
```

## 6.6 安全测试建议

### 建议的测试类型

| 测试类型 | 目标 | 工具 |
|---------|------|------|
| Fuzzing | 解析器、网络协议 | AFL、LibFuzzer |
| 静态分析 | 代码漏洞 | Coverity、CodeQL |
| 动态分析 | 运行时行为 | ASAN、TSAN |
| 渗透测试 | 系统安全 | 专业团队 |

### 重点测试区域

```
1. 内存管理模块 (runtime/src/Heap/)
   - GC 正确性
   - 内存分配/释放
   - 边界检查

2. 线程管理 (runtime/src/CJThread/)
   - 竞争条件
   - 死锁
   - 线程安全

3. 异常处理 (runtime/src/Exception/)
   - 异常传播
   - 资源清理
   - 栈展开

4. FFI 层
   - 类型转换
   - 内存共享
   - 生命周期

5. 标准库
   - 字符串处理
   - 网络解析
   - 文件操作
   - 正则表达式
```

## 6.7 合规性

### 许可证合规

| 组件 | 许可证 | 合规状态 |
|-----|-------|---------|
| cangjie_runtime | Apache-2.0 with Runtime Exception | ✅ 合规 |
| bounds_checking_function | 华为自有 | ✅ 合规 |
| PCRE2 | BSD | ✅ 合规 |
| flatbuffers | Apache-2.0 | ✅ 合规 |
| OpenSSL | Apache-2.0 | ✅ 合规 |

### 安全认证

| 认证 | 状态 | 说明 |
|-----|------|------|
| OAT (OH Open Source Compliance) | ✅ 通过 | OAT.xml |
| 安全代码审计 | 建议定期执行 | 华为内部 |

## 6.8 参考资源

### 安全公告

- OpenHarmony Security Advisory: https://gitee.com/openharmony/security
- Huawei PSIRT: https://www.huawei.com/en/psirt

### 漏洞数据库

- NVD (National Vulnerability Database): https://nvd.nist.gov/
- CVE Details: https://www.cvedetails.com/

### 安全最佳实践

- OWASP: https://owasp.org/
- CWE/SANS Top 25: https://cwe.mitre.org/top25/

---

*本文档基于 cangjie_runtime 1.1.0-alpha.69 版本编写*
*安全分析仅供参考，实际安全评估需结合具体部署环境*
