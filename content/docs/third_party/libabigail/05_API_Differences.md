# 05 - API/接口差异

## 5.1 概述

### 5.1.1 本库的特殊性

与其他第三方库不同，**libabigail 在 OH 中没有 API/接口差异**。

**原因**:
1. OH **不使用** libabigail 的库 API
2. OH **仅使用** libabigail 的命令行工具
3. 工具通过**标准输入输出**与 OH 构建系统集成

### 5.1.2 使用方式对比

| 库 | OH 使用方式 | 是否有 API 差异 |
|----|------------|----------------|
| libabigail | 命令行工具 | ❌ 无 |
| curl | 库 API | ✅ 有 |
| openssl | 库 API | ✅ 有 |
| libxml2 | 库 API | ✅ 有 |

## 5.2 命令行接口

### 5.2.1 abidw 工具

**功能**: 从 ELF 二进制文件提取 ABI 信息并输出为 ABIXML

**OH 使用方式**:
```bash
abidw [options] <elf-file> -o <output.xml>
```

**OH 常用选项**:
- `--no-corpus-path`: 不记录语料库路径
- `--no-show-locs`: 不显示位置信息

**在 check_abi_and_copy_deps.py 中的使用**:
```python
abidw_cmd = [
    abidw_path,
    "--no-corpus-path",
    "--no-show-locs",
    "-o", output_xml,
    elf_file
]
```

**与上游标准用法一致** ✅

### 5.2.2 abidiff 工具

**功能**: 比较两个 ABI 特征文件的差异

**OH 使用方式**:
```bash
abidiff [options] <abi-file1> <abi-file2>
```

**OH 常用选项**:
- `--no-added-syms`: 不报告新增的符号（可选）
- `--suppressions`: 指定抑制规则文件（可选）

**在 check_abi_and_copy_deps.py 中的使用**:
```python
abidiff_cmd = [
    abidiff_path,
    baseline_xml,
    current_xml
]
```

**与上游标准用法一致** ✅

### 5.2.3 命令行接口稳定性

libabigail 的命令行接口非常稳定：

| 版本 | 命令行接口变化 |
|------|---------------|
| 2.8 | 无破坏性变更 |
| 2.7 | 无破坏性变更 |
| 2.6 | 无破坏性变更 |
| 2.5 | 无破坏性变更 |
| 2.4 | 基线版本 |

## 5.3 OH 新增的集成方式

虽然 libabigail 本身没有变化，但 OH 创建了**新的集成方式**来调用这些工具：

### 5.3.1 check_abi_and_copy_deps 模板

**类型**: GN 模板 (gni)  
**路径**: `build/templates/update/module_update.gni`

**功能**:
- 在构建时自动调用 abidiff/abidw
- 管理工具依赖
- 传递参数给 Python 脚本

**代码片段**:
```gni
template("check_abi_and_copy_deps") {
  action(target_name) {
    abidiff_target = "//third_party/libabigail/tools:abidiff($host_toolchain)"
    abidw_target = "//third_party/libabigail/tools:abidw($host_toolchain)"
    
    deps = invoker.sources
    deps += [ abidiff_target ]
    deps += [ abidw_target ]
    
    script = "//build/ohos/update/check_abi_and_copy_deps.py"
    # ...
  }
}
```

### 5.3.2 check_abi_and_copy_deps.py 脚本

**类型**: Python 脚本  
**路径**: `build/ohos/update/check_abi_and_copy_deps.py`

**功能**:
- 解析 GN 传递的参数
- 调用 abidw 生成当前 ABI
- 调用 abidiff 对比 ABI
- 返回检查结果

**这是 OH 特有的包装层**，不是 libabigail 本身的修改。

### 5.3.3 ABI 基线文件

**路径**: `prebuilts/abi_dumps/`

**格式**: ABIXML（libabigail 标准格式）

**内容示例**:
```xml
<abixml version="2.4">
  <corpus path="libexample.so" architecture="elf-amd-x86_64">
    <elf-needed>libc.so.6</elf-needed>
    <symbols>
      <elf-symbol name='example_func' type='func-type' .../>
    </symbols>
    <abi-instr version='1.0' ...>
      <type-decl name='int' .../>
      <function-decl name='example_func' ...>
        ...
      </function-decl>
    </abi-instr>
  </corpus>
</abixml>
```

**这是 libabigail 的标准输出格式**，OH 未做任何修改。

## 5.4 接口边界

### 5.4.1 OH 与 libabigail 的接口边界

```
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarmony 构建系统                      │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  check_abi_and_copy_deps 模板 (GN)                    │  │
│  └───────────────────────────────────────────────────────┘  │
│                           │                                 │
│                           ▼                                 │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  check_abi_and_copy_deps.py (Python)                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                           │                                 │
└───────────────────────────┼─────────────────────────────────┘
                            │ 标准命令行调用
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                    libabigail 工具                          │
│  ┌─────────────┐              ┌─────────────┐              │
│  │   abidw     │              │   abidiff   │              │
│  │ (标准接口)   │              │ (标准接口)   │              │
│  └─────────────┘              └─────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

**接口方式**: 命令行参数 + 标准输入输出  
**接口复杂度**: 低（无 API 集成）  
**耦合度**: 低（可替换为兼容工具）

### 5.4.2 与库 API 集成的对比

如果 OH 使用 libabigail 的库 API，将需要：

```cpp
// 假设的 API 使用方式（实际未使用）
#include "abg-corpus.h"
#include "abg-writer.h"

using namespace abigail;

// 读取 ELF
corpus_sptr corp = read_corpus_from_elf("libexample.so");

// 写入 ABIXML
write_corpus_to_xml(corp, "output.xml");

// 对比两个 ABI
corpus_sptr base = read_corpus_from_xml("base.xml");
corpus_sptr curr = read_corpus_from_xml("current.xml");
diff_sptr d = compute_diff(base, curr);
```

**实际使用**: 命令行调用，无需包含头文件或链接库。

## 5.5 输入输出格式

### 5.5.1 输入

| 输入类型 | 格式 | 说明 |
|----------|------|------|
| ELF 文件 | ELF + DWARF | abidw 的输入 |
| ABIXML | XML | abidiff 的输入 |

### 5.5.2 输出

| 输出类型 | 格式 | 说明 |
|----------|------|------|
| ABIXML | XML | abidw 的输出 |
| 差异报告 | 文本 | abidiff 的输出 |

### 5.5.3 ABIXML 格式版本

从 `include/abg-version.h`:
```c
#define ABIGAIL_ABIXML_VERSION_MAJOR "2"
#define ABIGAIL_ABIXML_VERSION_MINOR "4"
```

当前 ABIXML 格式版本: **2.4**

## 5.6 兼容性说明

### 5.6.1 向前兼容

由于使用命令行接口：
- ✅ 升级 libabigail 通常不会影响 OH 构建
- ✅ 命令行参数向后兼容
- ✅ ABIXML 格式向后兼容

### 5.6.2 升级注意事项

唯一需要关注的变化：
1. **默认输出格式变化**（可能性极低）
2. **命令行参数行为变化**（可能性极低）
3. **ABIXML 格式版本升级**（需要重新生成基线）

## 5.7 与其他工具的兼容性

### 5.7.1 替代工具

由于使用标准命令行接口，理论上可以用其他 ABI 检查工具替换：

| 工具 | 兼容性 | 说明 |
|------|--------|------|
| libabigail | ✅ 原生 | OH 当前使用的工具 |
| abi-compliance-checker | ⚠️ 需适配 | 输出格式不同 |
| abigail（其他实现） | ⚠️ 需适配 | 需兼容命令行接口 |

### 5.7.2 切换成本

如果未来需要切换工具：
1. 修改 `check_abi_and_copy_deps.py`
2. 适配新的命令行参数
3. 适配新的输出格式
4. 重新生成 ABI 基线

**成本评级**: ⭐⭐ 低（因接口简单）

## 5.8 结论

### 5.8.1 核心结论

| 问题 | 答案 |
|------|------|
| 是否有 API 差异？ | 无，OH 不使用库 API |
| 是否有行为变更？ | 无，命令行接口标准 |
| OH 添加了什么？ | 集成模板和脚本（包装层） |
| 接口稳定性如何？ | 非常稳定 |

### 5.8.2 维护建议

由于无 API 差异，维护工作主要是：

1. **跟踪上游版本**
   - 关注命令行接口变更公告
   - 关注 ABIXML 格式版本升级

2. **验证工具行为**
   - 升级后验证 abidiff/abidw 基本功能
   - 验证与现有基线文件的兼容性

3. **无需关注的方面**
   - ❌ 头文件变更
   - ❌ 库 API 变更
   - ❌ 链接兼容性

---

**上一章**: [04_Usage_in_OH.md](04_Usage_in_OH.md)  
**下一章**: [06_Security.md](06_Security.md)
