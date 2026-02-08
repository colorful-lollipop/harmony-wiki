# module_sample - 示例模块

## 1. 概述

### 1.1 模块定位

module_sample 是 BUILD.gn 构建示例，用于演示如何构建内核模块（.ko 文件）。

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/module_sample/
├── ko_sample.c        # 示例模块 (30行)
├── sample_fun.c       # 示例函数 (21行)
└── BUILD.gn           # GN 构建配置 (23行)
```

---

## 3. 构建配置

### 3.1 BUILD.gn

**文件**: `BUILD.gn:1-23`

```gn
import("//build/templates/kernel/ohos_kernel_build.gni")

ohos_build_ko("ko_sample") {
  sources = [
    "ko_sample.c",
    "sample_fun.c",
  ]
  target_ko_name = "kosample"    # 输出: kosample.ko
  device_name = device_name       # 从构建继承
  device_arch = "arm64"
}
```

### 3.2 根目录 BUILD.gn

**文件**: `/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/BUILD.gn:14-16`

```gn
group("ko_build") {
  deps = [ "module_sample:ko_sample" ]
}
```

---

## 4. 示例代码

### 4.1 ko_sample.c

```c
static int kosample_init(void);    // 模块初始化
static void kosample_exit(void);   // 模块清理
```

### 4.2 sample_fun.c

```c
int kosample_fun(void);            // 导出的示例函数
```

---

## 5. 编译与构建

### 5.1 编译命令

```bash
./build.sh --product-name rk3568 --build-target mk_chip_ckm_img --ccache --jobs 4
```

### 5.2 产物位置

| 产物 | 位置 |
|------|------|
| .ko 文件 | `out/{device}/packages/phone/chip_ckm/*.ko` |
| 镜像文件 | `out/{device}/packages/phone/images/chip_ckm.img` |

**证据来源**: `README.md:114-133`

---

## 6. 使用指南

### 6.1 新增模块步骤

1. 在 `common_modules/` 下创建模块目录
2. 编写源码文件和 `BUILD.gn`
3. 在根目录 `BUILD.gn` 的 `deps` 中添加模块
4. 运行构建命令

### 6.2 BUILD.gn 模板

```gn
import("//build/templates/kernel/ohos_kernel_build.gni")

ohos_build_ko("your_module_name") {
  sources = [
    "your_source.c",
  ]
  target_ko_name = "your_ko_name"
  device_name = device_name
  device_arch = "arm64"
}
```

---

## 7. 相关文档

| 文档 | 说明 |
|------|------|
| [03_Build_System.md](../03_Build_System.md) | 构建系统详解 |
| [README.md](../README.md#ko模块指导) | ko 模块构建指导 |
