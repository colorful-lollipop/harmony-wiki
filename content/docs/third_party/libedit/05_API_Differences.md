# 05_API_Differences - API/接口差异

本文档分析 libedit 在 OpenHarmony 中的 API 和接口差异。

---

## 1. API 差异概览

### 1.1 差异总结

| 差异类型 | 数量 | 说明 |
|----------|------|------|
| **OH 新增 API** | 0 | 无 OH 特有 API |
| **行为变更 API** | 0 | 无 API 行为变更 |
| **废弃/禁用 API** | 0 | 无废弃或禁用功能 |
| **兼容性问题** | 0 | 完全兼容上游 |

**结论**：✅ **无 API 差异**

**证据来源**：源码分析、OH 代码搜索

### 1.2 差异评估

| 评估项 | 结果 | 说明 |
|--------|------|------|
| **API 完整性** | ✅ 完整 | 所有上游 API 均可用 |
| **API 兼容性** | ✅ 兼容 | 与上游完全兼容 |
| **行为一致性** | ✅ 一致 | 行为与上游一致 |
| **文档完整性** | ✅ 完整 | 使用上游文档 |

---

## 2. OH 新增 API

### 2.1 新增 API 清单

| API 名称 | 返回类型 | 参数 | 说明 | 状态 |
|----------|----------|------|------|------|
| - | - | - | - | ❌ 无 |

### 2.2 分析说明

**搜索 OH 特定 API**：

```bash
# 搜索 OH 特定宏定义
grep -r "#ifdef OHOS\|#ifdef OPENHARMONY" \
  /Volumes/lexar/code/d/work/oh/third_party/libedit/src \
  --include="*.c" --include="*.h"
```

**结果**：❌ 未发现

**搜索 OH 特定函数**：

```bash
# 搜索 OH 前缀的函数
grep -r "ohos_\|OHOS_" \
  /Volumes/lexar/code/d/work/oh/third_party/libedit/src \
  --include="*.c" --include="*.h"
```

**结果**：❌ 未发现

**结论**：无 OH 特定的新增 API。

---

## 3. 行为变更 API

### 3.1 行为变更清单

| API 名称 | 上游行为 | OH 行为 | 变更类型 | 状态 |
|----------|----------|---------|----------|------|
| - | - | - | - | ❌ 无 |

### 3.2 分析说明

**分析 OH 特定逻辑**：

```bash
# 搜索 OH 特定条件编译
grep -rn "__OHOS__\|__OPENHARMONY__" \
  /Volumes/lexar/code/d/work/oh/third_party/libedit/src
```

**结果**：❌ 未发现

**分析行为修改**：

```bash
# 查找所有条件编译
grep -rn "#if.*OHOS\|#elif.*OHOS\|#ifdef.*OHOS" \
  /Volumes/lexar/code/d/work/oh/third_party/libedit/src
```

**结果**：❌ 未发现

**结论**：无 API 行为变更。

---

## 4. 废弃/禁用 API

### 4.1 废弃 API 清单

| API 名称 | 废弃原因 | 替代方案 | 状态 |
|----------|----------|----------|------|
| - | - | - | ❌ 无 |

### 4.2 禁用功能清单

| 功能 | 禁用原因 | 重新启用方法 | 状态 |
|------|----------|--------------|------|
| - | - | - | ❌ 无 |

### 4.3 分析说明

**搜索禁用宏**：

```bash
# 搜索禁用功能的宏定义
grep -rn "#undef\|DISABLE\|NO_" \
  /Volumes/lexar/code/d/work/oh/third_party/libedit/config.h.in
```

**结果**：❌ 未发现 OH 特定禁用

**结论**：无废弃或禁用的 API 和功能。

---

## 5. 兼容性分析

### 5.1 头文件兼容性

| 头文件 | 上游版本 | OH 版本 | 兼容性 |
|--------|----------|---------|--------|
| `<histedit.h>` | 完整 | 完整 | ✅ 兼容 |
| `<editline/readline.h>` | 完整 | 完整 | ✅ 兼容 |

**验证命令**：

```bash
# 比较头文件差异
diff -u <(curl -s https://raw.githubusercontent.com/thrysoee/libedit/main/src/histedit.h) \
        src/histedit.h
```

**结果**：❌ 无差异（与上游一致）

### 5.2 函数签名兼容性

**示例函数**：`el_init()`

**上游签名**：
```c
EditLine *el_init(const char *prog, FILE *fin, FILE *fout, FILE *ferr);
```

**OH 签名**：
```c
EditLine *el_init(const char *prog, FILE *fin, FILE *fout, FILE *ferr);
```

**结论**：✅ 签名一致

**验证方法**：对比 `src/el.c` 和上游代码。

### 5.3 数据结构兼容性

**示例结构**：`EditLine`

**上游定义**：
```c
struct editline {
    /* ... */
};
typedef struct editline EditLine;
```

**OH 定义**：
```c
struct editline {
    /* ... */
};
typedef struct editline EditLine;
```

**结论**：✅ 结构一致

---

## 6. 宏定义兼容性

### 6.1 编译选项宏

| 宏名称 | 上游定义 | OH 定义 | 兼容性 |
|--------|----------|---------|--------|
| `HAVE_CONFIG_H` | 标准配置宏 | 标准配置宏 | ✅ 兼容 |
| `WIDECHAR` | 宽字符支持 | 宽字符支持 | ✅ 兼容 |

**分析**：所有宏定义与上游一致。

### 6.2 功能开关宏

| 宏名称 | 默认值 | OH 默认值 | 兼容性 |
|--------|--------|----------|--------|
| `ENABLE_WIDEC` | 启用 | 启用 | ✅ 兼容 |
| `ENABLE_EXAMPLES` | 禁用 | 禁用 | ✅ 兼容 |

**结论**：无宏定义差异。

---

## 7. 库兼容性

### 7.1 库文件兼容性

| 库文件 | 上游版本 | OH 版本 | 兼容性 |
|--------|----------|---------|--------|
| `libedit.so` | 标准 ABI | 标准 ABI | ✅ 兼容 |
| `libedit.a` | 标准 ABI | 标准 ABI | ✅ 兼容 |

**验证方法**：
```bash
# 检查符号表
nm -D lib/.libs/libedit.so | grep -E "el_|histedit"
```

**结果**：符号表与上游一致。

### 7.2 pkg-config 兼容性

**文件**：`libedit.pc.in`

**上游内容**：
```
Name: libedit
Version: @VERSION@
Description: NetBSD Editline library
Libs: -L${libdir} -ledit
Cflags: -I${includedir}
```

**OH 内容**：
```
Name: libedit
Version: @VERSION@
Description: NetBSD Editline library
Libs: -L${libdir} -ledit
Cflags: -I${includedir}
```

**结论**：✅ 完全一致

---

## 8. 使用示例

### 8.1 基本使用（与上游相同）

```c
#include <histedit.h>

int main() {
    // 初始化编辑器
    EditLine *el = el_init("myapp", stdin, stdout, stderr);

    // 读取一行输入
    const char *line;
    int count;
    while ((line = el_gets(el, &count)) != NULL) {
        printf("You entered: %s\n", line);
    }

    // 清理
    el_end(el);
    return 0;
}
```

**编译命令**：
```bash
gcc myapp.c -o myapp -I/path/to/libedit/include \
    -L/path/to/libedit/lib -ledit -lcurses
```

### 8.2 Readline 兼容模式（与上游相同）

```c
#include <editline/readline.h>

int main() {
    char *line;
    while ((line = readline("myapp> ")) != NULL) {
        if (*line) {
            add_history(line);
        }
        printf("You entered: %s\n", line);
        free(line);
    }
    return 0;
}
```

**编译命令**：
```bash
gcc myapp.c -o myapp -I/path/to/libedit/include \
    -L/path/to/libedit/lib -ledit -lcurses
```

### 8.3 历史记录功能（与上游相同）

```c
#include <histedit.h>

int main() {
    EditLine *el = el_init("myapp", stdin, stdout, stderr);
    HistEvent ev;
    History *hist = history_init();
    history(hist, &ev, H_SETSIZE, 100);
    el_set(el, EL_HIST, history, hist);

    const char *line;
    int count;
    while ((line = el_gets(el, &count)) != NULL) {
        history(hist, &ev, H_ENTER, line);
        printf("You entered: %s\n", line);
    }

    history_end(hist);
    el_end(el);
    return 0;
}
```

---

## 9. 迁移指南

### 9.1 从 GNU Readline 迁移

由于 libedit 提供 Readline 兼容 API，迁移非常简单：

**步骤 1**：替换头文件
```c
// 从
#include <readline/readline.h>
#include <readline/history.h>

// 到
#include <editline/readline.h>
```

**步骤 2**：重新链接
```bash
# 从
gcc myapp.c -o myapp -lreadline -lncurses

# 到
gcc myapp.c -o myapp -ledit -lncurses
```

**注意**：libedit 不支持所有 GNU Readline 功能，但支持核心功能。

### 9.2 从旧版 libedit 升级

**步骤 1**：检查 API 变化
- 阅读 CHANGELOG
- 检查 deprecated API

**步骤 2**：更新代码
- 替换 deprecated API
- 更新编译选项

**步骤 3**：测试
- 运行测试套件
- 验证功能正确性

---

## 10. 限制与差异

### 10.1 与 GNU Readline 的差异

虽然 libedit 提供 Readline 兼容 API，但与 GNU Readline 存在一些差异：

| 特性 | GNU Readline | libedit | 兼容性 |
|------|--------------|---------|--------|
| **许可证** | GPL-3.0+ | BSD-3-Clause | ⚠️ 不同 |
| **完整功能** | 完整 | 部分兼容 | ⚠️ 部分功能不支持 |
| **宏定义** | 丰富 | 基础 | ⚠️ 部分宏不支持 |

**注意**：这些差异是 libedit 和 GNU Readline 之间的差异，不是 OH 特定的。

### 10.2 OH 无额外限制

⚠️ **重要**：OpenHarmony 对 libedit 的 API **无任何额外限制或差异**。

所有 API 行为与上游完全一致。

---

## 11. 最佳实践

### 11.1 头文件包含

```c
// 推荐：使用 libedit 原生 API
#include <histedit.h>

// 或：使用 Readline 兼容 API
#include <editline/readline.h>

// 不推荐：直接引用源文件
// #include "src/histedit.h"
```

### 11.2 错误处理

```c
#include <histedit.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    EditLine *el = el_init("myapp", stdin, stdout, stderr);
    if (!el) {
        fprintf(stderr, "Failed to initialize EditLine\n");
        return 1;
    }

    // 使用 el...

    el_end(el);
    return 0;
}
```

### 11.3 资源管理

```c
// 确保清理资源
EditLine *el = el_init("myapp", stdin, stdout, stderr);
if (!el) {
    // 错误处理
}

// 使用 el...

// 必须清理
el_end(el);
```

---

## 12. 总结

### 12.1 核心发现

1. ✅ **无 OH 特定 API 差异**
2. ✅ **无 API 行为变更**
3. ✅ **无废弃或禁用功能**
4. ✅ **完全兼容上游**

### 12.2 兼容性评估

| 评估项 | 结果 |
|--------|------|
| **API 完整性** | ✅ 完整 |
| **API 兼容性** | ✅ 兼容 |
| **行为一致性** | ✅ 一致 |
| **文档完整性** | ✅ 完整 |

### 12.3 使用建议

| 场景 | 建议 |
|------|------|
| **当前（未使用）** | 无需关注 API 差异 |
| **如需使用** | 直接使用上游 API，无差异 |
| **从 Readline 迁移** | 参考 §9.1 迁移指南 |

---

## 相关文档

- **库概览**：[01_Overview.md](01_Overview.md)
- **Patch 分析**：[02_Patches.md](02_Patches.md)
- **使用情况**：[04_Usage_in_OH.md](04_Usage_in_OH.md)
- **构建适配**：[03_Build_Integration.md](03_Build_Integration.md)

---

**文档最后更新**：2025-02-07
**证据来源**：源码分析、上游文档对比
