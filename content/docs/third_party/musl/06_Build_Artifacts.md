# 编译产物

> OpenHarmony musl 编译产物说明

---

## 目的与适用范围

**目的**: 说明 musl 构建后的输出文件、安装路径和运行时加载关系。

**适用范围**: 系统集成工程师、发布工程师。

---

## 产物概览

### 库文件

| 产物 | 类型 | 说明 |
|------|------|------|
| `libc.so` | 共享库 | 主 C 库（动态链接）|
| `libc.a` | 静态库 | 主 C 库（静态链接）|
| `libm.a` | 静态库 | 数学库 |
| `libpthread.a` | 静态库 | 线程库 |
| `libdl.a` | 静态库 | 动态链接接口库 |
| `librt.a` | 静态库 | 实时库 |
| `libcrypt.a` | 静态库 | 加密库 |
| `libresolv.a` | 静态库 | DNS 解析库 |
| `libutil.a` | 静态库 | 工具库 |
| `libxnet.a` | 静态库 | X/Open 网络库 |

### 动态链接器

| 产物 | 说明 |
|------|------|
| `ld-musl-{arch}.so.1` | musl 动态链接器 |

### 头文件

| 产物 | 说明 |
|------|------|
| `include/` | 标准头文件目录 |
| `bits/` | 架构特定头文件 |

### C 运行时

| 产物 | 说明 |
|------|------|
| `crt1.o` | 程序入口（静态链接）|
| `Scrt1.o` | 程序入口（动态链接）|
| `rcrt1.o` | 可重定位入口 |
| `crti.o` | 初始化代码 |
| `crtn.o` | 终止代码 |
| `crtplus.o` | 扩展入口 |

---

## 输出目录结构

```
out/{target}/
├── obj/third_party/musl/
│   └── usr/
│       ├── include/
│       │   └── {arch}-linux-ohos/    # 头文件
│       │       ├── stdio.h
│       │       ├── stdlib.h
│       │       ├── bits/              # 架构特定
│       │       ├── sys/
│       │       └── ...
│       └── lib/
│           └── {arch}-linux-ohos/     # 库文件
│               ├── libc.so            # 动态库
│               ├── libc.a             # 静态库
│               ├── libm.a
│               ├── libpthread.a
│               ├── libdl.a
│               ├── librt.a
│               ├── libcrypt.a
│               ├── libresolv.a
│               ├── libutil.a
│               ├── libxnet.a
│               ├── sp/
│               │   └── libc.so        # 强保护版本
│               └── ld-musl-{arch}.so.1 -> libc.so  # 动态链接器
```

---

## 安装路径

### 系统镜像

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `libc.so` | `/system/lib64/` | 64位系统库目录 |
| `libc.so` | `/system/lib/` | 32位系统库目录 |
| `ld-musl-*.so.1` | `/system/bin/` | 动态链接器 |

### 头文件安装

| 产物 | 安装路径 |
|------|----------|
| 标准头文件 | `{sysroot}/usr/include/` |
| 架构特定头文件 | `{sysroot}/usr/include/bits/` |

### 配置文件

| 产物 | 安装路径 | 说明 |
|------|----------|------|
| `ld-musl-namespace-*.ini` | `/etc/` | Namespace 配置 |
| `musl.para` | `/system/etc/param/` | 系统参数 |
| `musl_init.cfg` | `/system/etc/init/` | 初始化配置 |

---

## 运行时加载关系

### 程序启动流程

```
1. 内核加载程序
   │
   ├──► 读取 ELF 头
   │
   ├──► 查找 INTERP 段
   │    └──► /system/bin/ld-musl-aarch64.so.1
   │
   └──► 加载动态链接器
        │
        ├──► 解析程序依赖
        ├──► 加载 libc.so 等库
        ├──► 符号解析和重定位
        └──► 跳转到程序入口
```

### 库依赖关系

```
应用程序
    │
    ├──► libc.so
    │     ├──► 系统调用
    │     ├──► 内存管理 (mallocng)
    │     ├──► 线程管理
    │     └──► 文件 I/O
    │
    ├──► libm.so (可选)
    │     └──► 数学函数
    │
    └──► 其他库
```

### Namespace 加载

```
应用程序
    │
    ├──► default namespace
    │     ├──► /system/lib64/*.so
    │     └──► inherits ndk namespace
    │
    └──► ndk namespace
          └──► /system/lib64/ndk/*.so
```

---

## 产物特性

### 1. 动态库 (libc.so)

| 特性 | 说明 |
|------|------|
| 入口点 | `_dlstart` |
| 导出符号 | 由 `libc.map.txt` 控制 |
| 依赖 | libdl, libpthread (内部) |
| 安全特性 | 地址随机化、RELRO、PIE |

### 2. 静态库 (libc.a)

| 特性 | 说明 |
|------|------|
| 完整库 | `complete_static_lib = true` |
| 包含 | 所有目标文件 |
| 使用 | 静态链接时使用 |

### 3. 强保护版本 (sp/libc.so)

| 特性 | 说明 |
|------|------|
| 适用 | aarch64 架构 |
| 额外保护 | 更强的栈保护 |
| 编译选项 | `-fstack-protector-strong` |

---

## 符号导出控制

### libc.map.txt

控制动态库导出的符号：

```
libc {
  global:
    # 标准 C 函数
    malloc;
    free;
    printf;
    scanf;
    # ... 更多
    
  local:
    # 内部符号
    *;
};
```

只有 `global` 部分的符号对外可见，其他符号被隐藏。

---

## 构建产物验证

### 检查动态库

```bash
# 查看依赖
readelf -d out/target/usr/lib/aarch64-linux-ohos/libc.so

# 查看导出符号
readelf -s out/target/usr/lib/aarch64-linux-ohos/libc.so | grep GLOBAL

# 检查安全特性
readelf -l out/target/usr/lib/aarch64-linux-ohos/libc.so | grep -E "(GNU_RELRO|GNU_STACK)"
```

### 检查静态库

```bash
# 查看包含的目标文件
ar -t out/target/usr/lib/aarch64-linux-ohos/libc.a

# 查看符号
nm out/target/usr/lib/aarch64-linux-ohos/libc.a | grep malloc
```

---

## 相关跳转

- [GN 构建目标](05_GN_Targets.md) - 构建配置详解
- [架构设计](02_Architecture.md) - 组件关系
- [问题排查](08_Troubleshooting.md) - 构建问题定位

