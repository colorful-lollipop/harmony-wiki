# OH 构建适配

## 3.1 构建系统概述

### 构建系统选择

| 项目 | 详情 |
|------|------|
| **上游构建系统** | GNU Make (Makefile) |
| **OH 构建系统** | GN (Generate Ninja) |
| **OH 构建入口** | BUILD.gn |

### 构建流程对比

```
上游构建：
    Build.sh → Makefile → mksh 可执行文件

OH 构建：
    BUILD.gn → GN → Ninja → mksh 可执行文件
```

---

## 3.2 配置文件结构

### 关键配置文件

```
third_party/mksh/
├── BUILD.gn          # 主要构建配置
├── mksh.gni          # GN 参数配置
├── Build.sh          # 上游构建脚本（保留）
└── check.pl          # 测试脚本
```

### BUILD.gn 结构解析

```gn
# 条件1：轻量级系统 (ohos_lite)
if (defined(ohos_lite)) {
  executable("mksh") {
    # 轻量级系统配置
  }
}

# 条件2：标准系统（默认）
else {
  import("//build/config/ohos/config.gni")
  import("//build/ohos.gni")
  import("mksh.gni")
  
  ohos_executable("sh") {
    # 标准系统配置
  }
}
```

---

## 3.3 编译选项详解

### 通用编译选项

```gn
cflags = [
  "-Wall",                          # 启用所有警告
  "-Wno-deprecated-declarations",  # 禁用废弃声明警告
  "-fno-asynchronous-unwind-tables", # 优化栈展开
  "-fwrapv",                        # 整数溢出包装
  "-fstack-protector-all",           # 栈保护
  "-fPIE",                          # 位置无关可执行文件
]
```

**选项说明**：

| 选项 | 作用 | 安全影响 |
|------|------|----------|
| `-Wall` | 启用所有编译警告 | ✅ 尽早发现问题 |
| `-fwrapv` | 整数溢出定义为定义行为 | ⚠️ 可能有安全影响 |
| `-fstack-protector-all` | 栈保护 | ✅ 防止栈溢出攻击 |
| `-fPIE` | 位置无关可执行文件 | ✅ ASLR 兼容 |

### OH 特定 Defines

```gn
defines = [
  "MKSH_OH_ADAPT",    # OH 适配开关
]

if (mksh_terminal_ext) {
  defines += [
    "MKSH_TERMINAL_EXT",  # 终端扩展功能
  ]
}
```

### 功能配置宏

```gn
# 编译时功能开关
"-DMKSH_ASSUME_UTF8",      # 假设 UTF-8 环境
"-DMKSH_DONT_EMIT_IDSTRING", # 不输出版本字符串
"-DMKSH_BUILDSH",          # 构建脚本模式
"-D_GNU_SOURCE",           # GNU 扩展
"-DSETUID_CAN_FAIL_WITH_EAGAIN", # SETUID 错误处理
```

### 系统特性检测宏

```gn
# HAVE_* 宏表示系统特性检测结果
"-DHAVE_SYS_TIME_H=1",     # 有 sys/time.h
"-DHAVE_TERMIOS_H=1",     # 有 termios.h
"-DHAVE_SELECT=1",         # 有 select()
"-DHAVE_MMAP=1",           # 有 mmap()
# ... 更多特性检测
```

---

## 3.4 源文件配置

### 编译的源文件

```gn
sources = [
  "edit.c",        # 行编辑
  "eval.c",        # 表达式求值
  "exec.c",        # 命令执行
  "expr.c",        # 算术表达式
  "funcs.c",       # 内置函数
  "histrap.c",     # 历史记录
  "jobs.c",        # 作业控制
  "lalloc.c",      # 内存分配
  "lex.c",         # 词法分析
  "main.c",        # 主程序
  "misc.c",        # 杂项
  "shf.c",         # 文件 I/O
  "strlcpy.c",     # 字符串
  "syn.c",         # 语法分析
  "tree.c",         # 语法树
  "var.c",         # 变量
]
```

**注意**：未包含 `os2.c`、`ulimit.c` 等平台特定文件。

### 包含路径

```gn
include_dirs = [
  "./",    # 当前目录（包含 sh.h 等头文件）
]
```

---

## 3.5 链接选项

```gn
ldflags = [
  "-pie",                    # 位置无关可执行文件
  "-Wl,-z,relro",           # 只读重定位
  "-Wl,-z,now",             # 立即绑定
  "-Wl,-z,noexecstack",     # 不可执行栈
]
```

**链接选项说明**：

| 选项 | 作用 | 安全级别 |
|------|------|----------|
| `-pie` | 位置无关可执行文件 | ✅ 高 |
| `-z,relro` | 只读重定位 | ✅ 高 |
| `-z,now` | 立即绑定符号 | ✅ 高 |
| `-z,noexecstack` | 不可执行栈 | ✅ 高 |

---

## 3.6 mksh.gni 参数配置

### 参数定义

```gn
# mksh.gni
declare_args() {
  mksh_terminal_ext = false  # 默认关闭终端扩展
}
```

### 使用方式

在 OH 构建配置中启用终端扩展：

```gn
# 在系统配置中启用
mksh_terminal_ext = true
```

### 影响范围

启用 `MKSH_TERMINAL_EXT` 后：

| 文件 | 影响 |
|------|------|
| edit.c | 终端清除字符串、键绑定修改 |
| main.c | 终端扩展初始化 |

---

## 3.7 安装配置

### 安装目标

```gn
install_images = [
  "system",    # 系统镜像
  "ramdisk",   # 内存盘
  "updater",   # 更新程序
]

part_name = "mksh"       # 部件名称
install_enable = true    # 启用安装
```

### 安装路径

编译产物将安装到 OH 系统的以下位置：

- `/bin/sh` - 默认 shell
- `/usr/bin/mksh` - mksh 本身

---

## 3.8 与上游构建的差异

### 构建脚本差异

| 方面 | 上游 Build.sh | OH BUILD.gn |
|------|--------------|-------------|
| 系统 | Unix/Linux | OHOS |
| 构建工具 | make | gn + ninja |
| 配置方式 | 编译时检测 | 预定义宏 |
| 输出 | mksh 可执行文件 | sh (ohos_executable) |

### 功能差异

| 功能 | 上游 | OH |
|------|------|-----|
| 平台检测 | 运行时检测 | 编译时定义 |
| 终端扩展 | 默认启用 | 可选启用 |
| OH 适配 | 无 | 有 (MKSH_OH_ADAPT) |

---

## 3.9 构建故障排查

### 常见问题

#### 问题 1：编译警告过多

**症状**：大量 `-Wdeprecated-declarations` 警告

**解决方案**：
```gn
cflags = [
  "-Wno-deprecated-declarations",  # 添加此选项
]
```

#### 问题 2：链接失败

**症状**：`cannot find -lxxx` 或类似错误

**排查步骤**：
1. 检查 `ldflags` 配置
2. 确认依赖库已正确链接
3. 检查库搜索路径

#### 问题 3：运行时崩溃

**症状**：`exec -a0` 导致崩溃

**原因**：未启用 `MKSH_OH_ADAPT`

**解决方案**：
```gn
defines = [
  "MKSH_OH_ADAPT",  # 确保已定义
]
```

---

## 3.10 构建性能优化

### 编译时间优化

| 优化项 | 当前配置 | 建议 |
|--------|----------|------|
| 源文件数量 | 16 个 | 无需优化 |
| 优化级别 | 默认 (-O0/-O2) | 使用 -O2 |
| 并行编译 | Ninja 自动处理 | 无需手动配置 |

### 产物大小优化

| 优化项 | 当前 | 建议 |
|--------|------|------|
| Strip | 未配置 | 添加 `-s` 链接选项 |
| Debug Info | 包含 | 发布版可移除 |

---

## 3.11 版本与构建信息

```gn
# 构建版本标识
"-DMKSH_BUILD_R=593",    # 构建版本号
"-DMKSH_UNLIMITED",     # 无限制模式
```

### 版本信息获取

```bash
# 查看 mksh 版本
./bin/mksh -c 'echo $KSH_VERSION'
```

输出示例：`@(#)MIRBSD KSH R59c`
