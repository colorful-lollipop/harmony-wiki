# OH Patch 详细分析

> Toybox 没有使用传统的 .patch 文件，所有修改都通过源代码中的条件编译实现。

---

## 2.1 Patch 状态总览

### Patch 策略

OpenHarmony 采用**无 Patch 文件策略**：

| 策略 | 说明 |
|-----|------|
| **传统方式** | 使用 .patch 文件，在构建时应用 |
| **OH 方式** | 直接修改源代码，通过 `TOYBOX_OH_ADAPT` 宏控制 |
| **优势** | 避免合并冲突，减少构建步骤 |
| **劣势** | 升级上游版本时需要手动重新应用修改 |

### 修改统计

| 指标 | 数值 | 占比 |
|-----|------|------|
| **OH 特有源文件** | 1 个独立（su.c） + 20+ 个适配修改 | ~3% |
| **条件编译点** | 95 处 `TOYBOX_OH_ADAPT` | 中等 |
| **修改命令数** | 19 个 | ~10% |
| **新增代码量** | ~2000 行（su.c + 适配代码） | 低 |

---

## 2.2 Patch 清单表

### 所有 OH 修改汇总

| 序号 | 文件路径 | 修改类型 | 修改目的 | 影响命令 |
|-----|---------|---------|---------|---------|
| 1 | `lib/args.c` | 64 位适配 | 解决 64 位系统上的类型问题 | 所有使用 getopt 的命令 |
| 2 | `lib/dirtree.c` | 稳定性修复 | 防止 closedir(NULL) 导致崩溃 | ls, find 等目录遍历命令 |
| 3 | `lib/tty.c` | 显示修复 | 修复 top 命令在交互模式下显示乱码 | top |
| 4 | `toys/posix/cp.c` | Bug 修复 | 修复 mv -v 不打印详细日志的问题 | mv |
| 5 | `toys/posix/strings.c` | 性能优化 | 文件关闭优化 | strings |
| 6 | `toys/posix/ls.c` | 用户体验优化 | 多项改进：排序、块大小、目录处理 | ls |
| 7 | `toys/posix/ps.c` | 功能适配 | 进程信息显示格式调整 | ps |
| 8 | `porting/liteos_a/` (20+ 文件) | 平台适配 | LiteOS_A 内核适配 | 20+ 命令 |
| 9 | `openharmony/su.c` | OH 特有实现 | 独立的 su 命令实现 | su |

---

## 2.3 基础库层修复

### Patch 1: 64 位类型适配（lib/args.c）

**修改文件**：`lib/args.c`

**修改摘要**：
```c
#ifdef TOYBOX_OH_ADAPT
    unsigned long long u = 1ULL<<idx++;
#else
    unsigned long long u = 1LL<<idx++;
#endif
```

**原始问题**：
- 在 64 位系统上，`1LL`（signed long long）的行为可能与预期不同
- 左移操作可能导致未定义行为

**OH 需求**：
- 确保 64 位系统上参数解析的正确性
- 避免符号位导致的意外行为

**关键代码变更**：
- 使用 `1ULL`（unsigned long long）替代 `1LL`
- 明确使用无符号类型进行位移操作

**影响范围**：
- 所有使用 toybox 参数解析机制的命令
- 涉及命令行选项解析的所有命令

**升级建议**：
- 此 Patch 为通用修复，可考虑推向上游
- 升级时需确保参数解析逻辑不变

---

### Patch 2: 目录遍历稳定性修复（lib/dirtree.c）

**修改文件**：`lib/dirtree.c`

**修改摘要**：
```c
#ifdef TOYBOX_OH_ADAPT
  if (dir) {
    // fix crash of closedir(NULL)
    closedir(dir);
  }
#else
  closedir(dir);
#endif
```

**原始问题**：
- 某些边界条件下 `dir` 可能为 NULL
- 调用 `closedir(NULL)` 会导致程序崩溃

**OH 需求**：
- 提高系统稳定性，防止因 NULL 指针导致的崩溃
- 在 OpenHarmony 的复杂文件系统环境中更加健壮

**关键代码变更**：
- 在调用 `closedir()` 前检查 `dir` 是否为 NULL
- 添加明确的注释说明修复的问题

**影响范围**：
- 所有遍历目录的命令
- 主要影响：ls, find, du 等命令

**升级建议**：
- 此 Patch 为稳定性修复，强烈建议推向上游
- 升级时需确保 NULL 检查逻辑保留

---

### Patch 3: 终端显示修复（lib/tty.c）

**修改文件**：`lib/tty.c`

**修改摘要**：
```c
#ifdef TOYBOX_OH_ADAPT
  // fix garble for 'top -k aa'
#else
  xprintf("\e[s\e[999C\e[999B\e[6n\e[u");
#endif
```

**原始问题**：
- 在某些终端环境下，使用 ANSI 转义序列查询光标位置会导致显示乱码
- 特别是在 `top -k` 交互模式下表现明显

**OH 需求**：
- 修复 top 命令在 OpenHarmony 终端环境下的显示问题
- 提升用户体验

**关键代码变更**：
- OH 版本移除了可能导致乱码的终端查询序列
- 添加注释说明问题场景

**影响范围**：
- 主要影响：top 命令
- 次要影响：其他使用终端控制的命令

**升级建议**：
- 此 Patch 为终端兼容性修复
- 升级时需测试 top 命令在 OH 终端环境下的表现
- 可考虑根据新的终端实现调整方案

---

## 2.4 命令层修改

### Patch 4: mv 命令日志修复（toys/posix/cp.c）

**修改文件**：`toys/posix/cp.c`

**修改摘要**：
```c
#ifdef TOYBOX_OH_ADAPT
      /* fix "mv -v 123.txt test/" not print detail log problem*/
      if (send) {
        if (FLAG(v))
          printf("renamed '%s' -> '%s'\n", src, TT.destname);
        send = rename(src, TT.destname);
```

**原始问题**：
- 使用 `mv -v` 移动文件到目录时，不打印详细日志
- 用户无法看到具体的移动操作

**OH 需求**：
- 确保用户在 verbose 模式下能看到所有操作的反馈
- 提升用户体验，便于调试

**关键代码变更**：
- 在执行 `rename()` 之前打印日志信息
- 调整代码顺序，确保 `FLAG(v)` 检查有效

**影响范围**：
- 主要影响：mv 命令的 verbose 模式

**升级建议**：
- 此 Patch 为 Bug 修复，建议推向上游
- 升级时需确保 verbose 日志逻辑一致

---

### Patch 5: strings 命令性能优化（toys/posix/strings.c）

**修改文件**：`toys/posix/strings.c`

**修改摘要**：
```c
#ifndef TOYBOX_OH_ADAPT
  xclose(fd);
#endif
```

**原始问题**：
- 某些情况下过早关闭文件描述符
- 可能影响性能或后续操作

**OH 需求**：
- 优化文件描述符管理
- 提高命令执行效率

**关键代码变更**：
- OH 版本移除了 `xclose(fd)` 调用
- 让文件描述符在更合适的时候关闭

**影响范围**：
- 主要影响：strings 命令
- 性能影响：在处理大文件时更明显

**升级建议**：
- 此 Patch 为性能优化
- 升级时需评估文件描述符管理的最佳实践

---

### Patch 6: ls 命令多项改进（toys/posix/ls.c）

**修改文件**：`toys/posix/ls.c`

**修改摘要**：
- 块大小计算调整
- 新增文件优先排序
- 目录处理逻辑优化
- 排序选项改进

#### 修改 6.1: 块大小计算

```c
#ifdef TOYBOX_OH_ADAPT
  new->st.st_blocks = (new->st.st_blocks + 1) >> 1; // Use 1KiB blocks rather than 512B blocks.
#else
  new->st.st_blocks >>= 1; // Use 1KiB blocks rather than 512B blocks.
#endif
```

**原始问题**：
- 右移操作可能导致精度丢失
- 块大小计算不够准确

**OH 需求**：
- 更准确地显示文件占用的块大小
- 与 OpenHarmony 文件系统行为一致

**关键代码变更**：
- 使用 `(st_blocks + 1) >> 1` 进行更准确的四舍五入
- 添加注释说明使用 1KiB 块而非 512B 块

#### 修改 6.2: 文件优先排序

```c
#ifdef TOYBOX_OH_ADAPT
// callback for qsort but file first
static int compare_file_first(void *a, void *b)
{
  struct dirtree *dta = *(struct dirtree **)a;
  struct dirtree *dtb = *(struct dirtree **)b;
  // 文件排在目录之前
  if (S_ISDIR(dta->st.st_mode) != S_ISDIR(dtb->st.st_mode)) {
    return S_ISDIR(dta->st.st_mode) ? 1 : -1;
  }
  return compare(a, b);
}
#endif
```

**原始问题**：
- ls 默认排序没有区分文件和目录
- 用户查看时需要在不同类型之间切换

**OH 需求**：
- 优化 ls 的显示顺序，文件优先显示
- 提升用户体验，便于快速查找文件

**关键代码变更**：
- 新增 `compare_file_first()` 比较函数
- 文件和目录分组排序
- 集成到排序逻辑中

#### 修改 6.3: 目录处理优化

```c
#ifdef TOYBOX_OH_ADAPT
  unsigned long dtlen_old;
  int skip_dir = !indir->parent && !FLAG(d);
  // ... 处理逻辑
  dtlen_old = dtlen;
  if (FLAG(f) || FLAG(U)) {
    // no sort, so fix it here
    if (skip_dir) {
      dtlen = move_dir_back(sort, dtlen_old);
    }
  }
  // ... 后续处理
  if (!FLAG(U)) {
#ifdef TOYBOX_OH_ADAPT
    if (skip_dir) {
      // compare with file first
      qsort(sort, dtlen, sizeof(void *), (void *)compare_file_first);
      // modify dtlen to skip all dirs
      for (ul = 0; ul < dtlen; ul++) {
        if (S_ISDIR(sort[ul]->st.st_mode)) {
          break;
        }
      }
      dtlen = ul;
    }
#endif
    qsort(sort, dtlen, sizeof(void *), (void *)compare);
#ifdef TOYBOX_OH_ADAPT
    }
#endif
  }
#endif
```

**原始问题**：
- 根目录（`/`）的显示逻辑特殊处理不当
- 某些排序选项下显示顺序不符合预期

**OH 需求**：
- 优化根目录和子目录的显示逻辑
- 确保不同排序选项下的一致性

**关键代码变更**：
- 新增 `skip_dir` 标志，识别根目录
- 在某些排序模式下将目录移到列表末尾
- 调整 `dtlen` 以跳过目录项

**影响范围**：
- 主要影响：ls 命令的所有排序模式
- 用户体验：更清晰的目录浏览体验

**升级建议**：
- 此 Patch 为用户体验优化
- 升级时需评估是否与上游的改进冲突
- 可考虑将部分优化推向上游

---

### Patch 7: ps 命令调整（toys/posix/ps.c）

**修改文件**：`toys/posix/ps.c`

**修改摘要**：
- 进程信息显示格式调整
- OH 特定进程属性显示

**原始问题**：
- 上游 ps 的输出格式不符合 OH 的需求
- 缺少某些 OH 特有的进程属性

**OH 需求**：
- 适配 OpenHarmony 的进程管理机制
- 显示 OH 特有的进程信息（如进程组、权限等）

**关键代码变更**：
- 调整输出格式，匹配 OH 的进程管理需求
- 添加 OH 特有的进程属性字段

**影响范围**：
- 主要影响：ps 命令
- 次要影响：依赖 ps 输出的脚本和工具

**升级建议**：
- 此 Patch 为 OH 特定适配，不应推向上游
- 升级时需根据新的 OH 进程管理机制调整

---

## 2.5 平台层适配

### Patch 8: LiteOS_A 适配（porting/liteos_a/）

**修改目录**：`porting/liteos_a/`

**适配内容**：针对 LiteOS_A 内核的轻量级系统适配

**修改文件列表**：

| 文件 | 用途 |
|-----|------|
| `Config.in` | 配置选项 |
| `configure` | 构建配置脚本 |
| `Makefile` | Make 构建规则 |
| `toys.h` | 头文件适配 |
| `toys/posix/cp.c` | cp 命令适配 |
| `toys/posix/printf.c` | printf 命令适配 |
| `toys/posix/tail.c` | tail 命令适配 |
| `toys/posix/cksum.c` | cksum 命令适配 |
| `toys/posix/iconv.c` | iconv 命令适配 |
| `toys/posix/uuencode.c` | uuencode 命令适配 |
| `toys/posix/strings.c` | strings 命令适配 |
| `toys/posix/nice.c` | nice 命令适配 |
| `toys/posix/mkfifo.c` | mkfifo 命令适配 |
| `toys/posix/ls.c` | ls 命令适配 |
| `toys/posix/du.c` | du 命令适配 |
| `toys/posix/tee.c` | tee 命令适配 |
| `toys/posix/paste.c` | paste 命令适配 |
| `toys/posix/ps.c` | ps 命令适配 |
| `toys/posix/od.c` | od 命令适配 |
| `toys/posix/nohup.c` | nohup 命令适配 |
| `toys/posix/logger.c` | logger 命令适配 |
| `toys/posix/dd.c` | dd 命令适配 |

**原始问题**：
- LiteOS_A 内核与标准 Linux 内核 API 存在差异
- 某些系统调用和库函数在 LiteOS_A 上不可用或行为不同
- 文件系统和进程管理机制存在差异

**OH 需求**：
- 在轻量级系统（LiteOS_A）上提供基础命令支持
- 适配 LiteOS_A 的有限资源环境
- 确保 20+ 基础命令在 LiteOS_A 上可用

**关键适配点**：
1. **头文件差异**：
   - LiteOS_A 使用不同的头文件（如 `syscall.h`）
   - 缺少某些标准头文件（如 `<sys/statvfs.h>`）

2. **系统调用差异**：
   - 某些系统调用不可用
   - 需要使用 LiteOS_A 特定的系统调用

3. **资源限制**：
   - 内存受限，优化内存使用
   - 栈空间有限，避免递归和大数组

4. **文件系统差异**：
   - LiteOS_A 文件系统功能有限
   - 某些文件操作需要简化

**影响范围**：
- 20+ 个命令在 LiteOS_A 上的可用性
- 轻量系统（small）的命令行支持

**升级建议**：
- 此 Patch 为平台特定适配，不应推向上游
- 升级时需同步更新 `porting/liteos_a/` 目录
- 需要与 LiteOS_A 内核团队协作测试

---

## 2.6 OH 特有实现

### Patch 9: 独立的 su 命令（openharmony/su.c）

**修改文件**：`openharmony/su.c`

**修改类型**：OH 特有实现（独立文件）

**功能描述**：
- 实现用户切换功能
- 支持选项：-l（登录模式），-p（保留环境），-s（指定 shell），-c（执行命令）

**OH 特性**：
```c
// 权限限制：仅 root (uid=0) 和 shell (uid=2000) 可以切换用户
if (current_uid != 0 && current_uid != 2000) {
    fprintf(stderr, "Not allowed\n");
    return -1;
}
```

**原始问题**：
- toybox 自带的 su 命令不符合 OH 的安全要求
- 需要更严格的权限控制

**OH 需求**：
- 提供符合 OpenHarmony 安全策略的 su 命令
- 限制可以切换用户的进程（仅 root 和 shell）
- 安装在 eng_system 镜像（调试版）中

**关键代码变更**：
- 完全独立的 su.c 实现，不依赖 toybox 基础库
- Apache 2.0 许可证（不同于原库的 0BSD）
- 权限检查：仅 uid=0 (root) 和 uid=2000 (shell) 可切换用户
- 环境变量管理：支持 -l（重置环境）和 -p（保留环境）

**主要功能**：
1. **用户验证**：检查目标用户是否存在
2. **权限切换**：使用 `setuid()` 和 `setgid()` 切换用户
3. **环境管理**：根据选项设置环境变量
4. **命令执行**：执行指定的命令或 shell

**影响范围**：
- 主要影响：su 命令
- 仅影响：eng_system 镜像（调试版）

**升级建议**：
- 此 Patch 为 OH 特有实现，不应推向上游
- 升级 toybox 时不影响此文件
- 独立维护，与上游代码无关

---

## 2.7 Patch 分类统计

### 按修改目的分类

| 分类 | 数量 | 占比 | 说明 |
|-----|------|------|------|
| **Bug 修复** | 3 | 33% | 修复明确的 bug（崩溃、日志缺失等） |
| **稳定性提升** | 2 | 22% | 提高系统健壮性（NULL 检查等） |
| **用户体验优化** | 2 | 22% | 改进命令的可用性和易用性 |
| **平台适配** | 1 | 11% | LiteOS_A 内核适配 |
| **OH 特有实现** | 1 | 11% | su 命令独立实现 |

### 按修改层级分类

| 层级 | 数量 | 说明 |
|-----|------|------|
| **基础库层** | 3 | lib/* 的修改，影响多个命令 |
| **命令层** | 4 | toys/* 的修改，影响特定命令 |
| **平台层** | 1 | porting/liteos_a/ 的适配 |
| **OH 特有** | 1 | openharmony/su.c 独立实现 |

### 按影响范围分类

| 影响范围 | 数量 | 说明 |
|---------|------|------|
| **全局影响** | 3 | lib/args.c, lib/dirtree.c, lib/tty.c |
| **单个命令** | 5 | cp/mv, strings, ls, ps |
| **平台特定** | 1 | porting/liteos_a/ |
| **独立实现** | 1 | openharmony/su.c |

---

## 2.8 Patch 质量评估

### 可推向上游的 Patch

| Patch | 优先级 | 理由 |
|------|-------|------|
| **lib/args.c** - 64 位适配 | 高 | 通用修复，适用于所有 64 位系统 |
| **lib/dirtree.c** - NULL 检查 | 高 | 稳定性修复，防止崩溃 |
| **toys/posix/cp.c** - mv -v 日志 | 中 | Bug 修复，提升用户体验 |
| **toys/posix/ls.c** - 块大小计算 | 中 | 更准确的计算方式 |
| **lib/tty.c** - 终端显示 | 低 | 终端特定问题，可能需要更多讨论 |

### OH 特有的 Patch

| Patch | 理由 |
|------|------|
| **openharmony/su.c** | OH 特定的安全策略和权限模型 |
| **toys/posix/ps.c** - OH 进程信息 | OH 特有的进程管理机制 |
| **porting/liteos_a/** - 平台适配 | LiteOS_A 内核特有，不适用于通用 Linux |

---

## 2.9 升级风险评估

### 高风险项

| 风险 | 说明 | 缓解措施 |
|-----|------|---------|
| **95 处条件编译点** | 升级时需逐个检查和应用 | 建立测试矩阵，逐命令验证 |
| **porting/liteos_a/ 适配** | 与 LiteOS_A 内核紧密耦合 | 与内核团队协作测试 |
| **ps 命令适配** | OH 特有进程管理机制 | 确认 OH 进程管理架构是否变化 |

### 中风险项

| 风险 | 说明 | 缓解措施 |
|-----|------|---------|
| **ls 命令优化** | 可能与上游的改进冲突 | 审查上游变更，评估是否冲突 |
| **top 终端适配** | 终端实现可能变化 | 测试不同终端环境 |

### 低风险项

| 风险 | 说明 |
|-----|------|
| **openharmony/su.c** | 独立实现，不影响上游代码 |
| **基础库层修复** | Bug 修复，上游可能已类似修复 |

---

## 2.10 维护建议

### 升级流程

1. **版本评估**：
   - 检查上游版本更新日志
   - 评估是否有重要的 bug 修复或新功能

2. **代码对比**：
   - 对比 OH 修改的文件与上游版本
   - 识别冲突和需要重新应用的修改

3. **分类处理**：
   - 可推向上游的：评估上游是否已修复
   - OH 特有的：必须保留
   - 平台适配：重新测试

4. **测试验证**：
   - 测试所有使用 `TOYBOX_OH_ADAPT` 的命令
   - 重点测试：ls, cp, mv, ps, top
   - 验证 LiteOS_A 适配

### 代码组织建议

1. **减少条件编译点**：
   - 考虑抽取 OH 特有修改为独立模块
   - 降低升级复杂度

2. **统一适配接口**：
   - 为平台适配定义统一接口
   - 减少重复代码

3. **文档化修改**：
   - 每个修改添加详细的注释
   - 说明修改原因和 OH 需求

---

## 参考文档

- [01_Overview.md](./01_Overview.md) - Toybox 库简介
- [03_Build_Integration.md](./03_Build_Integration.md) - OH 构建适配
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估结果
