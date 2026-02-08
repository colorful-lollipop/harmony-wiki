# Bootstrap_Lite - 安全风险评审

## 评审范围

本安全评审覆盖 `bootstrap_lite` 组件的所有源代码：

| 文件 | 评审范围 |
|------|----------|
| services/source/system_init.c | 系统初始化入口 |
| services/source/bootstrap_service.c | Bootstrap 服务实现 |
| services/source/bootstrap_service.h | 初始化宏定义 |
| services/source/core_main.h | 核心初始化宏 |

## 威胁模型

### 信任边界

```
[不可信区域] → [可信硬件] → [可信启动链] → [bootstrap_lite]
                                              ↓
                                    [高度可信：系统启动早期]
```

**信任边界说明**:
1. **可信硬件根**: 系统启动基于可信硬件 (ROM bootloader)
2. **可信启动链**: bootstrap_lite 在启动链后期执行
3. **高度可信**: bootstrap_lite 执行时操作系统尚未完全初始化，用户态不存在

### 攻击面分析

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| 链接器脚本段注入 | 低 | 依赖链接器安全性 |
| 初始化函数指针遍历 | 低 | 函数指针在可信内存 |
| SAMGR 消息处理 | 低 | 早期无外部消息源 |
| 配置篡改 | 低 | 构建期静态配置 |

## 已识别风险

### 风险 1: 链接器脚本段名称冲突

**证据**: `path:services/source/bootstrap_service.h:23-24`

```c
#define APP_NAME(name, step) ".zinitcall.app." #name #step ".init"
#define MODULE_NAME(name, step) ".zinitcall." #name #step ".init"
```

**风险描述**: 链接器脚本段名称使用硬编码字符串，如果其他模块使用相同段名称，可能导致初始化函数被错误合并或覆盖。

**触发条件**:
1. 其他模块定义了相同格式的链接器脚本段
2. 链接器将多个模块的相同段合并
3. 初始化顺序被打乱

**潜在影响**:
- 初始化顺序错误导致系统不稳定
- 关键初始化函数被跳过

**风险等级**: 低

**修复建议**:
1. 使用唯一命名前缀（如 `__bootstrap_app_`）
2. 在链接器配置中显式指定段边界
3. 添加链接器脚本验证

---

### 风险 2: 空指针解引用风险

**证据**: `path:services/source/core_main.h:46-50`

```c
#define SYS_BEGIN(name, step)                                 \
    ({  extern InitCall __zinitcall_sys_##name##_start;       \
        InitCall *initCall = &__zinitcall_sys_##name##_start; \
        (initCall);                                           \
    })
```

**风险描述**: 如果链接器脚本未正确生成 `__zinitcall_*_start` 符号，`&__zinitcall_sys_*_start` 可能产生意外行为。

**触发条件**:
1. 链接器配置错误导致符号未生成
2. 链接脚本未正确处理空段

**潜在影响**:
- 程序崩溃 (Crash)
- 启动失败

**风险等级**: 低

**修复建议**:
1. 添加符号存在性检查
2. 使用条件编译确保链接器配置正确
3. 添加启动验证逻辑

---

### 风险 3: 初始化函数返回类型不一致

**证据**: `path:services/source/core_main.h:31`

```c
(*initcall)();  // 调用初始化函数
```

**风险描述**: InitCall 被定义为函数指针类型，但初始化函数的实际返回类型可能不统一。虽然 C 语言允许忽略返回值，但可能导致：
1. 错误的初始化被忽略
2. 栈布局不一致

**触发条件**:
1. 初始化函数返回非预期值
2. 调用约定不匹配

**潜在影响**:
- 未定义行为
- 启动不稳定

**风险等级**: 低

**修复建议**:
1. 定义明确的 InitCall 类型（建议 `void (*)(void)`）
2. 在代码规范中强制约束
3. 添加静态分析检查

---

### 风险 4: Bootstrap Service 消息队列溢出

**证据**: `path:services/source/bootstrap_service.c:87`

```c
TaskConfig config = {LEVEL_HIGH, PRI_NORMAL, 0x800, 20, SHARED_TASK};
```

**风险描述**: Bootstrap Service 使用固定深度 20 的消息队列，如果多个组件在启动早期同时发送消息，可能导致队列溢出。

**触发条件**:
1. 多个组件同时发送 BOOT_* 消息
2. 消息处理速度跟不上发送速度

**潜在影响**:
- 消息丢失
- 启动卡住

**风险等级**: 低

**缓解因素**:
1. 系统启动早期消息源有限
2. SHARED_TASK 模式共享系统资源

**修复建议**:
1. 监控消息队列使用情况
2. 考虑使用动态队列或优先级队列

---

### 风险 5: 条件编译导致的边界检查缺失

**证据**: `path:services/source/BUILD.gn:24-28`

```gn
if (ohos_kernel_type == "liteos_m" || ohos_kernel_type == "uniproton") {
  include_dirs += []
} else if (ohos_kernel_type == "liteos_a" || ohos_kernel_type == "linux") {
  include_dirs += [ "//third_party/bounds_checking_function/include" ]
}
```

**风险描述**: bounds_checking_function 仅在 liteos_a/linux 启用，在 liteos_m/uniproton 上缺乏边界检查。

**触发条件**:
1. 在 liteos_m/uniproton 上运行
2. 缓冲区操作越界

**潜在影响**:
- 缓冲区溢出漏洞
- 内存损坏

**风险等级**: 中

**缓解因素**:
1. 轻量系统设备资源受限，攻击价值有限
2. bootstrap_lite 本身不执行复杂缓冲区操作

**修复建议**:
1. 为 liteos_m/uniproton 提供等效的边界检查
2. 或在运行时添加断言检查

## 安全假设

本组件基于以下安全假设：

| 假设 | 说明 |
|------|------|
| 硬件根信任 | 启动过程基于可信硬件 |
| 链接器正确性 | 链接器正确生成脚本段符号 |
| 构建环境可信 | 构建过程未被篡改 |
| 初始化函数可信 | 所有初始化函数来源可信 |

## 已知安全特性

### 1. 静态初始化
初始化在系统启动早期完成，不涉及运行时加载外部代码。

### 2. 编译期绑定
所有初始化函数在编译期确定，无运行时动态加载。

### 3. 阶段化执行
初始化分阶段执行，降低单点故障风险。

## 权限与能力

### 当前状态: **不涉及**

**证据**: `path:services/source/` + `grep` search for `permission|access token|uid|bundle|signature`

**说明**: Bootstrap_Lite 是系统启动引导组件，在用户态权限系统初始化之前执行，因此：
- 不涉及权限检查
- 不涉及访问令牌验证
- 不涉及签名校验
- 以最高权限运行（内核态/系统态）

## 审计结论

### 总体评估

Bootstrap_Lite 组件**安全风险较低**，原因如下：

1. **执行时机早**: 在用户态和权限系统初始化之前执行，无外部攻击面
2. **静态绑定**: 所有初始化函数编译期确定，无运行时加载
3. **代码量小**: 仅约 300 行核心代码，审计成本低
4. **无复杂逻辑**: 不涉及网络、文件、用户输入等外部数据

### 建议优先级

| 优先级 | 风险 | 建议 |
|--------|------|------|
| P3 | 链接器脚本段名称冲突 | 中期规划：使用唯一前缀 |
| P3 | 边界检查缺失 | 中期规划：统一边界检查 |
| P4 | 空指针风险 | 低优先级：已由链接器保证 |
| P4 | 消息队列溢出 | 低优先级：实际风险小 |

## 相关文档

- **[架构设计](01_Architecture.md)** - 组件关系图
- **[初始化机制](03_Initialization.md)** - 启动流程详解
- **[构建配置](02_Build.md)** - GN 构建说明
