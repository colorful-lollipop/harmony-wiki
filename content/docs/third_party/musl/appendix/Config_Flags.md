# 配置宏和特性开关

> OpenHarmony musl 配置宏完整列表

---

## 构建特性宏

### 基础定义

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `__MUSL__` | musl_template.gni:39 | musl 标识 |
| `_LIBCPP_HAS_MUSL_LIBC` | musl_template.gni:40 | libc++ 适配 |
| `__BUILD_LINUX_WITH_CLANG` | musl_template.gni:41 | Clang 构建标识 |
| `CRT` | musl_template.gni:661 | C 运行时标识 |

### 架构定义

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `MUSL_ARM_ARCH` | musl_template.gni:362 | ARM 架构 |
| `MUSL_AARCH64_ARCH` | musl_template.gni:365 | AArch64 架构 |
| `MUSL_X86_64_ARCH` | musl_template.gni:368 | x86_64 架构 |

## 安全相关宏

### 内存安全

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `MALLOC_FREELIST_HARDENED` | musl_template.gni:371 | 空闲列表加固 |
| `MALLOC_FREELIST_QUARANTINE` | musl_template.gni:374 | 隔离区机制 |
| `MALLOC_RED_ZONE` | musl_template.gni:377 | 红区保护 |
| `MALLOC_SECURE_ALL` | musl_template.gni:380 | 全部安全特性 |

### 堆保护

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `USE_GWP_ASAN` | musl_template.gni:14 | GWP-ASan 启用 |
| `HWASAN_REMOVE_CLEANUP` | musl_template.gni:435 | HWASan 清理 |

## Hook 相关宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `HOOK_ENABLE` | musl_template.gni:46 | 启用 Hook 机制 |
| `OHOS_SOCKET_HOOK_ENABLE` | musl_template.gni:47 | Socket Hook |
| `OHOS_FDTRACK_HOOK_ENABLE` | musl_template.gni:56 | FD 追踪 Hook |
| `OHOS_ENABLE_PARAMETER` | musl_template.gni:119 | 参数系统支持 |

## 线程相关宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `CXA_THREAD_USE_TSD` | musl_template.gni:359 | 使用 TSD 实现 |
| `PTHREAD_CANCEL_IN_STATIC_LIB` | musl_template.gni:424 | 静态库支持 cancel |
| `FEATURE_PTHREAD_CANCEL` | musl_template.gni:420 | pthread cancel 支持 |
| `USE_MUTEX_WAIT_OPT` | musl_template.gni:414 | 互斥锁优化 |

## 全球化宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `FEATURE_ICU_LOCALE` | musl_template.gni:351 | ICU 全球化 |
| `FEATURE_ICU_LOCALE_TMP` | musl_template.gni:353 | ICU 临时支持 |

## 动态链接器宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `HANDLE_RANDOMIZATION` | musl_template.gni:516 | 句柄随机化 |
| `LOAD_ORDER_RANDOMIZATION` | musl_template.gni:517 | 加载顺序随机化 |
| `BTI_SUPPORT` | musl_template.gni:533 | BTI 支持 |
| `MUSL_EXTERNAL_FUNCTION` | musl_template.gni:521 | 扩展功能 |

## 调试宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `IS_ASAN` | musl_template.gni:510 | ASan 构建 |
| `IS_TSAN` | musl_template.gni:513 | TSan 构建 |
| `UNIT_TEST_STATIC` | musl_template.gni:123 | 单元测试 |

## 其他宏

| 宏 | 定义位置 | 说明 |
|----|----------|------|
| `USE_JEMALLOC` | musl_template.gni:403 | 使用 jemalloc |
| `USE_JEMALLOC_DFX_INTF` | musl_template.gni:406 | jemalloc DFX 接口 |
| `MUSL_ITERATE_AND_STATS_API` | musl_template.gni:383 | 迭代和统计 API |
| `COMPONENT_BUILD` | musl_template.gni:69 | 组件构建 |

## 安全级别配置

```
musl_secure_level=0: 无额外保护
musl_secure_level=1: + MALLOC_FREELIST_HARDENED
musl_secure_level=2: + MALLOC_FREELIST_QUARANTINE
musl_secure_level=3: + MALLOC_RED_ZONE + MALLOC_SECURE_ALL
```

## 特性开关配置

### bundle.json features

```json
[
  "musl_use_encaps",
  "musl_ld128_flag",
  "musl_iterate_and_stats_api",
  "musl_is_legacy",
  "musl_enable_musl_log",
  "musl_unit_test_flag",
  "musl_use_flto",
  "musl_use_gwp_asan",
  "musl_use_pthread_cancel",
  "musl_uapi_dir",
  "musl_use_jemalloc",
  "musl_use_jemalloc_dfx_intf",
  "musl_use_jemalloc_recycle_func",
  "musl_guard_jemalloc_tsd",
  "musl_malloc_plugin",
  "musl_linux_kernel_dir",
  "musl_use_mutex_wait_opt",
  "musl_use_adlt",
  "musl_adlt_llvm",
  "musl_extended_function"
]
```

