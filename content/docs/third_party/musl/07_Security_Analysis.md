# 安全风险分析

> OpenHarmony musl 安全风险评审

---

## 目的与适用范围

**目的**: 识别 musl 中的潜在安全风险，提供攻击面分析和修复建议。

**适用范围**: 安全工程师、代码审计人员、系统架构师。

**检查范围**: 
- 内存管理 (mallocng, GWP-ASan)
- 动态链接器 (ldso, namespace)
- 字符串操作
- 文件系统操作
- 网络操作
- Hook 机制

---

## 威胁模型

### 攻击面概览

```
┌─────────────────────────────────────────────────────────────┐
│                      外部输入                                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │ 文件路径 │ │ 网络数据 │ │ 环境变量 │ │ 用户数据 │          │
│  └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘          │
└───────┼───────────┼───────────┼───────────┼─────────────────┘
        │           │           │           │
        └───────────┴─────┬─────┴───────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    musl 攻击面                              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐          │
│  │动态链接器│ │内存分配器│ │ 字符串  │ │ 文件IO  │          │
│  │ ldso    │ │mallocng │ │ 操作    │ │ 操作    │          │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 说明 |
|------|------|
| 应用 ↔ musl | 应用调用 libc API，musl 负责参数校验 |
| musl ↔ 内核 | musl 封装系统调用，处理返回值 |
| 库 ↔ 库 | Namespace 机制隔离不同来源的库 |

---

## 安全风险清单

### 风险 1: 内存分配器元数据损坏

**证据**: `src/malloc/mallocng/meta.h:144-171`

```c
static inline struct meta *get_meta(const unsigned char *p)
{
    // 从内存地址获取元数据
    int offset = *(const uint16_t *)(p - 2);
    const struct group *base = (const void *)(p - UNIT*offset - UNIT);
    const struct meta *meta = encode_ptr(base->meta, ctx.secret);
    // 安全检查...
}
```

**触发条件**: 
- 堆溢出覆盖相邻内存的元数据
- Use-after-free 后修改元数据

**影响**: 
- 内存泄漏
- 任意代码执行（通过控制分配地址）

**修复建议**:
1. 启用 `MALLOC_FREELIST_HARDENED` 加固空闲列表
2. 启用 `MALLOC_SECURE_ALL` 启用所有安全特性
3. 使用 GWP-ASan 检测内存错误

**当前状态**: ✅ 已有防护机制（指针混淆、安全检查）

---

### 风险 2: 动态链接器路径遍历

**证据**: `ldso/linux/namespace.h:94`

```c
bool is_accessible(ns_t *ns, const char *lib_pathname, bool is_asan, bool check_inherited);
```

**触发条件**:
- 构造包含 `../` 的库路径
- 利用软链接绕过路径检查

**影响**:
- 加载非授权路径的库
- 绕过 Namespace 隔离

**修复建议**:
1. 规范化路径（realpath）后再检查
2. 禁止路径中的 `..` 组件
3. 定期检查 permitted_paths 配置

**当前状态**: ⚠️ 需确认路径规范化处理

---

### 风险 3: 环境变量注入

**证据**: `ldso/linux/dynlink_rand.h:39` (LD_LIBRARY_PATH 处理)

**触发条件**:
- 攻击者控制环境变量
- 通过 setuid 程序利用

**影响**:
- 加载恶意库
- 劫持函数调用

**修复建议**:
1. setuid 程序清除敏感环境变量
2. Namespace 配置覆盖环境变量
3. 签名验证动态库

**当前状态**: ✅ Namespace 机制提供隔离

---

### 风险 4: 字符串操作缓冲区溢出

**证据**: 标准字符串函数（`src/string/strcpy.c` 等）

```c
char *strcpy(char *dest, const char *src)
{
    // 无长度检查
}
```

**触发条件**:
- 使用不安全的字符串函数
- 源字符串长度超过目标缓冲区

**影响**:
- 栈/堆溢出
- 代码执行

**修复建议**:
1. 使用 `strlcpy`, `strlcat` 等安全函数
2. 启用 FORTIFY_SOURCE 检查
3. 静态分析检测不安全调用

**当前状态**: ✅ 提供安全函数，但无法强制使用

---

### 风险 5: 文件描述符耗尽

**证据**: `src/internal/musl_fdsan.h:25-41`

```c
#define FdTableSize 128
struct FdTable {
    struct FdEntry entries[FdTableSize];
    _Atomic(struct FdTableOverflow*) overflow;
};
```

**触发条件**:
- 大量打开文件不关闭
- 攻击者耗尽 FD 资源

**影响**:
- 拒绝服务
- 无法打开新文件

**修复建议**:
1. 使用 FDSan 检测 FD 泄漏
2. 设置进程 FD 限制
3. 代码审查确保 FD 关闭

**当前状态**: ✅ 已集成 FDSan

---

### 风险 6: Hook 机制滥用

**证据**: `src/hook/linux/musl_preinit_common.h:58-76`

```c
#ifdef HOOK_ENABLE
extern void* function_of_shared_lib[];
extern volatile atomic_llong ohos_malloc_hook_shared_library;
#endif
```

**触发条件**:
- 恶意库注册 Hook
- 拦截敏感操作

**影响**:
- 信息泄露
- 行为篡改

**修复建议**:
1. Hook 库签名验证
2. 限制 Hook 注册权限
3. 审计 Hook 调用

**当前状态**: ⚠️ 需确认 Hook 库验证机制

---

### 风险 7: 整数溢出

**证据**: `src/malloc/mallocng/meta.h:255-262`

```c
static inline int size_overflows(size_t n)
{
    if (n >= SIZE_MAX/2 - 4096) {
        errno = ENOMEM;
        return 1;
    }
    return 0;
}
```

**触发条件**:
- 分配大小计算溢出
- 乘法/加法溢出

**影响**:
- 分配过小缓冲区
- 堆溢出

**修复建议**:
1. 使用 `size_overflows` 检查
2. 编译器整数溢出检查
3. 代码审查关键计算

**当前状态**: ✅ 已有溢出检查

---

### 风险 8: 竞态条件

**证据**: 文件操作、信号处理

**触发条件**:
- 多线程访问共享资源
- 信号处理程序中断

**影响**:
- 数据不一致
- 权限提升

**修复建议**:
1. 使用原子操作
2. 适当的锁保护
3. 信号处理程序简化

**当前状态**: ⚠️ 需具体场景分析

---

## 安全特性总结

| 特性 | 状态 | 说明 |
|------|------|------|
| mallocng 安全分配 | ✅ | 指针混淆、安全检查 |
| GWP-ASan | ✅ | 内存错误检测 |
| FDSan | ✅ | FD 泄漏检测 |
| FORTIFY | ✅ | 缓冲区溢出检查 |
| Namespace 隔离 | ✅ | 库加载隔离 |
| 地址随机化 | ✅ | 加载器随机化 |
| RELRO | ✅ | 重定位只读 |
| 栈保护 | ✅ | stack-protector-strong |

---

## 安全建议

### 编译时

```bash
# 启用最高安全级别
gn gen out --args="
  musl_secure_level=3
  musl_use_gwp_asan=true
  musl_use_flto=true
"
```

### 运行时

1. 使用 Namespace 隔离不同来源的库
2. 定期审计动态库签名
3. 监控系统调用异常

### 开发时

1. 使用安全字符串函数
2. 检查所有内存分配返回值
3. 避免使用已废弃的 API

---

## 相关跳转

- [项目概览](00_Overview.md) - 安全特性介绍
- [架构设计](02_Architecture.md) - 安全架构
- [附录/Config_Flags](appendix/Config_Flags.md) - 安全配置宏

