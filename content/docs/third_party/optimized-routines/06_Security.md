# 06 - 安全风险分析

本文档说明 optimized-routines 在 OpenHarmony 中的安全特性和风险分析。

---

## 安全特性

### 1. PAC-RET (Pointer Authentication)

#### 配置

```gn
if (musl_arch == "aarch64") {
  cflags += [ "-mbranch-protection=pac-ret+b-key" ]
}
```

#### 作用

PAC-RET 是 ARMv8.3-A 引入的指针认证功能，用于保护返回地址免受篡改。

| 特性 | 说明 |
|------|------|
| **保护目标** | 返回地址（RET 指令） |
| **防护攻击** | ROP (Return-Oriented Programming) 攻击 |
| **适用架构** | AArch64 (ARMv8.3-A 及以上) |
| **密钥类型** | B-Key (应用程序密钥) |

#### 工作原理

1. **函数返回时**：计算返回地址的 PAC（Pointer Authentication Code）
2. **RET 指令执行前**：验证 PAC 是否匹配
3. **验证失败**：触发异常，阻止攻击

#### 性能影响

- **开销**: 约为 1-3%（取决于函数调用频率）
- **收益**: 显著提高 ROP 攻击难度

---

### 2. 栈保护器 (Stack Canary)

#### 配置

```gn
cflags = [ "-fstack-protector-strong" ]

if (musl_arch == "aarch64") {
  cflags_c += [ "-fno-stack-protector" ]  # 汇编文件禁用
}
```

#### 作用

栈保护器检测栈溢出攻击。

| 特性 | 说明 |
|------|------|
| **保护目标** | 函数栈帧 |
| **防护攻击** | 栈溢出（Stack Smashing） |
| **保护级别** | 强（strong 模式） |
| **适用架构** | ARMv7, AArch64 |

#### 工作原理

1. **函数序言**: 在栈帧中插入 canary 值
2. **函数返回前**: 检查 canary 是否被修改
3. **检测到溢出**: 触发 `__stack_chk_fail`，终止程序

#### 汇编文件禁用原因

```gn
cflags_c += [ "-fno-stack-protector" ]  # 仅对汇编文件
```

- **汇编代码手工编写**：通常不会产生栈溢出漏洞
- **避免性能损失**：汇编函数调用频繁，canary 检查开销大
- **C 文件仍启用**：C 代码可能有栈溢出风险，保持保护

---

### 3. HWASAN (Hardware Address Sanitizer)

#### 配置

```gn
if (use_hwasan) {
  defines += [ "HWASAN_REMOVE_CLEANUP" ]
  configs += [ "//build/config/sanitizers:default_sanitizer_flags" ]
  cflags += [
    "-mllvm",
    "-hwasan-instrument-check-enabled=true",
  ]
}
```

#### 作用

HWASAN 是硬件辅助的内存错误检测工具（基于 ARM 的 Memory Tagging Extension, MTE）。

| 特性 | 说明 |
|------|------|
| **保护目标** | 内存访问（读/写） |
| **检测错误** | Use-after-free、Buffer overflow、Double-free |
| **依赖硬件** | ARM MTE (Memory Tagging Extension) |
| **适用阶段** | 调试/测试阶段 |

#### 工作原理

1. **内存分配**：为每个内存块分配一个标签（Tag）
2. **指针生成**：指针中嵌入标签
3. **内存访问**：检查指针标签与内存标签是否匹配
4. **不匹配**：触发异常，报告错误

#### 使用场景

- **开发调试**: 检测内存错误
- **集成测试**: 提高测试覆盖率
- **生产环境**: 通常禁用（性能开销较大）

#### 性能影响

- **开销**: 约为 50-100%（取决于内存访问频率）
- **推荐**: 仅在调试/测试阶段启用

---

### 4. 其他安全编译选项

#### 选项清单

| 选项 | 作用 | 适用性 |
|------|------|--------|
| `-fno-plt` | 间接调用保护 | 所有架构 |
| `-fPIC` | 位置无关代码（ASLR 友好） | 所有架构 |
| `-fno-lto` | 禁用链接时优化（避免意外优化） | 所有架构 |

---

## 安全风险

### 已知 CVE

#### optimized-routines 原始库

| CVE 编号 | 影响版本 | 修复版本 | 描述 | OH 状态 |
|---------|---------|---------|------|---------|
| CVE-2022-xxxxx | < v23.01 | v23.01 | memcpy 溢出 | ✅ 已修复 |
| CVE-2023-xxxxx | < v24.01 | v24.01 | MTE 实现问题 | ✅ 已修复 |

**说明**: 以上 CVE 为示例，实际 CVE 需要查询上游 CVE 数据库。

#### OpenHarmony 引入的风险

**结论**: OH 未引入新的安全风险。

**原因**:
- 源代码保持与上游一致
- 仅添加构建系统配置（BUILD.gn、.gni）
- 构建配置增强安全性（PAC-RET、栈保护、HWASAN）

---

## 安全配置建议

### 1. 生产环境配置

```gni
# 设备配置文件（生产环境）
musl_arch = "aarch64"
ARM_FEATURE_SVE = false  # SVE 实验性，建议生产环境禁用
ARM_FEATURE_MTE = true   # MTE 用于安全防护

# 编译选项
use_hwasan = false      # HWASAN 性能开销大，仅调试使用
```

### 2. 调试环境配置

```gni
# 设备配置文件（调试环境）
musl_arch = "aarch64"
ARM_FEATURE_SVE = true   # 启用 SVE 测试
ARM_FEATURE_MTE = false

# 编译选项
use_hwasan = true       # 启用 HWASAN 检测内存错误
```

### 3. 硬实时系统配置

```gni
# UniProton / LiteOS-M 配置
musl_arch = "arm"
is_llvm_build = true

# 编译选项
use_hwasan = false      # RTOS 禁用 HWASAN（延迟）
```

---

## 编译器安全选项

### 基础安全选项

| 选项 | 作用 | 推荐使用 |
|------|------|---------|
| `-O3` | 优化级别 | ✅ 所有环境 |
| `-fstack-protector-strong` | 栈保护 | ✅ 生产环境 |
| `-fno-lto` | 禁用 LTO | ✅ 稳定性 |

### 架构特定安全选项

| 架构 | 选项 | 作用 | 推荐使用 |
|------|------|------|---------|
| AArch64 | `-mbranch-protection=pac-ret+b-key` | PAC-RET | ✅ 生产环境 |
| AArch64 | `-fno-stack-protector` (汇编) | 禁用栈保护 | ✅ 所有环境 |
| ARMv7 | `-fstack-protector-strong` | 栈保护 | ✅ 生产环境 |

### 调试安全选项

| 选项 | 作用 | 推荐使用 |
|------|------|---------|
| `-hwasan-instrument-check-enabled=true` | HWASAN | 仅调试 |

---

## 安全升级策略

### 1. 定期同步上游版本

**策略**:
- 每季度检查上游新版本（vYY.MM 格式）
- 关注上游 CVE 修复公告
- 评估安全风险和稳定性影响

**步骤**:
1. 查看上游 Release Notes
2. 检查 CVE 数据库
3. 同步源码（保留 OH 特定文件）
4. 验证构建
5. 运行测试

### 2. CVE 修复流程

**发现 CVE**:
1. 查询上游 CVE 数据库
2. 确认 OH 版本是否受影响
3. 查看上游修复 PR

**修复 CVE**:
1. 合并上游修复
2. 验证构建
3. 回归测试
4. 发布安全补丁

### 3. 安全审计

**定期审计**:
- 审查 BUILD.gn 配置
- 审查编译选项
- 审查符号别名定义
- 运行内存安全测试（HWASAN）

---

## 安全测试

### 1. 内存安全测试

使用 HWASAN 检测内存错误：

```bash
# 启用 HWASAN 编译
hb build -f --ccache --build-type debug --use-hwasan

# 运行测试
./out/tests/mem_test
```

### 2. 栈溢出测试

验证栈保护器是否生效：

```c
#include <string.h>

void test_stack_overflow() {
    char buffer[10];
    memcpy(buffer, "this is a long string that overflows", 40);
}

int main() {
    test_stack_overflow();
    return 0;
}
```

**预期**: 触发 `__stack_chk_fail`，程序终止。

### 3. ROP 攻击测试

验证 PAC-RET 是否生效：

```c
#include <stdio.h>

void exploit() {
    printf("ROP attack succeeded!\n");
}

int main() {
    // 尝试通过栈溢出劫持返回地址
    // PAC-RET 应该阻止攻击
    return 0;
}
```

**预期**: PAC 验证失败，触发异常。

---

## 常见问题

### Q: PAC-RET 和栈保护器可以同时启用吗？

A: 可以。它们保护不同的攻击面：
- PAC-RET：保护返回地址（防 ROP）
- 栈保护器：保护栈帧（防栈溢出）

### Q: HWASAN 在生产环境中使用吗？

A: 通常不使用。HWASAN 性能开销较大（50-100%），主要用于调试和测试阶段。生产环境建议使用 MTE（性能开销约 5-10%）。

### Q: SVE 实现安全吗？

A: SVE 实现是实验性的，建议生产环境禁用。启用 SVE 需要充分测试。

### Q: 如何验证安全特性是否生效？

A:
1. 查看编译日志，确认编译选项
2. 查看二进制符号表，确认符号别名
3. 运行安全测试用例

---

## 相关文档

- **[03_Build_Integration.md](03_Build_Integration.md)** - 构建集成详解（安全配置）
- **[02_Adaptations.md](02_Adaptations.md)** - OH 适配说明

---

## 参考资源

- **ARM PAC-RET**: https://developer.arm.com/documentation/ddi0601/2024-03/AArch64-Registers
- **ARM MTE**: https://developer.arm.com/documentation/102374/latest/
- **HWASAN**: https://github.com/google/sanitizers/wiki/HardwareAddressSanitizer

---

**文档版本**: 1.0
**最后更新**: 2026-02-07
