# OH 构建适配

## BUILD.gn 结构说明

OpenHarmony 使用 GN (Generate Ninja) 作为构建系统，本库通过 `BUILD.gn` 文件集成到 OH 构建体系中。

### 文件结构

```gn
BUILD.gn
├── 变量定义（根目录）
├── config 定义
├── action 定义（MD5 检查）
├── ohos_shared_library("libsepol")
├── ohos_executable("chkcon")
├── selinux_sources（libselinux 源文件列表）
├── ohos_shared_library("libselinux")
├── ohos_static_library("libselinux_static")
└── 各种可执行工具定义
```

---

## 关键编译选项

### libsepol 编译选项

```gn
ohos_shared_library("libsepol") {
    cflags = [
        "-D_GNU_SOURCE",         # 使用 GNU 扩展
        "-DHAVE_REALLOCARRAY",   # 支持 reallocarray 函数
        "-w",                     # 禁用所有警告
    ]
}
```

### libselinux 编译选项

```gn
cflags = [
    "-DOHOS_FC_INIT",               # 【OH 特有】启用多文件 contexts 支持
    "-D_GNU_SOURCE",
    "-w",
    "-DSHARED",                      # 编译为共享库
    "-DUSE_PCRE2",                  # 使用 PCRE2 替代 PCRE
    "-U__BIONIC__",                 # 【OH 特有】取消 Bionic libc 特定代码
    "-DAUDITD_LOG_TAG=1003",        # 【OH 特有】OH 特定的 audit log tag
    "-DPCRE2_CODE_UNIT_WIDTH=8",   # PCRE2 8 位模式
    "-DHAVE_REALLOCARRAY",
]

# 架构特定配置
if (host_cpu == "arm64" && host_os == "linux") {
    cflags += [ "-DWITH_FREEBSD" ]  # ARM64 Linux 使用 FreeBSD 兼容层
}
```

### 选项详解

| 选项 | 说明 | OH 特定 |
|-----|------|--------|
| `-DOHOS_FC_INIT` | 启用多文件 file_contexts 支持 | ✅ 是 |
| `-DUSE_PCRE2` | 使用 PCRE2 正则表达式库 | ❌ 否（上游也支持） |
| `-U__BIONIC__` | 取消 Bionic libc 宏定义 | ✅ 是（避免 Android 特定代码） |
| `-DAUDITD_LOG_TAG=1003` | 设置 audit 日志 tag | ✅ 是 |
| `-DWITH_FREEBSD` | 启用 FreeBSD 兼容代码 | ✅ 是（ARM64） |

---

## 特殊配置说明

### 1. no-LTO 配置

```gn
config("third_party_selinux_nolto_config") {
    if (use_libfuzzer && !is_mac) {
        cflags = []
    } else {
        cflags = [
            "-fno-emulated-tls",          # 禁用模拟 TLS
            "-fno-lto",                    # 禁用链接时优化
            "-fno-whole-program-vtables", # 禁用全程序虚表优化
        ]
    }
}
```

**原因**：SELinux 库使用了一些动态特性，与 LTO 优化可能产生冲突。

### 2. include 路径配置

```gn
config("third_party_selinux_config") {
    include_dirs = [
        "$LIBSELINUX_ROOT_DIR/include",
        "$LIBSELINUX_ROOT_DIR",
    ]
}
```

### 3. PAC-RET 保护（ARM64）

```gn
ohos_shared_library("libselinux") {
    branch_protector_ret = "pac_ret"   # 返回地址保护
    // ...
}
```

### 4. 符号版本控制

```gn
ohos_shared_library("libsepol") {
    version_script = "libsepol.map"    # 控制导出符号版本
}
```

`libsepol.map` 文件内容示例：
```
LIBSEPOL_1.0 {
    global:
        sepol_*;
    local:
        *;
};
```

---

## 依赖配置

### 外部依赖

```gn
external_deps = [
    "FreeBSD:libfreebsd_static",    # FreeBSD 兼容层（ARM64）
    "pcre2:libpcre2",               # PCRE2 正则表达式库
]
```

### 头文件导出

```gn
inner_kits = [
    {
        "name": "//third_party/selinux:libselinux",
        "header": {
            "header_files": [],
            "header_base": "//third_party/selinux/libselinux/include/selinux"
        }
    },
    {
        "name": "//third_party/selinux:libsepol",
        "header": {
            "header_files": [],
            "header_base": "//third_party/selinux/libsepol/include/sepol"
        }
    },
]
```

---

## 安装配置

### 安装镜像

```gn
install_images = [
    "system",       # 系统分区
    "ramdisk",      # 内存文件系统
    "updater",      # 升级程序
]
```

**说明**：
- `system`：标准系统运行时需要
- `ramdisk`：启动早期阶段需要
- `updater`：系统更新时需要

### 安装开关

```gn
install_enable = true
```

---

## MD5 检查 Action

### 背景

SELinux 使用 flex/bison 生成 lexer 和 parser。为避免重复生成，使用 MD5 检查源文件是否变更。

### 实现

```gn
action("check_md5_libsepol") {
    script = "//third_party/selinux/exec_check_md5.sh"
    inputs = [
        "libsepol/cil/src/cil_lexer.l",    # flex 源文件
    ]
    args = [
        rebase_path("//third_party/selinux", root_build_dir),
        "libsepol",
        rebase_path("$target_gen_dir", root_build_dir),
    ]
    outputs = [ "$target_gen_dir/libsepol/cil/src/cil_lexer.c" ]
}

# libsepol 依赖此 action
ohos_shared_library("libsepol") {
    sources += get_target_outputs(":check_md5_libsepol")
    deps = [ ":check_md5_libsepol" ]
}
```

### 脚本说明

`exec_check_md5.sh` 脚本逻辑：
1. 计算输入文件（`.l`、`.y`）的 MD5
2. 与上次生成的 `.md5` 文件对比
3. 如果一致，复用上次的生成结果
4. 如果不一致，重新生成并更新 MD5

---

## 与上游构建系统的差异

| 特性 | 上游 (Makefile) | OH (BUILD.gn) |
|-----|-----------------|---------------|
| 构建系统 | Makefile | GN + Ninja |
| 编译选项 | 在 Makefile 中定义 | 在 BUILD.gn 中定义 |
| 依赖管理 | pkg-config | external_deps |
| 安装路径 | `make install` | install_images |
| 目标平台 | 通用 Linux | OH 特定 |
| PCRE 版本 | PCRE 或 PCRE2 | 仅 PCRE2 |
| 多文件 contexts | 不支持 | 通过 OHOS_FC_INIT 支持 |

### 上游 Makefile 关键选项对比

上游 Makefile 中的对应选项：

```makefile
# 上游 Makefile
PCRE2 ?= y                    # 使用 PCRE2
ANDROID_HOST ?= n             # 非 Android
FLAGS += -D_GNU_SOURCE
FLAGS += -DHAVE_REALLOCARRAY
```

OH BUILD.gn 中的对应：

```gn
# OH BUILD.gn
cflags = [
    "-DUSE_PCRE2",           # 对应 PCRE2=y
    "-U__BIONIC__",          # 对应 ANDROID_HOST=n（反向）
    "-D_GNU_SOURCE",
    "-DHAVE_REALLOCARRAY",
    "-DOHOS_FC_INIT",        # OH 特有
]
```

---

## 构建目标详解

### 动态库

| 目标 | 输出 | 说明 |
|-----|------|-----|
| libsepol | libsepol.so | 策略编译库 |
| libselinux | libselinux.so | SELinux 核心库 |

### 静态库

| 目标 | 输出 | 说明 |
|-----|------|-----|
| libselinux_static | libselinux_static.a | 静态链接版本（updater 使用） |

### 可执行工具

| 目标 | 源文件 | 功能 |
|-----|-------|-----|
| checkpolicy | checkpolicy/*.c | 编译二进制策略 |
| secilc | secilc/secilc.c | 编译 CIL 策略 |
| sefcontext_compile | utils/sefcontext_compile.c | 编译 file_contexts |
| setenforce | utils/setenforce.c | 设置 SELinux 状态 |
| getenforce | utils/getenforce.c | 获取 SELinux 状态 |
| setfilecon | utils/setfilecon.c | 设置文件上下文 |
| getfilecon | utils/getfilecon.c | 获取文件上下文 |
| getpidcon | utils/getpidcon.c | 获取进程上下文 |
| selinux_check_access | utils/selinux_check_access.c | 检查访问权限 |
| selinuxexeccon | utils/selinuxexeccon.c | 查询执行上下文 |
| chkcon | libsepol/utils/chkcon.c | 检查策略一致性 |

---

## 升级 BUILD.gn 指南

当升级上游 SELinux 版本时，可能需要更新 BUILD.gn：

### 检查清单

1. **源文件变更**
   ```gn
   # 检查上游是否有新增/删除的源文件
   sources = [
       # 对比上游 Makefile 的 SOURCES 变量
   ]
   ```

2. **头文件路径**
   ```gn
   # 检查 include_dirs 是否仍正确
   include_dirs = [
       # 对比上游目录结构
   ]
   ```

3. **编译选项**
   ```gn
   # 检查上游 Makefile 的 FLAGS 变更
   cflags = [
       # 同步上游新增的标志
   ]
   ```

4. **版本脚本**
   ```gn
   # 检查 libsepol.map 是否需要更新
   version_script = "libsepol.map"
   ```

### 自动化检查脚本

```bash
# 1. 对比源文件列表
diff <(cat BUILD.gn | grep '\.c"' | sort) <(cat upstream/Makefile | grep 'SOURCES' | sort)

# 2. 检查新增文件
find libselinux/src -name '*.c' -newer BUILD.gn
find libsepol/src -name '*.c' -newer BUILD.gn

# 3. 验证 BUILD.gn 语法
gn gen out
```

