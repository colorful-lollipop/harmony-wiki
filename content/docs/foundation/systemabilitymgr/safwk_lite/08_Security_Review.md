# 安全风险评审

## 评审概述

本章节对 `safwk_lite` 组件进行安全风险评审，识别潜在攻击面、信任边界及可被利用点。

## 评审范围

| 范围 | 说明 |
|------|------|
| 源码 | `src/main.c`, `BUILD.gn` |
| 构建配置 | feature flags, 依赖配置 |
| 运行时行为 | 进程启动、服务初始化、IPC 通信 |

**不包含范围**:
- 子服务内部实现（abilityms, bundlems, dmsfwk_lite 等）
- 测试代码
- 第三方依赖库（libc, libstdc++ 等）

## 攻击面分析

### 攻击面清单

| 入口点 | 类型 | 风险等级 | 说明 |
|--------|------|---------|------|
| 命令行参数 | 输入 | **低** | `main(int argc, char * const argv[])` |
| 环境变量 | 配置 | **低** | 可被修改但影响有限 |
| 配置文件 | 资源 | **中** | SA 配置文件的路径和内容 |
| Samgr IPC | 通信 | **高** | 跨进程通信的安全边界 |
| 信号处理 | 系统调用 | **低** | `pause()` 等待信号 |

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                      信任边界                               │
├─────────────────────────────────────────────────────────────┤
│  内部 (可信)                                                 │
│  ├── main.c 逻辑                                            │
│  ├── Samgr Lite 框架                                        │
│  └── 已注册的子服务                                          │
├─────────────────────────────────────────────────────────────┤
│  边界                                                       │
│  ├── IPC 消息验证 (由 Samgr/IPC 层负责)                     │
│  ├── 配置解析校验                                           │
│  └── 权限检查 (由 PMS 层负责)                               │
├─────────────────────────────────────────────────────────────┤
│  外部 (不可信)                                               │
│  ├── 用户态应用                                              │
│  ├── 网络输入 (若有)                                        │
│  └── 文件系统 (配置文件)                                     │
└─────────────────────────────────────────────────────────────┘
```

## 识别风险点

### 风险 1：命令行参数注入

**证据**: `src/main.c:38-46`

```c
int main(int argc, char * const argv[])
{
    OHOS_SystemInit();
    ...
}
```

| 属性 | 值 |
|------|-----|
| 位置 | `main.c:38` |
| 风险类型 | 输入验证不足 |
| 严重程度 | 低 |
| 可利用性 | 低 |

**分析**:
- `argc`/`argv` 未被校验直接使用
- 但当前版本未对 argv 进行解析

**修复建议**:
```c
// 建议在生产版本中添加参数校验
if (argc > 1) {
    // 验证参数格式和长度
    if (strlen(argv[1]) > MAX_PARAM_LEN) {
        fprintf(stderr, "Invalid parameter length\n");
        return -1;
    }
}
```

---

### 风险 2：弱符号覆盖风险

**证据**: `src/main.c:30`

```c
void __attribute__((weak)) OHOS_SystemInit(void)
{
    SAMGR_Bootstrap();
}
```

| 属性 | 值 |
|------|-----|
| 位置 | `main.c:30` |
| 风险类型 | 符号覆盖 |
| 严重程度 | 中 |
| 可利用性 | 中 |

**分析**:
- 使用 GCC 弱符号机制，允许其他目标文件覆盖实现
- 攻击者可能通过恶意代码覆盖 `OHOS_SystemInit()` 植入后门

**修复建议**:
1. 使用链接器脚本限制符号可见性
2. 在构建时校验最终链接的符号来源
3. 添加启动完整性检查

```c
// 方案：添加完整性校验
#define BOOTSTRAP_SYMBOL 0xDEADBEEF
static volatile uint32_t g_bootFlag = 0;

void OHOS_SystemInit(void)
{
    g_bootFlag = BOOTSTRAP_SYMBOL;
    SAMGR_Bootstrap();
}

int main(void)
{
    OHOS_SystemInit();
    if (g_bootFlag != BOOTSTRAP_SYMBOL) {
        // 启动失败
        return -1;
    }
    ...
}
```

---

### 风险 3：无限循环无看门狗

**证据**: `src/main.c:57-61`

```c
while (1) {
    (void)pause();  // pause only returns -1
}
```

| 属性 | 值 |
|------|-----|
| 位置 | `main.c:57` |
| 风险类型 | 拒绝服务 |
| 严重程度 | 低 |
| 可利用性 | 低 |

**分析**:
- 进程进入无限循环后无超时退出机制
- 若 Samgr 初始化失败，进程将永久挂起
- 系统需依赖外部看门狗或 init 重启

**修复建议**:
```c
// 添加超时监控
#define STARTUP_TIMEOUT_MS 5000
struct timespec ts = {STARTUP_TIMEOUT_MS / 1000, 
                      (STARTUP_TIMEOUT_MS % 1000) * 1000000L};

if (clock_nanosleep(CLOCK_MONOTONIC, TIMED_ABSTIME, &ts, NULL) == 0) {
    // 超时未完成初始化
    printf("[safwk_lite] Startup timeout!\n");
    return -1;
}
```

---

### 风险 4：调试符号泄露

**证据**: `BUILD.gn:42`

```gn
ldflags = [
  "-lstdc++",
  "-Wl,-Map=foundation.map",  // 生成 map 文件
]
```

| 属性 | 值 |
|------|-----|
| 位置 | `BUILD.gn:42` |
| 风险类型 | 信息泄露 |
| 严重程度 | 低 |
| 可利用性 | 低 |

**分析**:
- Map 文件包含完整符号信息
- 有助于逆向分析和漏洞利用

**修复建议**:
```gn
// 仅在调试版本生成 map 文件
if (is_debug_build) {
  ldflags += [ "-Wl,-Map=foundation.map" ]
}
```

---

### 风险 5：条件编译后门

**证据**: `BUILD.gn:16-21`

```gn
declare_args() {
  enable_timertask = false
  safwk_lite_feature_enable_abilityms = true
  ...
}
```

| 属性 | 值 |
|------|-----|
| 位置 | `BUILD.gn:16-21` |
| 风险类型 | 配置篡改 |
| 严重程度 | 低 |
| 可利用性 | 低 |

**分析**:
- Feature 开关可被恶意修改
- 可能导致未经授权的服务启用

**修复建议**:
1. 在 release 版本中硬编码关键 feature 状态
2. 使用签名配置验证 feature 配置完整性

## 安全建议汇总

| 优先级 | 建议 | 状态 |
|--------|------|------|
| 高 | 添加命令行参数校验 | 建议实施 |
| 中 | 限制弱符号覆盖风险 | 建议实施 |
| 中 | 添加启动超时监控 | 建议实施 |
| 低 | 生产版本移除 Map 文件 | 可选 |
| 低 | 配置文件签名校验 | 可选 |

## 相关安全组件

| 组件 | 作用 |
|------|------|
| `permission_lite` | 权限管理服务 (PMS) |
| `ipc_auth` | IPC 认证 |
| `samgr_lite` | 服务注册与 IPC 路由 |

## 参考标准

- [OWASP IoT Top 10](https://owasp.org/www-project-internet-of-things/)
- [CWE-119: Improper Restriction of Operations within Memory Buffer](https://cwe.mitre.org/data/definitions/119.html)
- [CWE-754: Improper Check for Unusual or Exceptional Conditions](https://cwe.mitre.org/data/definitions/754.html)
