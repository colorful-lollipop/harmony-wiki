# OH 构建集成

> Toybox 通过 BUILD.gn 适配 OpenHarmony 构建系统，支持标准系统和轻量系统。

---

## 3.1 BUILD.gn 结构

### 主 BUILD.gn 文件

**文件路径**：`third_party/toybox/BUILD.gn`

**结构概览**：
```gn
if (defined(ohos_lite)) {
  # LiteOS_A 构建配置
  executable("toybox") { ... }
  foreach(path, cmd_long_path) {
    exec_script("install.py", ...)  # 创建符号链接
  }
} else {
  # 标准系统构建配置
  import("//build/ohos.gni")
  import("toybox.gni")

  ohos_executable("su") { ... }      # 独立的 su 命令
  ohos_executable("toybox") { ... }   # 主可执行文件
}
```

### 构建目标

| 目标名称 | 类型 | 说明 | 产物 |
|----------|------|------|------|
| `toybox` | ohos_executable | 主可执行文件，提供 200+ 命令 | `toybox` 可执行文件 + 符号链接 |
| `su` | ohos_executable | 用户切换命令 | `su` 可执行文件 |

---

## 3.2 源文件管理

### 标准系统源文件列表

**文件路径**：`toys/posix/*.c`, `toys/lsb/*.c`, `toys/other/*.c`, `toys/net/*.c`, `toys/pending/*.c`, `lib/*.c`

**主要源文件**：
- **库文件**（lib/*）：共 17 个
  - `lib/args.c` - 命令行参数解析
  - `lib/dirtree.c` - 目录遍历
  - `lib/elf.c` - ELF 文件处理
  - `lib/hash.c` - 哈希计算
  - `lib/lib.c` - 基础库函数
  - `lib/net.c` - 网络相关
  - `lib/password.c` - 密码处理
  - `lib/portability.c` - 可移植性适配
  - `lib/tty.c` - 终端控制
  - `lib/utf8.c` - UTF-8 支持
  - `lib/xwrap.c` - 包装函数
  - 等等...

- **命令文件**（toys/*）：共 190+ 个
  - **posix/**：ls, cp, mv, cat, ps, grep, sed 等标准命令
  - **lsb/**：dmesg, gzip, hostname, mount, ping 等命令
  - **other/**：free, stat, chroot, login 等命令
  - **net/**：ifconfig, netstat, netcat, ping 等网络命令
  - **pending/**：awk, diff, wget, telnet 等扩展命令
  - **android/**：sendevent 等特定命令

### su 命令源文件

**文件路径**：`openharmony/su.c`

**特点**：
- 独立的源文件，不依赖 toybox 基础库
- Apache 2.0 许可证
- OH 特有的安全策略实现

### LiteOS_A 适配源文件

**文件路径**：`porting/liteos_a/`

**适配的命令**：cp, printf, tail, cksum, iconv, uuencode, strings, nice, mkfifo, ls, du, tee, paste, ps, od, nohup, logger, dd 等 20+ 个命令

---

## 3.3 编译选项

### C 编译标志

```gn
cflags_c = [
  "-std=gnu11",                     # C11 标准
  "-Wall",                          # 所有警告
  "-Wundef",                        # 未定义宏警告
  "-Wno-char-subscripts",            # 忽略字符下标警告
  "-Wno-implicit-function-declaration",  # 忽略隐式函数声明
  "-Wno-unused-variable",           # 忽略未使用变量警告
  "-Wno-unused-value",              # 忽略未使用值警告
  "-Wno-incompatible-pointer-types", # 忽略不兼容指针类型
  "-Wno-int-conversion",           # 忽略整数转换警告
  "-Wno-sign-compare",             # 忽略符号比较警告
  "-Wno-format",                   # 忽略格式警告
  "-Wno-unused-result",            # 忽略未使用结果警告
  "-Os",                           # 优化体积
  "-ffunction-sections",            # 函数分段
  "-fdata-sections",               # 数据分段
  "-fno-asynchronous-unwind-tables", # 禁用异步展开表
  "-fPIE",                        # 位置无关可执行文件
  "-funsigned-char",               # 无符号字符
  "-Wno-string-plus-int",          # 忽略字符串+整数警告
  "-Wno-tautological-constant-compare", # 忽略常量比较警告
  "-Wno-string-conversion",        # 忽略字符串转换警告
  "-Wno-unused-but-set-variable",  # 忽略设置但未使用的变量
]
```

**说明**：
- **`-Os`**：优化体积，减少 ROM 占用
- **`-ffunction-sections -fdata-sections`**：将函数和数据分段，便于链接器去除未使用代码
- **`-fPIE`**：生成位置无关可执行文件，支持 ASLR（地址空间布局随机化）
- **大量 `-Wno-*`**：抑制 toybox 代码中的警告，避免编译失败

### 链接标志

```gn
ldflags = [
  "-pie",                  # 位置无关可执行文件
  "-Wl,-z,relro",         # 重定位后只读
  "-Wl,-z,now",           # 立即绑定
  "-Wl,-z,noexecstack",    # 不可执行栈
  "-lm",                  # 数学库
  "-lcrypt",              # 加密库
]
```

**安全加固说明**：

| 标志 | 安全作用 | 说明 |
|-----|---------|------|
| **`-pie`** | ASLR 支持 | 可执行文件加载地址随机化，防止 ROP 攻击 |
| **`-Wl,-z,relro`** | RELRO 保护 | 重定位表只读，防止 GOT 覆盖 |
| **`-Wl,-z,now`** | 立即绑定 | 禁用延迟绑定，结合 RELRO 使用 |
| **`-Wl,-z,noexecstack`** | NX 保护 | 栈不可执行，防止 shellcode 注入 |

### 宏定义

#### 基础宏

```gn
defines = [
  "_DEFAULT_SOURCE",      # 启用默认源码特性
  "TOYBOX_OH_ADAPT",    # OH 适配宏（95 处使用）
]
```

#### LiteOS 特定宏

```gn
if (defined(ohos_lite)) {
  defines += [
    "OHOS_LITE",         # 轻量系统适配
    "TOYBOX_OH_ADAPT",
  ]
}
```

#### SELinux 宏

```gn
if (build_selinux) {
  cflags_c += [
    "-D_GNU_SOURCE",     # 启用 GNU 扩展
    "-DUSE_PCRE2",      # 使用 PCRE2 正则表达式库
    "-w",               # 忽略警告
    "-DWITH_SELINUX",    # 启用 SELinux 支持
    "-DCUSTOM_GET_CONTEXT",  # 自定义上下文获取
  ]
}
```

#### 扩展命令宏

```gn
if (toybox_extended_cmd) {
  defines += [ "TOYBOX_EXTENDED_CMD" ]
}
```

#### brctl 宏

```gn
if (toybox_enable_brctl) {
  defines += [ "TOYBOX_ENABLE_BRCTL" ]
}
```

#### 构建变体宏

```gn
if (build_variant == "user") {
  defines += [ "TOYBOX_BUILD_USER" ]
}
```

---

## 3.4 功能开关

### 扩展命令支持（toybox_extended_cmd）

**启用命令**：
- `wget` - HTTP/FTP 下载工具
- `awk` - 文本处理工具
- `diff` - 文件比较工具
- `expr` - 表式计算
- `getfattr` - 获取扩展属性
- `ipcs` - 进程间通信状态
- `telnet` - Telnet 客户端
- `tr` - 字符转换
- `traceroute` - 路由追踪
- `netcat` - 网络工具
- `route` - 路由管理

**额外依赖**：
```gn
external_deps = [
  "openssl:libcrypto_shared",  # OpenSSL 加密库
  "openssl:libssl_shared",    # OpenSSL SSL/TLS 库
  "selinux:libselinux",      # SELinux 库
]
```

**使用场景**：
- 网络调试（wget, telnet, traceroute）
- 文本处理（awk, diff, tr）
- 系统管理（ipcs, getfattr）

### brctl 支持（toybox_enable_brctl）

**启用命令**：
- `brctl` - 网桥管理工具

**使用场景**：
- 网络桥接配置
- 虚拟网络环境

### usr 符号链接支持（toybox_feature_support_usr_symlink）

**功能**：
- 创建到 `/usr/bin` 的符号链接
- 符合 FHS（文件系统层次结构标准）

**示例**：
```
/bin/toybox -> toybox 可执行文件
/usr/bin/ls -> /bin/toybox
/usr/bin/cat -> /bin/toybox
...
```

**使用场景**：
- 与标准 Linux 系统保持一致
- 兼容依赖 `/usr/bin` 的脚本和工具

### SELinux 支持（build_selinux）

**启用命令**：
- `chcon` - 修改文件上下文
- `restorecon` - 恢复文件上下文

**额外依赖**：
```gn
sources += [ "toys/other/chcon.c" ]
external_deps = [
  "openssl:libcrypto_shared",
  "openssl:libssl_shared",
  "selinux:libselinux",
]
```

**使用场景**：
- SELinux 策略管理
- 文件上下文维护

---

## 3.5 安装配置

### 安装位置

#### toybox 可执行文件

```gn
install_images = [
  "system",      # 标准系统运行时
  "ramdisk",     # 系统启动早期环境
  "updater",     # 系统更新工具
]
install_enable = true
```

**安装路径**：
- 标准系统：`/system/bin/toybox`
- 轻量系统：`/bin/toybox`

#### su 可执行文件

```gn
part_name = "toybox"
subsystem_name = "thirdparty"
install_images = [ "eng_system" ]  # 仅调试版
install_enable = true
```

**安装路径**：`/system/bin/su`（仅 eng_system 镜像）

**安全说明**：
- su 命令仅在调试版镜像中可用
- 生产版本不包含 su 命令
- 只有 root (uid=0) 和 shell (uid=2000) 可执行

### 符号链接生成

#### 标准系统

**自动生成**：通过 OH 构建系统自动创建符号链接

**链接目标**：约 200+ 个命令

```gn
symlink_target_name = [
  "acpi", "ascii", "base64", "basename", "blockdev",
  "bunzip2", "bzcat", "cal", "cat", "chattr", "chcon",
  "chgrp", "chmod", "chown", "chroot", "chrt", "chvt",
  # ... 共 200+ 个命令
]
```

**符号链接示例**：
```
/bin/ls -> toybox
/bin/cat -> toybox
/bin/cp -> toybox
/bin/mv -> toybox
...
```

#### LiteOS_A

**手动生成**：通过 install.py 脚本创建

```gn
cmd_long_path = [
  "bin/chmod", "bin/chown", "bin/chroot",
  # ... 约 250+ 个命令
]

foreach(path, cmd_long_path) {
  exec_script("install.py",
              [
                "--long_path",
                path,
                "--out_dir",
                rebase_path("$root_out_dir"),
              ])
}
```

---

## 3.6 多调用二进制机制

### 工作原理

Toybox 采用多调用二进制（multicall binary）模式：

```
┌─────────────────────────────────────┐
│       toybox 可执行文件            │
│                                 │
│  main() -> toy_exec()            │
│    ↓                            │
│  根据 argv[0] 确定命令           │
│    ↓                            │
│  调用对应的 xxx_main() 函数      │
└─────────────────────────────────────┘
         ↑ ↑ ↑ ↑ ↑
         │ │ │ │ │
    ls cat cp mv ps  (符号链接)
```

### 符号链接机制

**创建方式**：
```bash
# 通过 OH 构建系统自动创建
ln -s toybox ls
ln -s toybox cat
ln -s toybox cp
...
```

**执行流程**：
```bash
# 用户执行
$ ls -l /bin/ls

# 符号链接指向
lrwxrwxrwx 1 root root 6 Feb  8 03:17 /bin/ls -> toybox

# 实际执行
$ /bin/ls -l

# toybox 内部处理
argv[0] = "/bin/ls"
-> 提取命令名："ls"
-> 查找并调用 ls_main() 函数
```

### 优势

| 优势 | 说明 |
|-----|------|
| **节省空间** | 单个二进制文件（~73KB） vs 200+ 个独立文件（~2MB） |
| **启动快速** | 共享代码段，减少 I/O 和加载时间 |
| **易于维护** | 统一的基础库，减少代码重复 |
| **动态扩展** | 添加新命令只需添加源文件和符号链接 |

---

## 3.7 与上游构建系统的差异

### 上游构建系统（Make）

**配置文件**：`.config`, `Config.in`

**构建命令**：
```bash
make defconfig    # 生成默认配置
make menuconfig   # 交互式配置
make             # 编译
make install     # 安装
```

**特点**：
- 使用 Kconfig 系统进行配置
- 支持 `make` 和 `make install`
- 生成的 `generated/*` 目录包含自动生成的文件

### OH 构建系统（GN）

**配置文件**：BUILD.gn, toybox.gni

**构建命令**：
```bash
hb build        # OH 构建命令
```

**特点**：
- 使用 GN（Generate Ninja）构建系统
- 通过 BUILD.gn 定义构建规则
- 支持 standard 和 small 系统类型
- 集成 OH 部署和安装流程

### 主要差异

| 方面 | 上游 | OH |
|-----|------|----|
| **构建系统** | Make + Kconfig | GN + Ninja |
| **配置方式** | make menuconfig | BUILD.gn + args |
| **符号链接** | make install 自动创建 | OH 构建系统自动创建 |
| **安全加固** | 无 | PIE/RELRO/NX |
| **平台支持** | Linux | Linux + LiteOS_A |
| **条件编译** | 无 | TOYBOX_OH_ADAPT 宏 |
| **功能开关** | Kconfig | GN args |

---

## 3.8 构建配置示例

### 标准系统完整配置

```gn
import("//build/ohos.gni")
import("toybox.gni")

# su 命令（调试版）
ohos_executable("su") {
  sources = [ "openharmony/su.c" ]
  include_dirs = [ "./openharmony" ]
  cflags_c = [ ... ]
  ldflags = [ ... ]
  part_name = "toybox"
  subsystem_name = "thirdparty"
  install_images = [ "eng_system" ]
  install_enable = true
}

# toybox 主可执行文件
ohos_executable("toybox") {
  sources = [
    # 190+ 个源文件
    "lib/args.c",
    "lib/commas.c",
    # ...
    "toys/posix/ls.c",
    "toys/posix/cp.c",
    # ...
  ]
  include_dirs = [ "./" ]
  cflags_c = [ ... ]
  ldflags = [ ... ]
  defines = [
    "_DEFAULT_SOURCE",
    "TOYBOX_OH_ADAPT",
  ]

  # SELinux 支持
  if (build_selinux) {
    cflags_c += [ "-DWITH_SELINUX", ... ]
    sources += [ "toys/other/chcon.c" ]
    external_deps = [
      "openssl:libcrypto_shared",
      "openssl:libssl_shared",
      "selinux:libselinux",
    ]
    symlink_target_name += [ "chcon" ]
  }

  # 扩展命令支持
  if (toybox_extended_cmd) {
    defines += [ "TOYBOX_EXTENDED_CMD" ]
    sources += [ ... ]
    symlink_target_name += [ "wget", "awk", "diff", ... ]
  }

  # 符号链接列表（200+ 个命令）
  symlink_target_name = [
    "acpi", "ascii", "base64", "basename", ...
  ]

  part_name = "toybox"
  subsystem_name = "thirdparty"
  install_images = [ "system", "ramdisk", "updater" ]
  install_enable = true
}
```

### LiteOS_A 构建配置

```gn
if (defined(ohos_lite)) {
  executable("toybox") {
    sources = [
      # 190+ 个源文件（与标准系统类似）
    ]
    include_dirs = [ "./" ]
    defines = [
      "_DEFAULT_SOURCE",
      "OHOS_LITE",
      "TOYBOX_OH_ADAPT",
    ]
    cflags_c = [ ... ]
    ldflags = [ ... ]
  }

  # 创建符号链接
  cmd_long_path = [ "bin/ls", "bin/cat", ... ]
  foreach(path, cmd_long_path) {
    exec_script("install.py", ...)
  }
}
```

---

## 3.9 构建参数（toybox.gni）

**文件路径**：`third_party/toybox/toybox.gni`

```gn
TOYBOX_SRC_DIR = [ "//third_party/toybox" ]

declare_args() {
  toybox_extended_cmd = false          # 扩展命令支持
  toybox_enable_brctl = false          # brctl 网桥管理
  toybox_feature_support_usr_symlink = false  # /usr/bin 符号链接
}
```

**使用方式**：

在产品配置文件中设置：
```json
{
  "toybox": {
    "toybox_extended_cmd": true,
    "toybox_enable_brctl": true,
    "toybox_feature_support_usr_symlink": true
  }
}
```

---

## 参考文档

- [01_Overview.md](./01_Overview.md) - Toybox 库简介
- [02_Patches.md](./02_Patches.md) - OH Patch 详细分析
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估结果
