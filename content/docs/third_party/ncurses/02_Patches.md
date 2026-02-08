# ncurses Patch 详细分析

## Patch 总览

ncurses 在 OpenHarmony 中应用了 **6 个 Patch**，可分为以下几类：

| 类别 | Patch 数量 | 说明 |
|------|-----------|------|
| **终端类型增强** | 2 | 添加 rxvt-unicode 支持，修复退格键处理 |
| **构建系统适配** | 2 | 调整库链接参数，简化配置脚本 |
| **OH 平台适配** | 1 | 添加 OpenHarmony 交叉编译支持 |
| **安全修复** | 1 | CVE-2023-29491 漏洞修复 |

---

## Patch 清单表

| Patch 文件 | 修改文件 | 修改类型 | 主要目的 | OH 相关性 |
|-----------|----------|----------|----------|-----------|
| [ncurses-kbs.patch](#ncurses-kbspatch) | `misc/terminfo.src` | 功能增强 | 修复 rxvt 和 screen.teraterm 的退格键处理 | 通用修复 |
| [ncurses-urxvt.patch](#ncurses-urxvtpatch) | `misc/terminfo.src` | 功能增强 | 添加 rxvt-unicode 终端类型定义 | 通用修复 |
| [ncurses-libs.patch](#ncurses-libspatch) | `c++/Makefile.in`, `form/Makefile.in`, `menu/Makefile.in`, `panel/Makefile.in` | 构建修复 | 调整共享库链接参数 | OH 适配 |
| [ncurses-config.patch](#ncurses-configpatch) | `misc/gen-pkgconfig.in`, `misc/ncurses-config.in` | 构建修复 | 简化 pkg-config 和 ncurses-config 输出 | **OH 特有** |
| [cross_compile_support_ohos.patch](#cross_compile_support_ohospatch) | `config.sub`, `configure` | OH 适配 | 添加 OpenHarmony 交叉编译支持 | **OH 特有** |
| [backport-0002-CVE-2023-29491-env-access.patch](#cve-2023-29491-patch) | `ncurses/tinfo/access.c` | 安全修复 | 修复 CVE-2023-29491 安全漏洞 | 安全修复 |

---

## 详细 Patch 分析

### ncurses-kbs.patch

**修改文件**: `misc/terminfo.src`

**修改摘要**:
```diff
# 在 rxvt-basic 定义中添加 xterm+kbs
- use=rxvt+pcfkeys, use=vt220+cvis, use=vt220+keypad,
+ use=rxvt+pcfkeys, use=vt220+cvis, use=vt220+keypad, use=xterm+kbs,

# 在 screen.teraterm 中添加退格键定义
+ kbs=^H,
```

**原始问题**:
- rxvt 终端模拟器在默认配置下退格键行为异常
- screen.teraterm 类型缺少明确的退格键定义

**修改内容**:
1. 为 `rxvt-basic` 终端类型添加 `xterm+kbs` 能力引用，统一退格键处理
2. 为 `screen.teraterm` 显式定义 `kbs=^H` (Ctrl+H，即退格字符)

**OH 价值**:
- 改善终端兼容性，确保退格键在 rxvt 和 screen.teraterm 环境下正常工作
- 通用修复，提升用户体验

**回归风险**: 低 - 纯功能增强，无破坏性变更

**升级建议**: 可尝试推向上游，属于通用改进

---

### ncurses-urxvt.patch

**修改文件**: `misc/terminfo.src`

**修改摘要**: 添加完整的 rxvt-unicode (urxvt) 终端类型定义（约 170 行）

**关键字段**:
```
rxvt-unicode|rxvt-unicode terminal (X Window System),
    am, bce, eo, km, msgr, xenl, hs,
    cols#80, lines#24,
    colors#88, pairs#7744,
    # ... 完整终端能力定义
```

**原始问题**:
- ncurses 上游未包含 rxvt-unicode 终端类型的完整定义
- urxvt 用户在 ncurses 应用中遇到显示或键盘问题

**修改内容**:
- 基于 rxvt-unicode 官方 terminfo 定义添加完整条目
- 包含：88 色支持、7744 色对、完整功能键映射、光标控制序列等

**OH 价值**:
- 支持 rxvt-unicode 终端模拟器，提供更好的终端兼容性
- 适用于在 OH 上使用 urxvt 的开发者

**回归风险**: 低 - 新增定义，不影响现有终端类型

**升级建议**: 可尝试推向上游，属于通用改进

---

### ncurses-libs.patch

**修改文件**:
- `c++/Makefile.in`
- `form/Makefile.in`
- `menu/Makefile.in`
- `panel/Makefile.in`

**修改摘要**:
```diff
# c++/Makefile.in
- -lncurses@USE_LIB_SUFFIX@ @SHLIB_LIST@
+ -lncurses@USE_LIB_SUFFIX@ #@SHLIB_LIST@

- LDFLAGS = $(TEST_ARGS) @LDFLAGS@ \
-     @LD_MODEL@ $(TEST_LIBS) @LIBS@ $(CXXLIBS)
+ LDFLAGS = @LDFLAGS@ @LD_MODEL@ @LIBS@ $(CXXLIBS)

# form/menu/panel/Makefile.in
- SHLIB_LIST = $(SHLIB_DIRS) -lncurses@USE_LIB_SUFFIX@ @SHLIB_LIST@
+ SHLIB_LIST = $(SHLIB_DIRS) -lncurses@USE_LIB_SUFFIX@ #@SHLIB_LIST@
```

**原始问题**:
- 原始 Makefile 链接过多系统库，可能导致：
  - 循环依赖问题
  - 不必要的库被链接
  - 与 OpenHarmony 库结构的兼容性问题

**修改内容**:
1. 注释掉 `@SHLIB_LIST@` 宏，避免链接额外的共享库
2. 简化 c++ 库的 LDFLAGS，移除 `$(TEST_ARGS)` 和 `$(TEST_LIBS)`

**OH 价值**:
- 解决 OpenHarmony 构建环境下的库链接问题
- 减少不必要的依赖，使库更轻量

**回归风险**: 中 - 需要验证在 OH 环境下功能完整性

**升级建议**: **OH 特有 Patch**，升级时需重新适配

---

### ncurses-config.patch

**修改文件**:
- `misc/gen-pkgconfig.in`
- `misc/ncurses-config.in`

**修改摘要**:
```diff
# gen-pkgconfig.in
- for opt in -L$libdir @EXTRA_PKG_LDFLAGS@ @LIBS@
+ for opt in -L$libdir @LIBS@

# ncurses-config.in
- libdir="@libdir@"
  
- for opt in -L$libdir @EXTRA_PKG_LDFLAGS@ $LIBS
+ for opt in $LIBS

- @LD_SEARCHPATH@) # skip standard libdir
+ ////) # skip standard libdir (disabled for multilib)

- LIBDIRS="@LD_SEARCHPATH@"
+ LIBDIRS=""

- echo "${libdir}"  # --libdir 输出
+  # 空输出
```

**原始问题**:
- ncurses-config 和 pkg-config 脚本输出包含完整路径和额外链接参数
- 这些输出可能与 OpenHarmony 的多库架构冲突
- `libdir` 和搜索路径的硬编码可能导致配置错误

**修改内容**:
1. 从库标志循环中移除 `@EXTRA_PKG_LDFLAGS@`
2. 移除 `libdir` 变量定义
3. 禁用标准库目录搜索（替换为 `////`）
4. `--libdir` 选项返回空值

**OH 价值**:
- **核心 OH 适配**: 简化配置脚本输出，适配 OH 的库路径结构
- 避免硬编码路径与 OH 构建系统冲突
- 使 ncurses 能更好地集成到 OH 的构建流程

**回归风险**: 中 - 需要确保其他工具不依赖这些输出

**升级建议**: **OH 特有 Patch**，升级时需重新适配

---

### cross_compile_support_ohos.patch

**修改文件**:
- `config.sub`
- `configure`

**修改摘要**:
```diff
# config.sub
- fiwix* | mlibc* | cos* | mbr* | ironclad* )
+ fiwix* | mlibc* | cos* | mbr* | ironclad* | ohos* )

+ *-ohos*-)
+     ;;

# configure
+ if test "${with_strip_program+set}" = set; then
+   INSTALL_OPT_S="$INSTALL_OPT_S --strip-program=$with_strip_program"
+ fi
```

**原始问题**:
- ncurses 上游不支持 `ohos` 作为有效的目标平台
- 交叉编译到 OpenHarmony 时 configure 脚本失败

**修改内容**:
1. 在 `config.sub` 中添加 `ohos*` 到有效操作系统列表
2. 添加 `*-ohos*-` 内核-OS-对象处理规则
3. 添加 `--with-strip-program` 选项支持，用于交叉编译时的 strip 工具指定

**OH 价值**:
- **关键 OH 适配**: 使 ncurses 能够在 OpenHarmony 上进行交叉编译
- 支持使用 OH 工具链进行构建

**标记说明**:
- 在 `ncurses.spec` 中明确标记为 `# OHOS_LOCAL`
- 表示这是 OpenHarmony 特有的本地修改

**回归风险**: 低 - 纯新增支持，不影响其他平台

**升级建议**: **OH 特有 Patch**，每次升级必须保留

---

### backport-0002-CVE-2023-29491-env-access.patch

**修改文件**: `ncurses/tinfo/access.c`

**修改摘要**:
```diff
- #if !defined(USE_ROOT_ENVIRON)
-     if ((getuid() == ROOT_UID) || (geteuid() == ROOT_UID)) {
-         result = FALSE;
-     }
- #endif
```

**安全漏洞详情**:

| 项目 | 内容 |
|------|------|
| **CVE ID** | CVE-2023-29491 |
| **类型** | 环境变量访问控制绕过 |
| **影响版本** | ncurses < 6.4-20230408 |
| **严重程度** | 中等 |

**原始问题**:
- `_nc_env_access()` 函数检查 `TERMINFO` 和 `TERMINFO_DIRS` 环境变量是否应该被使用
- 原代码对 root 用户有特殊处理：如果 UID/EUID 为 0，则禁用环境变量访问
- 这种处理方式存在安全风险，可能被本地攻击者利用

**漏洞原理**:
- root 检查 (`getuid() == ROOT_UID`) 可能被绕过
- 环境变量访问控制不一致可能导致特权升级或信息泄露

**修复内容**:
- 移除对 root 用户的特殊处理代码块
- 统一环境变量访问策略，无论是否 root 用户都使用相同的检查逻辑

**OH 价值**:
- **关键安全修复**: 修补已知安全漏洞
- 确保 OpenHarmony 系统的安全性

**回归风险**: 低 - 安全修复，符合安全最佳实践

**升级建议**:
- 此 Patch 是上游安全修复的回迁 (backport)
- 当升级到包含此修复的上游版本时可移除
- 持续跟踪上游安全公告

---

## Patch 分类与维护建议

### 按类型分类

| 类型 | Patch | 是否可推向上游 | 维护策略 |
|------|-------|---------------|----------|
| 终端类型增强 | ncurses-kbs.patch, ncurses-urxvt.patch | 是 | 尝试推向上游 |
| 构建系统适配 | ncurses-libs.patch, ncurses-config.patch | 否 | OH 特有，保留 |
| 平台适配 | cross_compile_support_ohos.patch | 否 | OH 特有，保留 |
| 安全修复 | CVE-2023-29491.patch | 已在上游 | 升级后移除 |

### 升级时的注意事项

#### 1. 保留 OH 特有 Patch
以下 Patch 必须在升级后重新应用：
- `cross_compile_support_ohos.patch` - 平台支持
- `ncurses-config.patch` - 配置脚本适配
- `ncurses-libs.patch` - 库链接适配

#### 2. 检查上游已合并的 Patch
- 定期检查上游是否已合并终端类型增强 Patch
- 确认 CVE-2023-29491 修复是否已包含在新版本中

#### 3. 验证构建
- 升级后在 OH 构建环境下进行完整编译测试
- 验证所有工具 (tic, infocmp, tput 等) 功能正常

### Patch 管理建议

```
ncurses/
├── patches/
│   ├── 0001-ncurses-kbs.patch          # 终端类型增强
│   ├── 0002-ncurses-urxvt.patch        # 终端类型增强
│   ├── 0003-ncurses-libs.patch         # OH 构建适配
│   ├── 0004-ncurses-config.patch       # OH 构建适配
│   ├── 0005-cross-compile-ohos.patch   # OH 平台适配 [OHOS_LOCAL]
│   └── 0006-CVE-2023-29491.patch       # 安全修复 [backport]
└── ncurses.spec                        # 定义 Patch 应用顺序
```

---

## 总结

ncurses 的 6 个 Patch 覆盖了终端兼容性、构建系统适配、平台支持和安全修复四个方面。其中：

1. **2 个 Patch** 是通用功能增强，可尝试推向上游
2. **3 个 Patch** 是 OH 特有适配，升级时必须保留
3. **1 个 Patch** 是关键安全修复，需持续跟踪上游

所有 Patch 都经过了审慎的评估，确保在提升功能的同时保持与 OpenHarmony 架构的兼容性。
