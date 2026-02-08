# 编译产物

## 目的

本文档描述 c_utils 的编译产物清单、输出路径和运行时加载关系。

## 适用范围

- 系统集成工程师
- 部署和发布工程师
- 运行时问题排查

---

## 产物清单

### 动态库

| 产物名 | 类型 | 说明 | 安装位置 |
|--------|------|------|----------|
| `libutils.so` | 共享库 | C++ 公共基础库 | `/system/lib/` 或 `/system/lib64/` |
| `libutils_rust.dylib.so` | Rust共享库 | Rust FFI 接口 | `/system/lib/` 或 `/system/lib64/` |

### 静态库

| 产物名 | 类型 | 说明 | 使用场景 |
|--------|------|------|----------|
| `libutilsbase.a` | 静态库 | C++ 基础库（静态链接）| 特定模块静态链接 |
| `libutilsbase_rtti.a` | 静态库 | 带RTTI的版本 | 多媒体引擎 |
| `libutils_static_cxx_rust.a` | 静态库 | Rust/C++桥接 | Rust FFI |

### 头文件

| 路径 | 说明 |
|------|------|
| `base/include/*.h` | 32个对外头文件 |

---

## 输出路径

### 标准构建输出

```
out/{product}/
├── commonlibrary/c_utils/base/
│   ├── libutils.so              # 动态库
│   ├── libutilsbase.a           # 静态库
│   ├── libutilsbase_rtti.a      # 带RTTI静态库
│   └── gen/                     # 生成的代码
│       └── cxx_rust_gen/        # Rust绑定生成代码
│
├── system/lib/
│   └── libutils.so              # 安装后的动态库（32位）
│
├── system/lib64/
│   └── libutils.so              # 安装后的动态库（64位）
│
└── obj/commonlibrary/c_utils/   # 中间对象文件
```

### Rust 特定输出

```
out/{product}/
├── commonlibrary/c_utils/base/
│   ├── libutils_rust.dylib.so   # Rust动态库
│   └── rust/
│       └── libutils_rust.a      # Rust静态库（中间产物）
│
└── gen/commonlibrary/c_utils/   # Rust生成代码
    └── cxx_rust_gen/
        └── rust/cxx.h           # C++头文件生成
        └── rust/cxx.cc          # C++源文件生成
```

---

## 运行时加载关系

### 系统启动时

```
┌─────────────────────────────────────────────────────────────┐
│                      系统启动流程                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 内核启动                                                 │
│       │                                                     │
│       ▼                                                     │
│  2. init 进程                                                │
│       │                                                     │
│       ▼                                                     │
│  3. 加载系统分区                                             │
│       │                                                     │
│       ├──► /system/lib/libutils.so                          │
│       │       │                                             │
│       │       ├──► 依赖: libsec_shared.so                   │
│       │       └──► 依赖: libhilog.so (非Android/iOS)        │
│       │                                                     │
│       └──► /system/lib64/libutils.so (64位系统)              │
│               │                                             │
│               ├──► 依赖: libsec_shared.so                   │
│               └──► 依赖: libhilog.so                        │
│                                                             │
│  4. 系统服务启动                                             │
│       │                                                     │
│       ├──► foundation/ability/ability_runtime               │
│       │       └──► 加载 libutils.so                         │
│       │                                                     │
│       ├──► foundation/communication/ipc                     │
│       │       └──► 加载 libutils.so (Parcel)                │
│       │                                                     │
│       ├──► foundation/multimedia/media_foundation           │
│       │       └──► 可能静态链接 libutilsbase.a              │
│       │                                                     │
│       └──► ... 其他服务                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 进程加载时

```
进程启动
    │
    ├──► 加载器 (ld.so)
    │       │
    │       ├──► 解析 ELF 依赖
    │       │       ├──► libutils.so ──► 从 /system/lib 加载
    │       │       ├──► libsec_shared.so ──► 安全C库
    │       │       └──► libhilog.so ──► 日志库
    │       │
    │       └──► 符号解析与重定位
    │
    └──► 进程主代码执行
```

---

## 依赖关系

### 动态依赖

```bash
# 查看 libutils.so 依赖
readelf -d out/rk3568/commonlibrary/c_utils/base/libutils.so

# 典型输出:
Dynamic section at offset 0x12345 contains 25 entries:
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libsec_shared.so]
 0x00000001 (NEEDED)                     Shared library: [libhilog.so]
 0x00000001 (NEEDED)                     Shared library: [libc++.so]
 0x00000001 (NEEDED)                     Shared library: [libc.so]
```

### 符号导出

```bash
# 查看导出的符号
readelf -s out/rk3568/commonlibrary/c_utils/base/libutils.so | grep GLOBAL

# 典型导出符号:
# OHOS::RefBase::RefBase()
# OHOS::RefBase::~RefBase()
# OHOS::Parcel::Parcel()
# OHOS::Parcel::~Parcel()
# OHOS::ThreadPool::Start(int)
# ...
```

---

## 安装配置

### 安装镜像

`base/BUILD.gn:253-256`:
```gn
install_images = [
  "system",      # 安装到系统分区
  "updater",     # 安装到升级分区
]
```

### 安装路径规则

| 架构 | 安装路径 |
|------|----------|
| arm | `/system/lib/` |
| arm64 | `/system/lib64/` |
| x86_64 | `/system/lib64/` |

---

## 版本与兼容性

### 当前版本

`bundle.json:3`:
```json
{
  "name": "@ohos/c_utils",
  "version": "3.1.0"
}
```

### 兼容性策略

- **ABI 兼容**: 动态库保持向后兼容
- **API 兼容**: 头文件接口稳定
- **版本号**: 遵循语义化版本（SemVer）

---

## 故障排查

### 1. 找不到 libutils.so

**症状**:
```
CANNOT LINK EXECUTABLE: library "libutils.so" not found
```

**排查**:
```bash
# 检查文件是否存在
ls -la /system/lib/libutils.so
ls -la /system/lib64/libutils.so

# 检查依赖
ldd /system/bin/my_service | grep utils

# 检查 linker 搜索路径
cat /system/etc/ld.config.txt | grep search
```

### 2. 符号未找到

**症状**:
```
undefined symbol: _ZN4OHOS7RefBaseC1Ev
```

**排查**:
```bash
# 检查符号是否存在
readelf -s /system/lib/libutils.so | grep RefBase

# 检查版本匹配
strings /system/lib/libutils.so | grep version
```

### 3. 静态链接冲突

**症状**:
```
multiple definition of `OHOS::RefBase::RefBase()'
```

**原因**: 同时链接了 libutils.so 和 libutilsbase.a

**解决**: 统一使用动态库或静态库，不要混用

---

## 相关跳转

- [GN Targets](06_GN_Targets.md) - 构建配置
- [对外 API](04_Public_API.md) - 接口使用
- [问题排查](09_Troubleshooting.md) - 常见问题
