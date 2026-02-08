# ncurses API 差异分析

## 概述

经过对 ncurses Patch 的详细分析，**未发现** OpenHarmony 对 ncurses API 的修改或扩展。

### API 变更情况

| 变更类型 | 数量 | 说明 |
|----------|------|------|
| **新增 API** | 0 | 未添加新函数或宏 |
| **修改 API** | 0 | 未修改现有函数行为 |
| **废弃 API** | 0 | 未废弃任何接口 |
| **行为变更** | 0 | 未改变 API 语义 |

### Patch 对 API 的影响

ncurses 的 6 个 Patch **均未涉及 API 层面的修改**：

| Patch 文件 | 修改范围 | 是否影响 API |
|-----------|----------|-------------|
| ncurses-kbs.patch | `misc/terminfo.src` | ❌ 否 (仅终端定义) |
| ncurses-urxvt.patch | `misc/terminfo.src` | ❌ 否 (仅终端定义) |
| ncurses-libs.patch | `Makefile.in` | ❌ 否 (仅构建配置) |
| ncurses-config.patch | `gen-pkgconfig.in`, `ncurses-config.in` | ❌ 否 (仅配置脚本) |
| cross_compile_support_ohos.patch | `config.sub`, `configure` | ❌ 否 (仅构建系统) |
| backport-0002-CVE-2023-29491-env-access.patch | `access.c` | ❌ 否 (仅内部实现) |

---

## 终端能力数据库变更

虽然 API 没有变更，但 Patch 修改了终端能力数据库 (`misc/terminfo.src`)，这可能影响应用程序在特定终端上的行为。

### 终端类型增强

#### 1. rxvt 退格键修复 (ncurses-kbs.patch)

**变更内容**:
```
rxvt-basic: 添加 use=xterm+kbs
screen.teraterm: 添加 kbs=^H
```

**对应用的影响**:
- 使用 `rxvt` 终端类型的应用：退格键现在正确发送 `^H` (Ctrl+H)
- 使用 `screen.teraterm` 的应用：退格键行为修复

**兼容性**:
- ✅ 向后兼容：修复了之前不正确的行为
- ✅ 对不使用这些终端的应用无影响

#### 2. rxvt-unicode 支持 (ncurses-urxvt.patch)

**变更内容**:
- 新增 `rxvt-unicode` (urxvt) 终端类型定义
- 支持 88 色、7744 色对
- 完整功能键映射

**对应用的影响**:
- 使用 `TERM=rxvt-unicode` 的应用现在可以正确识别终端能力
- 支持 urxvt 特有的功能（如扩展颜色、鼠标支持）

**兼容性**:
- ✅ 向后兼容：新增终端类型，不影响现有类型
- ✅ 应用可显式使用 `rxvt-unicode` 获得更好体验

---

## 内部实现变更

### CVE-2023-29491 修复

**变更文件**: `ncurses/tinfo/access.c`

**变更内容**:
```c
// 移除的代码
#if !defined(USE_ROOT_ENVIRON)
    if ((getuid() == ROOT_UID) || (geteuid() == ROOT_UID)) {
        result = FALSE;
    }
#endif
```

**行为变更**:
- **之前**: root 用户 (`UID==0`) 不能使用 `TERMINFO`/`TERMINFO_DIRS` 环境变量
- **之后**: root 用户和普用户统一处理环境变量

**对应用的影响**:
- root 用户运行的 ncurses 应用现在可以读取 `TERMINFO` 环境变量
- 这**修复了**之前 root 用户可能使用错误 terminfo 数据库的问题

**注意**: 这是安全修复，不是 API 变更，但改变了运行时行为。

---

## 与上游 API 兼容性

### 完全兼容

OpenHarmony 的 ncurses 与上游 ncurses 6.5 **100% API 兼容**：

```c
// 所有标准 ncurses API 保持不变
#include <ncurses.h>

// 初始化
initscr();
start_color();
cbreak();
noecho();
keypad(stdscr, TRUE);

// 输出
printw("Hello, World!");
refresh();

// 输入
int ch = getch();

// 结束
endwin();
```

### 函数签名保持不变

所有标准函数签名与上游一致：

| 函数 | 签名 | 状态 |
|------|------|------|
| `initscr` | `WINDOW *initscr(void)` | ✅ 未变更 |
| `printw` | `int printw(const char *fmt, ...)` | ✅ 未变更 |
| `getch` | `int getch(void)` | ✅ 未变更 |
| `refresh` | `int refresh(void)` | ✅ 未变更 |
| `endwin` | `int endwin(void)` | ✅ 未变更 |
| `newwin` | `WINDOW *newwin(int nlines, int ncols, int begin_y, int begin_x)` | ✅ 未变更 |
| `box` | `int box(WINDOW *win, chtype verch, chtype horch)` | ✅ 未变更 |

### 数据结构保持不变

```c
// WINDOW 结构不变
typedef struct _win_st WINDOW;

// chtype 不变
typedef unsigned long chtype;

// attr_t 不变
typedef chtype attr_t;
```

### 宏定义保持不变

```c
// 颜色相关宏
#define COLOR_BLACK     0
#define COLOR_RED       1
#define COLOR_GREEN     2
// ... 等等

// 属性宏
#define A_NORMAL        0
#define A_STANDOUT      0x00100000
#define A_UNDERLINE     0x00200000
// ... 等等
```

---

## 对应用开发者的影响

### 移植到 OH 的考虑

由于 API 完全兼容，将使用 ncurses 的应用移植到 OpenHarmony 时：

#### ✅ 无需修改的方面

1. **源代码**: 直接使用 ncurses API 的代码无需修改
2. **编译**: 使用相同的编译命令和链接选项
3. **运行时**: 应用行为与在其他平台一致

#### ⚠️ 需要注意的方面

1. **终端类型**: 如果应用依赖特定终端能力，确认 OH 包含所需的 terminfo 定义
2. **环境变量**: root 用户现在可以读取 `TERMINFO` 环境变量（CVE 修复后的行为）

### 使用建议

```c
// 标准 ncurses 程序在 OH 上无需修改
#include <ncurses.h>

int main() {
    // 初始化
    initscr();
    
    // 检查终端能力
    if (has_colors()) {
        start_color();
        init_pair(1, COLOR_RED, COLOR_BLACK);
    }
    
    // 使用颜色
    attron(COLOR_PAIR(1));
    printw("Hello from OpenHarmony!");
    attroff(COLOR_PAIR(1));
    
    refresh();
    getch();
    endwin();
    
    return 0;
}

// 编译命令（假设 ncurses 安装在 /system）
// cc -o myapp myapp.c -I/system/include -L/system/lib -lncurses
```

---

## 与上游版本的功能差异

### 功能一致性

| 功能 | 上游 6.5 | OH 6.5 | 差异 |
|------|----------|--------|------|
| 核心 curses API | ✅ | ✅ | 无 |
| 颜色支持 | ✅ | ✅ | 无 |
| 宽字符支持 | ✅ | ✅ | 无 |
| 鼠标支持 | ✅ | ✅ | 无 |
| 面板 (panel) | ✅ | ✅ | 无 |
| 菜单 (menu) | ✅ | ✅ | 无 |
| 表单 (form) | ✅ | ✅ | 无 |
| rxvt 支持 | ✅ | ✅+ | 有增强 |
| urxvt 支持 | ❌ | ✅ | OH 新增 |

### 终端支持差异

OH 版本相比上游额外支持：
- `rxvt-unicode` 终端类型（完整定义）
- 改进的 `rxvt` 退格键处理
- 改进的 `screen.teraterm` 退格键处理

---

## 总结

### API 状态

```
┌────────────────────────────────────────────────────────────┐
│                    API 差异总结                             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│   标准 ncurses API:                                        │
│   ┌──────────────────────────────────────────────────┐    │
│   │  无变更 - 100% 兼容上游 6.5                       │    │
│   └──────────────────────────────────────────────────┘    │
│                                                            │
│   终端能力数据库:                                          │
│   ┌──────────────────────────────────────────────────┐    │
│   │  • 新增 rxvt-unicode 支持                         │    │
│   │  • 修复 rxvt 退格键处理                           │    │
│   │  • 修复 screen.teraterm 退格键处理                │    │
│   └──────────────────────────────────────────────────┘    │
│                                                            │
│   内部实现:                                                │
│   ┌──────────────────────────────────────────────────┐    │
│   │  • CVE-2023-29491 安全修复 (env 访问控制)         │    │
│   │  • 无 API 变更                                    │    │
│   └──────────────────────────────────────────────────┘    │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

### 开发者结论

对于使用 ncurses 的开发者：

1. **源代码兼容性**: ✅ 100% 兼容，无需修改
2. **二进制兼容性**: ✅ 与上游 ncurses 6.5 兼容
3. **功能增强**: urxvt 用户可获得更好的终端支持
4. **安全性**: 包含关键 CVE 修复，更安全

OpenHarmony 的 ncurses 是上游 6.5 的**完整兼容实现**，仅增加了终端类型支持和安全修复，没有 API 层面的任何修改。
