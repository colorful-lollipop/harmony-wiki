# 阅读路线建议

## 根据角色的阅读建议

### 如果你是一位...

#### 1. OpenHarmony 开发者（应用/系统服务开发）

**建议阅读顺序**：
1. [01_Overview.md](./01_Overview.md) - 了解 CMSIS 是什么
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解谁会用到 CMSIS

**关键要点**：
- 通常不需要直接引用 CMSIS
- 如需使用 RTOS 功能，通过 `kal` 框架间接使用

#### 2. 内核/驱动开发者

**建议阅读顺序**：
1. [01_Overview.md](./01_Overview.md) - 了解 CMSIS 架构
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解如何引入 CMSIS
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 深入理解适配层实现
4. [02_Patches.md](./02_Patches.md) - 了解是否有 OH 特定修改

**关键要点**：
- 引入 CMSIS：`import("//third_party/cmsis/cmsis.gni")`
- 使用 `CMSIS_INCLUDE_DIRS` 获取包含路径
- 避免直接修改本库，适配工作在外部完成

#### 3. 移植工程师（新芯片适配）

**建议阅读顺序**：
1. [01_Overview.md](./01_Overview.md) - 了解 CMSIS-Core 作用
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 参考现有 SoC 适配模式
3. `device/soc/hisilicon/hi3861v100/hi3861_adapter/kal/cmsis/` - 参考实现

**关键要点**：
- CMSIS-Core 提供处理器寄存器定义
- 芯片启动代码需配合 CMSIS 头文件使用
- RTOS 适配参考 `cmsis_liteos2.c` 实现模式

#### 4. 维护者/升级负责人

**建议阅读顺序**：
1. [02_Patches.md](./02_Patches.md) - 确认无 Patch
2. [03_Build_Integration.md](./03_Build_Integration.md) - 了解构建配置
3. [06_Security.md](./06_Security.md) - 安全检查
4. 运行 XTS 测试套件验证

**关键要点**：
- 升级简单：直接替换头文件即可
- 重点验证 `kal/cmsis` 适配层兼容性
- 关注 ARM 官方安全公告

---

## 快速参考

### 常见问题

| 问题 | 答案位置 |
|------|----------|
| 如何引入 CMSIS？ | [03_Build_Integration.md](./03_Build_Integration.md) |
| 有哪些 OH 特定修改？ | [02_Patches.md](./02_Patches.md) |
| 谁在依赖这个库？ | [04_Usage_in_OH.md](./04_Usage_in_OH.md) |
| 升级版本要注意什么？ | [06_Security.md](./06_Security.md) |

### 关键文件索引

| 文件路径 | 说明 |
|----------|------|
| `cmsis.gni` | OH 构建配置入口 |
| `CMSIS/Core/Include/core_cm*.h` | Cortex-M 内核头文件 |
| `CMSIS/RTOS2/Include/cmsis_os2.h` | RTOS2 API 头文件 |
| `kernel/liteos_m/kal/cmsis/cmsis_liteos2.c` | OH 适配实现（外部） |

---

## 深入阅读

### 外部参考文档

- [ARM CMSIS 官方文档](https://arm-software.github.io/CMSIS_6/)
- [CMSIS-RTOS2 API 参考](https://arm-software.github.io/CMSIS_6/latest/RTOS2/index.html)
- [LiteOS-M 内核文档](https://gitee.com/openharmony/kernel_liteos_m)

### 相关 Wiki

- `kernel/liteos_m` Wiki - 内核实现细节
- `device/soc` 各芯片 Wiki - 芯片特定适配
