# API 使用说明

## OH 特定 API 使用

### Panda 内存分配器集成

VIXL 在 OH 中可以与 Panda 内存分配器集成，用于优化方舟编译器的内存使用：

```cpp
// 启用 Panda 分配器
#define VIXL_USE_PANDA_ALLOC
#include "utils-vixl.h"

// 使用 Arena 分配器的容器
vixl::List<int> myList;
vixl::Map<std::string, int> myMap;
vixl::Vector<T> myVector;
```

### 分配器对比

| 分配器 | 宏定义 | 用途 |
|-------|--------|------|
| 标准 STL | 无 | 默认使用 `std::` 容器 |
| Panda Arena | `VIXL_USE_PANDA_ALLOC` | 使用 `ark::` Arena 分配器 |

### 条件编译结构

```cpp
#ifdef VIXL_USE_PANDA_ALLOC
#include "mem/arena_allocator_stl_adapter.h"
#include "mem/arena_allocator.h"
#include "utils/arena_containers.h"
#else
#include <list>
#include <map>
#include <vector>
#endif
```

## 常用 API 速查

### MacroAssembler 类

`MacroAssembler` 是最常用的 VIXL 接口，用于生成 ARM 指令：

#### 常用方法

| 方法 | 说明 | 示例 |
|-----|------|------|
| `Mov(dst, src)` | 移动数据 | `masm.Mov(x0, Operand(42))` |
| `Add(dst, src1, src2)` | 加法 | `masm.Add(x0, x1, x2)` |
| `Sub(dst, src1, src2)` | 减法 | `masm.Sub(x0, x1, Operand(1))` |
| `B(label)` | 无条件跳转 | `masm.B(&loop)` |
| `B(cond, label)` | 条件跳转 | `masm.B(EQ, &end)` |
| `Cmp(reg1, reg2)` | 比较 | `masm.Cmp(x0, x1)` |
| `Ret()` | 返回 | `masm.Ret()` |
| `Bind(label)` | 绑定标签 | `masm.Bind(&start)` |

### CPUFeatures 类

用于检测和启用 CPU 特性：

```cpp
// 检测 CPU 特性
vixl::aarch64::CPUFeatures features;
features.Report();

// 启用特定特性
vixl::aarch64::MacroAssembler masm;
masm.GetCPUFeatures()->Combine(vixl::aarch64::CPUFeatures::kNEON);
```

### CodeBuffer 类

代码缓冲区管理：

```cpp
// 创建缓冲区
vixl::CodeBuffer buffer(64 * 1024);  // 64KB

// 写入指令
vixl::aarch64::MacroAssembler masm(&buffer);

// 设置为可执行
#ifdef VIXL_CODE_BUFFER_MMAP
buffer.SetExecutable();
#endif

// 重置缓冲区
buffer.Reset();
```

### Label 类

用于标记指令地址：

```cpp
vixl::aarch64::Label start;
vixl::aarch64::Label end;

// 绑定标签
masm.Bind(&start);
// ... 生成指令
masm.Bind(&end);

// 使用标签跳转
masm.B(&start);
```

## 方舟编译器特定适配

### 寄存器映射

方舟编译器定义了与 VIXL 的寄存器映射：

#### AArch64 寄存器映射

```cpp
// arkcompiler 中的寄存器转换
inline vixl::aarch64::Register VixlReg(Reg reg) {
    auto vixlReg = vixl::aarch64::Register(reg.GetId(), WORD_SIZE);
    if (reg.GetId() == vixl::aarch64::sp.GetCode()) {
        return vixl::aarch64::sp;
    }
    return vixl::aarch64::xzr;
}
```

#### AArch32 寄存器映射

```cpp
// AArch32 寄存器转换
inline vixl::aarch32::Register VixlReg(Reg reg) {
    auto vixlReg = vixl::aarch32::Register(reg.GetId());
    if (reg.IsZero()) {
        return vixl::aarch32::Register();
    }
    return vixlReg;
}
```

### 标签类型定义

```cpp
// 在方舟编译器中使用 VIXL 标签
using LabelType = vixl::aarch64::Label;
```

### 寄存器列表管理

```cpp
// 使用 VIXL 的寄存器列表
static constexpr auto CALLER_VREG_LIST = vixl::aarch64::CPURegList(
    vixl::aarch64::CPURegister::kRegister,
    vixl::aarch64::kXRegSize,
    GetCallerRegsMask(Arch::AARCH64, true).GetValue());
```

## 指令编码示例

### 基本指令

```cpp
// 加载立即数
masm.Mov(x0, Operand(42));

// 寄存器间移动
masm.Mov(x1, x0);

// 内存加载
masm.Ldr(x0, MemOperand(x1, 8));

// 内存存储
masm.Str(x0, MemOperand(x1, 8));

// 加法
masm.Add(x0, x1, Operand(1));

// 减法
masm.Sub(x0, x1, Operand(1));
```

### 跳转指令

```cpp
vixl::aarch64::Label loop;
vixl::aarch64::Label exit;

masm.Bind(&loop);
masm.Cmp(x0, Operand(0));
masm.B(vixl::aarch64::EQ, &exit);
masm.Sub(x0, x0, Operand(1));
masm.B(&loop);
masm.Bind(&exit);
```

### 函数调用

```cpp
// 调用函数
masm.Bl(&function_label);

// 返回
masm.Ret();

// 保存/恢复寄存器
vixl::aarch64::CPURegList saved_regs(vixl::aarch64::kCalleeSaved);
masm.Push(saved_regs);
// ... 函数体
masm.Pop(saved_regs);
```

## 内存操作

### 代码缓冲区操作

```cpp
// 获取当前代码位置
byte* code_start = buffer.GetStartAddress();
byte* code_end = buffer.GetCursor();

// 获取已使用大小
size_t code_size = buffer.GetCursorOffset();
```

### 内存屏障

```cpp
// 数据内存屏障
masm.Dmb(vixl::aarch64::InnerShareable, vixl::aarch64::DSB_ISH);

// 指令同步屏障
masm.Dsb(vixl::aarch64::ISH);
```

## 错误处理

### 断言

```cpp
// 调试模式断言
VIXL_ASSERT(condition);

// 检查条件，不满足则终止
VIXL_CHECK(condition);

// 不可达代码标记
VIXL_UNREACHABLE();
```

### 异常处理

VIXL 在 OH 中禁用异常，因此所有错误处理使用断言：

```cpp
// 分配检查
VIXL_CHECK(buffer_ != NULL);

// 缓冲区空间检查
VIXL_ASSERT(HasSpaceFor(size));
```

## 常见问题解决

### 1. 寄存器溢出

**问题**: 需要的寄存器数量超过可用寄存器

**解决**: 使用栈存储临时值

```cpp
// 保存到栈
masm.Str(x1, MemOperand(sp, -8));
masm.Sub(sp, sp, Operand(8));

// 恢复
masm.Add(sp, sp, Operand(8));
masm.Ldr(x1, MemOperand(sp, 8));
```

### 2. 立即数过大

**问题**: 立即数值超出指令编码范围

**解决**: 使用多个指令或Literal Pool

```cpp
// 分步加载大立即数
masm.Movz(x0, 0x1234, 0);      // 低16位
masm.Movk(x0, 0x5678, 16);     // 高16位
```

### 3. 标签未绑定

**问题**: 跳转目标标签未绑定

**解决**: 确保标签在使用前已绑定

```cpp
// ✅ 正确顺序
vixl::aarch64::Label target;
masm.Bind(&target);
// ... 后续跳转
masm.B(&target);

// ❌ 错误顺序
vixl::aarch64::Label target;
masm.B(&target);  // 跳转目标未绑定
masm.Bind(&target);
```

## 参考资源

- **上游文档**: [VIXL GitHub](https://github.com/Linaro/vixl)
- **API 文档**: `doc/aarch64/` 目录下的详细文档
- **示例代码**: `examples/` 目录
- **测试代码**: `test/` 目录包含大量使用示例
