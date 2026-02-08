# 02 - Patch 详细分析

## 2.1 Patch 清单概览

### 结论：本库无任何 Patch 文件

经过全面搜索，**CMSIS 库（third_party/cmsis）不包含任何 Patch 文件**。

```bash
# 搜索结果
$ find . -name "*.patch" -o -name "patches" -type d
# 无输出 - 未找到任何 patch 文件或目录
```

### Patch 统计表

| Patch 类型 | 数量 | 文件路径 |
|------------|------|----------|
| Bugfix Patch | 0 | - |
| Feature Patch | 0 | - |
| OH 适配 Patch | 0 | - |
| 性能优化 Patch | 0 | - |
| **总计** | **0** | - |

---

## 2.2 无 Patch 的设计原因分析

### 原因一：CMSIS 是纯粹的头文件库

CMSIS 的本质是**接口定义**，而非功能实现：

| 组件 | 内容 | 是否需要 Patch |
|------|------|----------------|
| **CMSIS-Core** | 处理器寄存器定义、内联汇编函数 | 否（纯定义） |
| **CMSIS-RTOS2** | RTOS API 头文件（函数声明、结构体定义） | 否（纯接口） |

**特点**：
- 所有功能通过 `static inline` 或宏实现
- 无 `.c` 源文件需要编译
- 无运行时库依赖

### 原因二：OH 适配采用外部实现模式

OpenHarmony 的 CMSIS 适配工作**完全在外部目录完成**：

```
third_party/cmsis/          ← 本仓库：只有头文件，无 Patch
├── CMSIS/Core/Include/     ← 寄存器定义
└── CMSIS/RTOS2/Include/    ← API 接口定义
        ↓
kernel/liteos_m/kal/cmsis/  ← 外部：适配实现
├── cmsis_liteos2.c         ← LiteOS-M 适配实现
└── BUILD.gn                ← 模块构建配置
```

**适配层职责**：
- 实现 `cmsis_os2.h` 中声明的所有 API
- 将 CMSIS-RTOS2 调用转换为 LiteOS-M 内核调用
- 处理 OH 特有的扩展需求

### 原因三：保持与上游同步的便利性

| 策略 | 优势 |
|------|------|
| **无 Patch** | 可直接替换头文件升级上游版本 |
| **外部适配** | 适配逻辑独立于上游演进 |
| **接口标准** | ARM 维护标准，OH 专注实现 |

---

## 2.3 类比：CMSIS 在 OH 中的角色

### 类比例子：C 标准库头文件

| 类比 | CMSIS | C 标准库 |
|------|-------|----------|
| **头文件** | `core_cm*.h`, `cmsis_os2.h` | `stdio.h`, `stdlib.h` |
| **实现** | 外部适配层 (`cmsis_liteos2.c`) | libc 实现 (`newlibc`, `musl`) |
| **是否需要 Patch 头文件** | 否 | 否 |
| **定制在哪里** | 适配实现 | libc 配置 |

### 类比：Java 接口与实现

```java
// CMSIS：只定义接口（头文件）
public interface CMSIS_RTOS2 {
    osThreadId_t osThreadNew(...);
    osStatus_t osDelay(...);
}

// OH 适配：提供具体实现（外部目录）
public class LiteOSM_Adapter implements CMSIS_RTOS2 {
    @Override
    public osThreadId_t osThreadNew(...) {
        return LOS_TaskCreate(...);
    }
}
```

---

## 2.4 相关代码分析

### 典型头文件内容示例

#### CMSIS-Core (`core_cm4.h` 片段)

```c
// 处理器寄存器定义 - 纯数据结构
typedef struct {
  __IOM uint32_t ISER[8U];               /*!< Offset: 0x000 (R/W)  Interrupt Set Enable Register */
        uint32_t RESERVED0[24U];
  __IOM uint32_t ICER[8U];               /*!< Offset: 0x080 (R/W)  Interrupt Clear Enable Register */
  // ...
}  NVIC_Type;

// 内联函数 - 直接编译为处理器指令
__STATIC_INLINE void __NVIC_EnableIRQ(IRQn_Type IRQn) {
  if ((int32_t)(IRQn) >= 0) {
    NVIC->ISER[(((uint32_t)IRQn) >> 5UL)] = (uint32_t)(1UL << (((uint32_t)IRQn) & 0x1FUL));
  }
}
```

**分析**：
- 只有寄存器地址定义和内联函数
- 无逻辑分支需要针对 OH 修改
- 无外部依赖

#### CMSIS-RTOS2 (`cmsis_os2.h` 片段)

```c
// 函数声明 - 无实现
osThreadId_t osThreadNew(osThreadFunc_t func, void *argument, const osThreadAttr_t *attr);
uint32_t osThreadFlagsSet(osThreadId_t thread_id, uint32_t flags);
osStatus_t osDelay(uint32_t ticks);

// 结构体定义
typedef struct {
  const char                   *name;   ///< Name of the thread
  uint32_t                 attr_bits;   ///< Attribute bits
  void                      *cb_mem;    ///< Memory for control block
  uint32_t                   cb_size;   ///< Size of provided memory for control block
  void                   *stack_mem;    ///< Memory for stack
  uint32_t                stack_size;   ///< Size of stack
  osPriority_t              priority;   ///< Initial thread priority
  // ...
} osThreadAttr_t;
```

**分析**：
- 只有 API 声明，无具体实现
- 实现由 OH 的 `cmsis_liteos2.c` 提供
- 头文件定义标准，无需修改

---

## 2.5 OH 定制化实现位置

虽然 CMSIS 库本身无 Patch，但 OH 的**适配实现**包含定制化代码：

### 主要适配文件

| 文件路径 | 功能 | 定制化内容 |
|----------|------|------------|
| `kernel/liteos_m/kal/cmsis/cmsis_liteos2.c` | RTOS 适配实现 | 完整的 CMSIS-RTOS2 → LiteOS-M 映射 |
| `device/soc/*/kal/cmsis/cmsis_liteos.c` | SoC 特定适配 | 芯片启动代码、中断处理 |

### 定制化示例：优先级映射

```c
// kernel/liteos_m/kal/cmsis/cmsis_liteos2.c

/* LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO <---> osPriorityNormal */
#define LOS_PRIORITY(cmsisPriority) (LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO - ((cmsisPriority) - osPriorityNormal))
#define CMSIS_PRIORITY(losPriority) (osPriorityNormal + (LOSCFG_BASE_CORE_TSK_DEFAULT_PRIO - (losPriority)))
```

**说明**：
- 将 CMSIS-RTOS2 的优先级范围映射到 LiteOS-M 内部优先级
- 这是 OH 特有的映射关系，不属于 CMSIS 标准
- 因此实现在外部适配层，而非 Patch CMSIS

### 定制化示例：内存池实现

```c
// kernel/liteos_m/kal/cmsis/cmsis_liteos2.c

// OH 特定的内存池控制块扩展
typedef struct {
    LOS_MEMBOX_INFO poolInfo;
    void            *poolBase;
    uint32_t        poolSize;
    uint32_t        status;
    const char      *name;
} MemPoolCB;
```

**说明**：
- CMSIS-RTOS2 标准只定义接口，不规定实现细节
- OH 使用 LiteOS-M 的内存盒（MemBox）实现内存池
- 实现细节完全在适配层控制

---

## 2.6 升级建议

### 升级上游 CMSIS 版本

| 步骤 | 操作 | 风险 |
|------|------|------|
| 1 | 直接替换头文件 | 低（无 Patch） |
| 2 | 检查 API 兼容性 | 中（需验证适配层） |
| 3 | 运行 XTS 测试套件 | - |
| 4 | 验证各 SoC 编译 | 低 |

### 升级检查清单

- [ ] 替换 `CMSIS/Core/Include` 头文件
- [ ] 替换 `CMSIS/RTOS2/Include` 头文件
- [ ] 检查是否有新增 API（`cmsis_os2.h`）
- [ ] 如有新增 API，在 `kal/cmsis` 中实现
- [ ] 检查是否有废弃 API 警告
- [ ] 运行 XTS 测试：`test/xts/acts/kernel_lite/kernelcmsis_hal`
- [ ] 验证芯片编译：`device/soc/*/kal/cmsis`

---

## 2.7 总结

### 核心结论

| 问题 | 答案 |
|------|------|
| CMSIS 是否有 Patch？ | **否**，零 Patch |
| OH 定制化在哪里？ | **外部适配层** (`kal/cmsis`) |
| 升级是否容易？ | **是**，直接替换头文件即可 |
| 适配层是否需要维护？ | **是**，随上游 API 更新 |

### 设计优势

1. **维护成本低**：无需管理 Patch 冲突
2. **升级风险低**：头文件替换即可
3. **架构清晰**：接口定义与实现分离
4. **可测试性好**：适配层可独立测试

### 相关文档

- [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 详细依赖关系
- `kernel/liteos_m/kal/cmsis/cmsis_liteos2.c` - 适配实现源码

---

*文档版本: 1.0*  
*最后更新: 2025-02-08*
