# JerryScript Patch 详细分析

## 1. Patch 清单

| 序号 | Patch 文件 | 修改文件 | 修改类型 | 优先级 |
|------|------------|---------|---------|--------|
| 1 | `tests/test262-es6.patch` | `tools/packaging/test262.py` | Bugfix | 低 |

**备注**: 除了上述 patch 文件，OpenHarmony 还通过**直接代码修改**方式引入了多个重要变更（见第 2 节）。

---

## 2. 直接代码修改（非 Patch 文件）

### 2.1 内存管理优化（2023 重要变更）

#### 修改文件: `jerry-core/jmem/jmem-heap.c`

**修改类型**: 性能优化

**原始问题**:
- 原始的 `jmem_heap_realloc_block` 逻辑复杂，尝试原地扩展内存块
- 在嵌入式环境中，原地扩展成功率低，反而增加了代码复杂度和执行时间

**修改内容**:
```c
// 简化的重新分配逻辑（2023 patch）
static void *
jmem_heap_realloc_block (jmem_heap_t *heap_p, void *ptr, size_t new_size)
{
    // 旧逻辑：尝试原地扩展，失败则分配新块 + 拷贝 + 释放
    // 新逻辑：直接分配新块 + 拷贝 + 释放（简化路径）

    void *new_ptr = jmem_heap_alloc_block(heap_p, new_size);
    if (new_ptr != NULL) {
        memcpy(new_ptr, ptr, old_size);
        jmem_heap_free_block(heap_p, ptr);
    }
    return new_ptr;
}
```

**OH 价值**:
- 降低代码复杂度，减少维护成本
- 提升嵌入式环境下内存重分配的可预测性
- 减少分支预测失败的次数

**回归风险**: 低。行为逻辑一致，只是简化了实现路径。

**升级建议**: 可评估是否推向上游。此简化在资源受限环境中更有优势。

---

#### 修改文件: `jerry-core/jmem/jmem-heap.c`

**新增功能**: 小对象缓存钩子

**修改内容**:
```c
// 新增钩子函数（JUPITER 平台）
#ifdef JERRY_IAR_JUPITER
void *JerryHeapMalloc(size_t size);
void JerryHeapFree(void *ptr);
#endif
```

**功能说明**:
- 为 8-24 字节的小对象提供外部缓存机制
- 减少频繁的小对象分配/释放开销

**OH 需求**: JUPITER 平台（华为 IoT 芯片）的内存性能优化

**升级建议**: JUPITER 平台特有，不建议推向上游。

---

### 2.2 GC 控制增强

#### 修改文件: `jerry-core/ecma/base/ecma-gc.c`

**修改类型**: 功能增强

**修改内容**:
```c
// 新增 API
void ecma_gc_enable (void)
{
    JERRY_CONTEXT (ecma_gc_mark_recursion_limit) = JERRY_GC_MARK_LIMIT_VALUE;
}

void ecma_gc_disable (void)
{
    JERRY_CONTEXT (ecma_gc_mark_recursion_limit) = 0;
}
```

**功能说明**:
- `ecma_gc_enable()`: 启用垃圾回收
- `ecma_gc_disable()`: 禁用垃圾回收（递归限制设为 0）

**OH 需求**:
- 在关键代码执行期间暂停 GC，避免性能抖动
- 例如：UI 渲染、音频播放等对延迟敏感的场景

**使用场景**:
```cpp
// ace_engine_lite 中的使用示例
ecma_gc_disable();
// 执行关键路径代码（如动画帧渲染）
ecma_gc_enable();  // 恢复 GC
```

**升级建议**: 此功能通用性强，建议向上游贡献。

---

### 2.3 字符串优化（2023 重要变更）

#### 修改文件: `jerry-core/ecma/base/ecma-helpers-string.c`

**修改类型**: 性能优化

**新增功能 1**: 非引用字符串创建
```c
ecma_string_t *
ecma_new_nonref_ecma_string_from_utf8 (const lit_utf8_byte_t *string_p, size_t length)
{
    // 创建不占用堆内存的临时字符串
    // 用于只读比较场景，避免内存分配
}
```

**功能说明**:
- 创建临时的、不占用堆内存的字符串对象
- 用于字符串比较操作，无需持久化

**OH 价值**:
- 减少高频字符串操作的内存分配
- 提升字符串比较性能

---

**新增功能 2**: 字面量比较优化
```c
bool
ecma_compare_ecma_strings_with_literal (ecma_string_t *string_p, const lit_utf8_byte_t *literal_p)
{
    // 利用字面量缓存，优化字符串比较
}
```

**关联文件**: `jerry-core/api/jerry_literal_cache.h`（新增）

**功能说明**:
- 集成字符串字面量缓存机制
- 优化重复字符串的查找和比较

**OH 价值**:
- 大幅减少重复字符串的内存占用
- 提升字符串查找速度（特别是常量字符串）

**升级建议**: 字面量缓存机制通用性强，建议向上游贡献。

---

### 2.4 GC Mark 递归限制

#### 修改文件: `jerry-core/jcontext/jcontext.h`

**修改类型**: 安全增强

**修改内容**:
```c
typedef struct jerry_context_t
{
    // ... 其他字段

    uint32_t ecma_gc_mark_recursion_limit;  // 新增字段
    // ...
}
```

**配置**: `JERRY_GC_MARK_LIMIT`（默认 8）

**功能说明**:
- 限制 GC mark 阶段的递归深度
- 防止在复杂对象图中发生栈溢出

**OH 价值**:
- 提升嵌入式系统的安全性
- 避免深度嵌套对象导致的崩溃

**升级建议**: 安全特性，建议向上游贡献。

---

### 2.5 外部上下文支持

#### 新增文件: `jerry-core/api/external-context-helpers.c`

**修改类型**: 功能扩展

**功能说明**:
- 支持多个独立的 JerryScript 上下文
- 在多任务环境中动态切换上下文

**OH 需求**:
- JUPITER 平台支持 BMS 任务和 JS 任务各自独立的堆内存
- 避免任务间相互干扰

**关联配置**:
- `JERRY_EXTERNAL_CONTEXT=1` (engine.gni 中默认开启)
- `BMS_TASK_HEAP_SIZE=64`, `JS_TASK_HEAP_SIZE=64`

**升级建议**: 可能已在上游实现，需确认上游版本是否已有类似功能。

---

### 2.6 OH 特定文件

#### 新增文件: `jerry-port/config-jupiter.h`

**修改类型**: 平台适配

**关键代码**:
```c
#ifdef JERRY_IAR_JUPITER
#include "ohos_types.h"

#define INPUTJS_BUFFER_SIZE (32 * 1024)
#define SNAPSHOT_BUFFER_SIZE (24 * 1024)
#define BMS_TASK_HEAP_SIZE (64)
#define JS_TASK_HEAP_SIZE (64)

#define JERRY_ENABLE_SNAPSHOT_VERSION_CHECK (1)
#endif
```

**功能说明**:
- JUPITER 平台（华为 IoT 芯片）专用配置
- 定义 OH 特定的缓冲区大小和内存配置

**升级建议**: OH 特有平台适配，不可推向上游。

---

#### 新增文件: `jerry-port/config-gt.h`

**修改类型**: 平台适配

**关键代码**:
```c
#ifdef JERRY_IAR_GT
#include "mc_fs.h"
#include "mc_memory_config.h"

__no_init static uint8_t input_buffer[INPUTJS_BUFFER_SIZE] @ ACE_CACHE_ADDRESS;
__no_init static uint8_t snapshot_buffer[SNAPSHOT_BUFFER_SIZE] @ SNAPSHOT_24K_ADDRESS;
#endif
```

**功能说明**:
- GT 平台专用配置
- 使用 IAR 编译器的 `@` 语法指定内存地址

**升级建议**: OH 特有平台适配，不可推向上游。

---

#### 新增文件: `jerry-core/api/generate-bytecode.c` & `.h`

**修改类型**: 功能扩展

**功能说明**:
- 字节码生成工具的 OH 适配
- 支持从 JS 源码生成 JerryScript 字节码

**OH 需求**:
- bundle_framework_lite 需要 JS 字节码转换功能

**升级建议**: 可评估向上游贡献。

---

#### 新增文件: `jerry-core/api/jerryscript_adapter.c`

**修改类型**: 适配层

**功能说明**:
- JerryScript 与 OH 系统的适配层
- 提供 OH 特定的 API 封装

**使用示例**:
```c
// bundle_framework_lite 中的使用
#include "jerryscript_adapter.h"

void JerryBmsPsRamMemInit(void);
void bms_task_context_init(void);
int get_jerry_version_no(void);
```

**升级建议**: OH 特有适配层，不建议推向上游。

---

## 3. 测试 Patch 详细分析

### 3.1 test262-es6.patch

#### 基本信息

| 项目 | 内容 |
|------|------|
| **文件路径** | `tests/test262-es6.patch` |
| **修改文件** | `tools/packaging/test262.py` |
| **修改类型** | Bugfix |
| **OH 需求** | ES6/ES2015 测试支持 |

#### 修改内容

```diff
diff --git a/tools/packaging/test262.py b/tools/packaging/test262.py
index 921360a05e..27a2938e48 100755
--- a/tools/packaging/test262.py
+++ b/tools/packaging/test262.py
@@ -469,8 +469,8 @@ class TestSuite(object):
            if self.ShouldRun(rel_path, tests):
              basename = path.basename(full_path)[:-3]
              name = rel_path.split(path.sep)[:-1] + [basename]
-            if EXCLUDE_LIST.count(basename) >= 1:
-              print 'Excluded: ' + basename
+            if rel_path in EXCLUDE_LIST:
+              print 'Excluded: ' + rel_path
              else:
                if not self.non_strict_only:
                  strict_case = TestCase(self, name, full_path, True)
```

#### 原始问题

test262 是 JavaScript 标准测试套件，包含大量 ES6/ES2015 兼容性测试。原始的排除逻辑有缺陷：

- **只检查文件名** (`basename`): 如果 `excluded_tests.js` 中包含 `"array.js"`，则所有目录下的 `array.js` 都会被排除
- **无法精确控制**: 无法排除特定路径的测试（如 `built-ins/array/array.js` 但保留 `other/array.js`）

#### 修改说明

- **改为检查完整相对路径** (`rel_path`): 如 `"built-ins/array/array.js"`
- **更精确的排除**: 可以只排除特定目录下的测试文件
- **提升灵活性**: 允许在测试排除列表中使用路径匹配

#### OH 需求

OpenHarmony 需要运行 test262 测试来验证 JerryScript 的 ES6 特性支持。此 patch 使测试框架更灵活，能够：

1. 排除已知失败的 ES6 测试
2. 保留可以通过的测试
3. 避免因文件名重复导致的错误排除

#### 回归风险

**低风险**：
- 仅影响测试工具，不影响引擎核心功能
- 改动简单，逻辑清晰

#### 升级建议

**强烈建议推向上游**：
- 这是通用的测试框架改进
- 所有使用 test262 的项目都能受益
- 不涉及平台特定逻辑

---

## 4. Patch 分类汇总

### 4.1 按功能分类

| 分类 | 数量 | 说明 |
|------|------|------|
| **性能优化** | 3 | 内存管理、字符串优化、小对象缓存 |
| **功能增强** | 3 | GC 控制、外部上下文、字节码生成 |
| **安全加固** | 1 | GC Mark 递归限制 |
| **平台适配** | 3 | JUPITER、GT 平台配置 |
| **测试改进** | 1 | test262 排除逻辑 |

### 4.2 按优先级分类

| 优先级 | Patch/修改 | 说明 |
|--------|-----------|------|
| **高** | GC 控制、字符串优化 | 性能和功能关键，影响核心使用 |
| **中** | 内存管理优化、外部上下文 | 性能优化，但行为一致 |
| **低** | 平台适配、测试改进 | 平台特定或仅测试相关 |

### 4.3 按可向上游贡献分类

| 可贡献 | 说明 |
|--------|------|
| **推荐贡献** | test262 patch、GC 控制、GC Mark 限制、字符串优化 |
| **可评估贡献** | 内存管理简化、字节码生成 |
| **不推荐贡献** | JUPITER/GT 平台适配、jerryscript_adapter（OH 特有） |

---

## 5. 升级建议

### 5.1 升级前的准备工作

1. **代码审计**:
   - 列出所有 OH 特定的代码修改
   - 标记需要保留的 patch
   - 确认上游版本是否已包含类似功能

2. **功能验证**:
   - 验证上游版本是否支持外部上下文
   - 检查 ES2015 特性支持情况
   - 确认安全相关功能（CVE 修复）是否已合入

3. **性能对比**:
   - 对比 OH 优化的性能影响
   - 如果上游版本性能更优，可移除 OH 优化
   - 如果 OH 优化更优，保留为 patch

### 5.2 升级策略

**保守策略**（推荐）:
1. 先合入上游的 bug 修复和安全修复
2. 保留 OH 的性能优化和功能增强
3. 在测试环境验证通过后，再考虑推向上游

**激进策略**:
1. 尝试将 OH 优化推向上游
2. 接受上游版本，放弃部分 OH 优化
3. 通过重构方式实现相同效果

### 5.3 回归测试重点

升级后需要重点测试：

1. **ace_engine_lite**: UI 引擎 JS 运行时
2. **bundle_framework_lite**: 字节码转换
3. **netmanager_base**: PAC 代理脚本执行
4. **内存压力测试**: 堆内存分配和释放
5. **GC 行为**: EnableGC/DisableGC 功能
6. **字符串操作**: 字面量缓存效果

---

## 6. Patch 维护建议

### 6.1 定期审查

建议每 6 个月审查一次 patch 列表：

- 检查上游版本更新
- 评估 OH patch 是否仍有必要
- 识别可推向上游的改进

### 6.2 文档更新

每次升级后，更新本文档：

- 记录新的 patch/修改
- 更新升级建议
- 标记已合入上游的功能

### 6.3 风险管理

- 在 `ASSESSMENT.md` 中记录高风险 patch
- 在升级前进行影响评估
- 保留关键 patch 的详细说明和测试用例
