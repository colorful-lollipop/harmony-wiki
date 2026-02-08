# Hilog Lite 安全风险评审

本文档对 hilog_lite 组件进行安全风险评审，识别潜在攻击面和可被利用点。

## 评审范围

| 范围 | 描述 |
|------|------|
| 代码范围 | `frameworks/`、`services/`、`command/` 目录下源代码 |
| 接口范围 | `interfaces/` 目录下所有对外 API |
| 构建配置 | `BUILD.gn`、`bundle.json` 中所有配置项 |
| **不包括** | 测试代码、内核驱动实现 |

---

## 威胁模型

### 信任边界

```
┌─────────────────────────────────────────────────────────────────┐
│                         信任边界                                │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Kernel Space (Ring Buffer Driver)                        │  │
│  │  - 内核驱动                                               │  │
│  │  - ioctl 接口                                             │  │
│  └────────────────────────┬──────────────────────────────────┘  │
│                           │ ioctl                              │
│  ┌────────────────────────▼──────────────────────────────────┐  │
│  │  User Space (hilog_lite)                                  │  │
│  │  - 框架代码 (frameworks/)                                  │  │
│  │  - 服务进程 (hilogcat/apphilogcat)                        │  │
│  │  - 命令工具 (hilogcat)                                     │  │
│  └────────────────────────┬──────────────────────────────────┘  │
│                           │ system call / pipe / file           │
│  ┌────────────────────────▼──────────────────────────────────┐  │
│  │  Application Space                                        │  │
│  │  - 用户应用进程                                            │  │
│  │  - JS 应用                                                │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流

| 阶段 | 数据流 | 安全相关 |
|------|--------|----------|
| 1 | App → HiLogPrint | 参数校验 |
| 2 | Framework → ioctl | 系统调用 |
| 3 | Kernel Ring Buffer | 内核存储 |
| 4 | hilogcat → stdout | 输出处理 |
| 5 | apphilogcat → 文件 | 落盘存储 |

---

## 攻击面分析

### 1. API 参数注入

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 格式字符串 | 用户可控的格式字符串参数 | 中 |
| 模块 ID | 超出范围或未注册的模块 ID | 低 |
| 标签长度 | 超过 32 字节的标签 | 低 |
| 格式字符串长度 | 超过 1024 字节的格式字符串 | 低 |

**现有防护**:
```c
// hiview_log.c:81-95 - 参数校验
if ((id >= HILOG_MODULE_MAX) || name == NULL || g_logModuleInfo[id].name != NULL) {
    return FALSE;
}

uint32 len = (uint32)strnlen(name, LOG_MODULE_NAME_LEN + 1);
if (len >= LOG_MODULE_NAME_LEN - 1) {
    return FALSE;
}
```

> 证据来源: hiview_log.c:81-95

### 2. 隐私信息泄露

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 隐私标识绕过 | 使用 `%{private}` 但参数被错误处理 | 低 |
| 默认隐私行为 | 开发者忘记添加隐私标识 | 中 |
| 内存残留 | 隐私参数内存未清零 | 低 |

**现有防护**:
```c
// hilog_module.cpp:55-57 - 隐私处理
static const char PRIV_STR[10] = "<private>";

// 隐私参数替换
HilogString::Puts(
    showPriv ? PRIV_STR : HilogVector::GetStr(params, *(outParams->count)),
    outParams->logContent);
```

> 证据来源: hilog_module.cpp:55-57

### 3. 格式化字符串漏洞

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| %n 格式化符 | 使用 `%n` 可能导致内存写入 | **高** |
| 内存读取 | 格式化字符串访问非法内存 | 中 |
| 栈溢出 | 格式字符串参数过多 | 低 |

**现有防护**:
```c
// hilog_module.cpp:139-142 - 未知格式符处理
default:
    HilogString::Putc(format[*(outParams->pos)], outParams->logContent);
    break;
```

> 证据来源: hilog_module.cpp:139-142

**⚠️ 风险点**: 代码未显式过滤 `%n` 格式化符。

### 4. 资源耗尽攻击

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 日志洪泛 | 大量日志写入导致内存耗尽 | 中 |
| 文件系统 | 日志落盘耗尽存储空间 | 低 |
| 环形缓冲区 | Ring Buffer 溢出 | 低 |

**现有防护**:
```c
// hiview_log.c:121-124 - 限流检查
if (g_hiviewConfig.logSwitch == HIVIEW_FEATURE_OFF ||
    !CheckParameters(module, level) ||
    !LOG_IS_OUTPUT(module)) {
    return;
}
```

> 证据来源: hiview_log.c:121-124

**⚠️ 风险点**: 限流配置可能被绕过或未正确初始化。

### 5. 路径遍历 (日志落盘)

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 日志目录 | 配置的日志目录可能被利用 | **高** |
| 文件名 | 日志文件名生成逻辑 | 中 |

**现有防护**:
```gn
# BUILD.gn 中硬编码路径
hilog_lite_apphilogcat_log_dir = "/storage/data/log"
```

> 证据来源: services/apphilogcat/BUILD.gn:24

**⚠️ 风险点**: 日志目录路径应严格限制在可信路径范围内。

### 6. 竞态条件

| 攻击面 | 描述 | 风险等级 |
|--------|------|----------|
| 环形缓冲区 | 多线程并发写入 Ring Buffer | 中 |
| 全局配置 | g_hiviewConfig 并发访问 | 低 |
| 模块注册 | HiLogRegisterModule 并发调用 | 低 |

**现有防护**:
```c
// frameworks/featured/hiview_log.c:42
static atomic_int g_hiLogGetIdCallCount = 0;
```

> 证据来源: frameworks/featured/hiview_log.c:42

**⚠️ 风险点**: 部分关键操作未使用原子操作保护。

---

## 可被利用点汇总

### 高风险 (需优先修复)

| # | 风险描述 | 证据来源 | 触发条件 | 影响 | 修复建议 |
|---|----------|----------|----------|------|----------|
| 1 | 格式化字符串 `%n` 未过滤 | hilog_module.cpp:139-142 | 用户输入格式字符串包含 `%n` | 内存写入，可能导致代码执行 | 显式过滤 `%n` 格式化符 |
| 2 | 日志目录路径可配置 | apphilogcat/BUILD.gn:24 | 配置错误或恶意配置 | 写入任意路径 | 使用白名单路径验证 |

### 中风险 (建议改进)

| # | 风险描述 | 证据来源 | 触发条件 | 影响 | 修复建议 |
|---|----------|----------|----------|------|----------|
| 3 | 隐私标识可能遗漏 | log.h:174-182 | 开发者忘记添加 `%{private}` | 敏感信息泄露 | 提供 IDE 插件检查 |
| 4 | Ring Buffer 可能溢出 | 无明显保护 | 高速日志写入 | 日志丢失或损坏 | 增加溢出检测 |
| 5 | 并发写入 Ring Buffer | hiview_log.c:30 | 多线程同时写入 | 数据竞争 | 使用原子操作或锁 |

### 低风险 (可接受)

| # | 风险描述 | 证据来源 | 触发条件 | 影响 | 修复建议 |
|---|----------|----------|----------|------|----------|
| 6 | 模块 ID 校验绕过 | hiview_log.c:72 | 编译时级别低于运行时 | 低级别日志被过滤 | 确保编译配置一致 |
| 7 | 标签长度校验 | hiview_log.c:203-206 | 标签超过 32 字节 | 截断或拒绝 | 合理限制，文档说明 |
| 8 | 格式字符串长度限制 | hiview_log.c:263-266 | 格式字符串超长 | 拒绝或截断 | 合理限制，文档说明 |

---

## 安全建议

### 1. 紧急修复 (高风险)

#### 修复 %n 格式化符问题

```c
// 在 hilog_module.cpp 中添加格式化符检查
static bool IsValidFormatChar(char c) {
    const char *valid = "diouxXeEfgGaAcspn%";
    return strchr(valid, c) != NULL;
}

// 在 ParseLogContent 中
if (format[*(outParams->pos) + 1] == 'n') {
    // 拒绝或警告 %n
    HILOG_HILOGE("Format specifier %%n is not allowed");
    return;
}
```

#### 修复日志目录路径问题

```c
// 使用路径白名单验证
static const char *kAllowedLogDirs[] = {
    "/storage/data/log",
    "/var/log",
    NULL
};

bool IsLogDirAllowed(const char *path) {
    for (int i = 0; kAllowedLogDirs[i] != NULL; i++) {
        if (strncmp(path, kAllowedLogDirs[i], strlen(kAllowedLogDirs[i])) == 0) {
            return true;
        }
    }
    return false;
}
```

### 2. 长期改进 (中风险)

| 改进项 | 描述 | 优先级 |
|--------|------|--------|
| 日志脱敏 | 提供自动脱敏工具 | 高 |
| 沙箱隔离 | 应用日志写入隔离 | 中 |
| 完整性保护 | 日志防篡改签名 | 低 |
| 审计日志 | 日志访问审计 | 低 |

### 3. 开发规范

| 规范项 | 描述 |
|--------|------|
| 隐私优先 | 默认所有参数为隐私，使用 `%{public}` 显式公开 |
| 敏感信息 | 禁止在日志中直接打印密码、密钥等 |
| 格式化安全 | 使用编译期检查检测格式化漏洞 |

---

## 安全相关配置

| 配置项 | 安全影响 | 推荐值 |
|--------|----------|--------|
| `hilog_lite_disable_privacy_feature` | 禁用隐私保护 | **false** (保持启用) |
| `hilog_lite_limit_level_default` | 日志限流级别 | 根据需求设置 |
| `hilog_lite_disable_print_limit` | 禁用打印限流 | **false** (保持启用) |
| `hilog_lite_apphilogcat_log_dir` | 日志目录 | 使用可信路径 |

---

## 结论

### 总体评估

| 指标 | 评级 | 说明 |
|------|------|------|
| 代码质量 | 良好 | 基础安全措施已实现 |
| 隐私保护 | 良好 | 隐私标识机制有效 |
| 输入校验 | 中等 | 基础校验存在，缺少 %n 过滤 |
| 竞态安全 | 中等 | 部分使用原子操作 |

### 建议优先级

1. **立即处理**: %n 格式化符过滤
2. **短期处理**: 日志目录路径验证
3. **长期改进**: 沙箱隔离、完整性保护

---

## 相关文档

- [概览](01_Overview.md)
- [架构设计](02_Architecture.md)
- [Native API](03_Native_API.md)
- [GN Targets](05_GN_Targets.md)
