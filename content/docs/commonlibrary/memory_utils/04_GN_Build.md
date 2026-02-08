# GN 构建配置

本文档描述 memory_utils 各模块的 GN 构建配置，包括 targets、依赖关系和编译产物。

---

## 构建入口

### 根构建配置

| 文件 | 用途 |
|------|------|
| `bundle.json` | 部件配置文件，定义子组件和 inner_kits |
| `purgeable_mem_config.gni` | purgeable memory 配置导入文件 |

**证据来源**: `bundle.json:31-36`

```json
"build": {
  "sub_component": [
    "//commonlibrary/memory_utils/libdmabufheap:libdmabufheap",
    "//commonlibrary/memory_utils/libmeminfo:libmeminfo",
    "//commonlibrary/memory_utils/libpurgeablemem:libpurgeablemem",
    "//commonlibrary/memory_utils/libpurgeablemem:purgeable_memory_ndk"
  ]
}
```

---

## libdmabufheap 构建配置

### BUILD.gn 路径

```
libdmabufheap/BUILD.gn
```

### Target 定义

```gn
ohos_shared_library("libdmabufheap") {
  sources = [ "src/dmabuf_alloc.c" ]
  
  include_dirs = [ "include" ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
  ]
  
  public_configs = [ ":libdmabufheap_config" ]
  
  sanitize = {
    cfi = true
    cfi_cross_dso = true
  }
  
  branch_protector = "pac_ret"
  
  subsystem_name = "commonlibrary"
  part_name = "memory_utils"
}
```

### 配置详情

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **Target 类型** | ohos_shared_library | 共享库 |
| **Sources** | src/dmabuf_alloc.c | 1 个 C 源文件 |
| **Include Dirs** | include | 头文件目录 |
| **External Deps** | c_utils:utils, hilog:libhilog | 外部依赖 |
| **Public Configs** | :libdmabufheap_config | 公共配置 |
| **Sanitize** | cfi=true, cfi_cross_dso=true | CFI 防护 |
| **Branch Protector** | pac_ret | PAC 指针认证 |

### 内部配置

```gn
config("libdmabufheap_config") {
  include_dirs = [ "include" ]
  defines = []
  configs = []
}
```

### 产物映射

| Source | Target | 输出产物 |
|--------|--------|---------|
| dmabuf_alloc.c | libdmabufheap | libdmabufheap.so |

**安装路径**: `system/lib64/` (默认共享库路径)

---

## libmeminfo 构建配置

### BUILD.gn 路径

```
libmeminfo/BUILD.gn
```

### Target 定义

```gn
ohos_shared_library("libmeminfo") {
  sources = [ "src/meminfo.cpp" ]
  
  include_dirs = [ "include" ]
  
  external_deps = [
    "c_utils:utils",
    "drivers_interface_memorytracker:libmemorytracker_proxy_1.0",
    "hilog:libhilog",
  ]
  
  public_configs = [ ":libmeminfo_config" ]
  
  sanitize = {
    cfi = true
    cfi_cross_dso = true
  }
  
  branch_protector = "pac_ret"
  
  subsystem_name = "commonlibrary"
  part_name = "memory_utils"
}
```

### 配置详情

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **Target 类型** | ohos_shared_library | 共享库 |
| **Sources** | src/meminfo.cpp | 1 个 C++ 源文件 |
| **Include Dirs** | include | 头文件目录 |
| **External Deps** | c_utils, drivers_interface_memorytracker, hilog | 外部依赖 |
| **Public Configs** | :libmeminfo_config | 公共配置 |
| **Sanitize** | cfi=true, cfi_cross_dso=true | CFI 防护 |
| **Branch Protector** | pac_ret | PAC 指针认证 |

### 产物映射

| Source | Target | 输出产物 |
|--------|--------|---------|
| meminfo.cpp | libmeminfo | libmeminfo.so |

**安装路径**: `system/lib64/`

---

## libpurgeablemem 构建配置

### BUILD.gn 路径

```
libpurgeablemem/BUILD.gn
```

### Target 定义

```gn
ohos_shared_library("libpurgeablemem") {
  sources = [
    "c/src/purgeable_mem_builder_c.c",
    "c/src/purgeable_mem_c.c",
    "c/src/purgeable_memory.c",
    "common/src/pm_state_c.c",
    "common/src/ux_page_table_c.c",
    "cpp/src/purgeable_ashmem.cpp",
    "cpp/src/purgeable_mem.cpp",
    "cpp/src/purgeable_mem_base.cpp",
    "cpp/src/purgeable_mem_builder.cpp",
    "cpp/src/ux_page_table.cpp",
  ]
  
  include_dirs = [ "include" ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "hitrace:hitrace_meter",
    "init:libbegetutil",
    "ipc:ipc_core",
  ]
  
  public_configs = [ ":libpurgeable_config" ]
  
  cflags_cc = [ "-fexceptions" ]
  
  sanitize = {
    cfi = true
    cfi_cross_dso = true
  }
  
  branch_protector = "pac_ret"
  
  subsystem_name = "commonlibrary"
  part_name = "memory_utils"
}
```

### 配置详情

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **Target 类型** | ohos_shared_library | 共享库 |
| **Sources** | 10 个文件 | C 和 C++ 混合 |
| **Include Dirs** | include | 头文件目录 |
| **External Deps** | c_utils, hilog, hitrace, init, ipc | 5 个外部依赖 |
| **Public Configs** | :libpurgeable_config | 公共配置 |
| **CFlags CC** | -fexceptions | 启用 C++ 异常 |
| **Sanitize** | cfi=true, cfi_cross_dso=true | CFI 防护 |

### Source 文件清单

| 文件 | 语言 | 说明 |
|------|------|------|
| c/src/purgeable_mem_builder_c.c | C | C 构建器实现 |
| c/src/purgeable_mem_c.c | C | C 接口实现 |
| c/src/purgeable_memory.c | C | NDK 接口实现 |
| common/src/pm_state_c.c | C | 状态码实现 |
| common/src/ux_page_table_c.c | C | C 页表操作 |
| cpp/src/purgeable_ashmem.cpp | C++ | Ashmem 方案 |
| cpp/src/purgeable_mem.cpp | C++ | 主类实现 |
| cpp/src/purgeable_mem_base.cpp | C++ | 基类实现 |
| cpp/src/purgeable_mem_builder.cpp | C++ | 构建器实现 |
| cpp/src/ux_page_table.cpp | C++ | 页表封装 |

### 内部配置

```gn
config("libpurgeable_config") {
  include_dirs = [
    "c/include",
    "common/include",
    "cpp/include",
    "interfaces/kits/c",
  ]
  defines = []
  configs = []
}
```

### 产物映射

| Source | Target | 输出产物 |
|--------|--------|---------|
| 所有源文件 | libpurgeablemem | libpurgeablemem.z.so |

**安装路径**: `system/lib64/`

---

## purgeable_memory_ndk 构建配置

### BUILD.gn 路径

```
libpurgeablemem/BUILD.gn
```

### Target 定义

```gn
ohos_shared_library("purgeable_memory_ndk") {
  sources = [ "c/src/purgeable_memory.c" ]
  
  include_dirs = [ "interfaces/kits/c" ]
  
  deps = [ ":libpurgeablemem" ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
  ]
  
  relative_install_dir = "ndk"
  
  sanitize = {
    cfi = true
    cfi_cross_dso = true
  }
  
  branch_protector = "pac_ret"
  
  subsystem_name = "commonlibrary"
  part_name = "memory_utils"
}
```

### 配置详情

| 配置项 | 值 | 说明 |
|--------|-----|------|
| **Target 类型** | ohos_shared_library | 共享库 |
| **Sources** | c/src/purgeable_memory.c | NDK 接口实现 |
| **Include Dirs** | interfaces/kits/c | NDK 头文件目录 |
| **Deps** | :libpurgeablemem | 内部依赖（链接 libpurgeablemem） |
| **External Deps** | c_utils, hilog | 外部依赖 |
| **Install Dir** | ndk | NDK 专用目录 |
| **Sanitize** | cfi=true, cfi_cross_dso=true | CFI 防护 |

### 产物映射

| Source | Target | 输出产物 | 安装路径 |
|--------|--------|---------|---------|
| purgeable_memory.c | purgeable_memory_ndk | purgeable_memory_ndk.z.so | system/lib64/ndk/ |

---

## 测试构建配置

### libdmabufheap 测试

**路径**: `libdmabufheap/test/BUILD.gn`

```gn
ohos_unittest("unittest") {
  sources = [ "dmabuf_alloc_test.cpp" ]
  external_deps = [ "dmabufheap:libdmabufheap" ]
}
```

### libmeminfo 测试

**路径**: `libmeminfo/test/BUILD.gn`

```gn
ohos_unittest("libmeminfo_test") {
  sources = [ "meminfo_test.cpp" ]
  external_deps = [ "meminfo:libmeminfo" ]
}
```

### libpurgeablemem 测试

**路径**: `libpurgeablemem/test/BUILD.gn`

```gn
ohos_unittest("libpurgeablemem_test") {
  sources = [
    "purgeable_c_test.cpp",
    "purgeable_cpp_test.cpp",
    "purgeable_memory_test.cpp",
    "purgeableashmem_test.cpp",
  ]
  external_deps = [ "purgeablemem:libpurgeablemem" ]
}
```

---

## 依赖关系图

```
                              ┌─────────────────┐
                              │   libc / libm    │
                              └────────┬────────┘
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        │                              │                              │
        ▼                              ▼                              ▼
┌─────────────────┐           ┌─────────────────┐           ┌─────────────────┐
│ libdmabufheap  │           │  libmeminfo    │           │ libpurgeablemem │
│                 │           │                 │           │                 │
├─────────────────┤           ├─────────────────┤           ├─────────────────┤
│ deps:           │           │ deps:           │           │ deps:           │
│ - c_utils       │           │ - c_utils       │           │ - c_utils       │
│ - hilog         │           │ - hilog         │           │ - hilog         │
│                 │           │ - memorytracker │           │ - hitrace       │
│                 │           │                 │           │ - init          │
│                 │           │                 │           │ - ipc           │
└────────┬────────┘           └────────┬────────┘           └────────┬────────┘
         │                             │                             │
         │                             │                             │
         ▼                             │                             ▼
┌─────────────────┐                   │                     ┌─────────────────┐
│ libdmabufheap.z │                   │                     │ purgeable_      │
│ .so              │                   │                     │ memory_ndk.z.so │
└─────────────────┘                   │                     └─────────────────┘
                                       │                             │
                                       │                             │
                                       ▼                             │
                               ┌─────────────────┐                  │
                               │  libmeminfo.z   │                  │
                               │  .so             │                  │
                               └─────────────────┘                  │
```

---

## 产物清单

| 产物 | 类型 | 大小 (约) | 用途 |
|------|------|----------|------|
| libdmabufheap.z.so | 共享库 | ~20KB | DMA 缓冲区分配 |
| libmeminfo.z.so | 共享库 | ~30KB | 内存信息查询 |
| libpurgeablemem.z.so | 共享库 | ~50KB | 可回收内存管理 |
| purgeable_memory_ndk.z.so | 共享库 (NDK) | ~10KB | NDK 接口导出 |

**ROM 占用**: ~120KB (bundle.json:17)
**RAM 占用**: ~200KB (bundle.json:18)

---

## 编译开关

### bundle.json 中的特性

```json
"features": [
  "memory_utils_purgeable_ashmem_enable"
]
```

**说明**: 启用 Ashmem 方案作为可回收内存的备选实现。

### 编译选项

| 选项 | 值 | 说明 |
|------|-----|------|
| **cfi** | true | 启用 Control Flow Integrity |
| **cfi_cross_dso** | true | 跨 DSO CFI 防护 |
| **branch_protector** | pac_ret | PAC 指针认证 |
| **cflags_cc** | -fexceptions | 启用 C++ 异常 |

---

## 构建命令

### 完整构建

```bash
# 构建整个 memory_utils 部件
hb build -p memory_utils
```

### 单模块构建

```bash
# 构建 libdmabufheap
hb build //commonlibrary/memory_utils/libdmabufheap:libdmabufheap

# 构建 libmeminfo
hb build //commonlibrary/memory_utils/libmeminfo:libmeminfo

# 构建 libpurgeablemem
hb build //commonlibrary/memory_utils/libpurgeablemem:libpurgeablemem

# 构建 NDK
hb build //commonlibrary/memory_utils/libpurgeablemem:purgeable_memory_ndk
```

### 运行测试

```bash
# 运行所有测试
hb test //commonlibrary/memory_utils/...:unittest

# 运行单个测试
hb test //commonlibrary/memory_utils/libdmabufheap/test:unittest
hb test //commonlibrary/memory_utils/libmeminfo/test:libmeminfo_test
hb test //commonlibrary/memory_utils/libpurgeablemem/test:libpurgeablemem_test
```

---

## 运行时加载关系

```
应用进程
   │
   ├──► dlopen("libpurgeablemem.z.so") ──────► 系统加载
   │           │
   │           └──► 依赖: libutils.z.so, libhilog.z.so...
   │
   ├──► dlopen("libmeminfo.z.so") ──────────► 系统加载
   │           │
   │           └──► 依赖: libhilog.z.so, libmemorytracker_proxy_1.0.so...
   │
   └──► dlopen("libdmabufheap.z.so") ───────► 系统加载
               │
               └──► 依赖: libhilog.z.so
```

---

**最后更新**: 2026-02-06
