# SANE-backends Wiki

## 库概览

**SANE-backends** 是 OpenHarmony 系统中用于支持扫描仪设备的核心第三方库，提供标准化的扫描仪硬件访问接口。

| 属性 | 值 |
|------|-----|
| **组件名称** | @ohos/backends |
| **上游名称** | sane-backends |
| **版本** | 1.4.0 |
| **许可证** | GPL v2 |
| **子系统** | thirdparty |

---

## 什么是 SANE？

**SANE** (Scanner Access Now Easy) 是一个开源项目，提供统一的 API 来访问各种光栅图像扫描仪硬件，包括：
- 平板扫描仪
- 手持扫描仪
- 视频和静态相机
- 帧抓取器

官方网站: [http://sane-project.org/](http://sane-project.org/)

---

## 在 OpenHarmony 中的作用

### 南向生态适配
在 OpenHarmony 南向生态发展过程中，需要对存量市场的扫描仪进行兼容。使用 SANE 扫描系统能：
- 直接对接市场上大部分的扫描仪
- 减少扫描仪驱动适配 OpenHarmony 系统的难度
- 提供统一的扫描服务接口

### 应用场景
- 文档扫描应用
- 照片数字化
- 条码/二维码识别
- OCR 文字识别

---

## OH 适配概述

### 主要适配工作

| 适配类型 | 说明 |
|----------|------|
| **性能优化** | 添加线程池支持，实现并发设备发现 |
| **日志集成** | 接入 HiLog 日志系统，统一日志管理 |
| **路径适配** | 适配 OH 沙箱目录结构 |
| **构建适配** | 从 Autotools 迁移到 GN 构建系统 |

### Patch 统计

- **总 Patch 数**: 8 个
- **OH 特有 Patch**: 4 个
- **上游移植 Patch**: 4 个

### 主要 Patch

1. **`add_thread_poll.patch`** - 线程池并发优化
2. **`hilog_debug.patch`** - HiLog 日志集成
3. **`modifying_driver_search_path.patch`** - 驱动路径适配
4. **`modify_load_function.patch`** - 加载错误处理优化

---

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](./02_Patches.md) | Patch 详细分析（核心文档） |
| [03_Build_Integration.md](./03_Build_Integration.md) | OH 构建适配说明 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用方式 |

---

## 快速开始

### 头文件导入
```c
#include <sane/sane.h>
```

### 添加编译依赖
在 `bundle.json` 中添加：
```json
"deps": {
  "third_party": [
    "backends"
  ]
}
```

在 `BUILD.gn` 中添加：
```gn
deps += [ "//third_party/backends:third_sane" ]
```

### 基础使用示例
```c
SANE_Status status;
SANE_Handle handle;

// 初始化 SANE
status = sane_init(NULL, NULL);
if (status != SANE_STATUS_GOOD) {
    // 处理错误
}

// 获取设备列表
const SANE_Device **device_list;
status = sane_get_devices(&device_list, SANE_FALSE);

// 打开设备
status = sane_open(device_list[0]->name, &handle);

// 开始扫描
status = sane_start(handle);

// ... 读取扫描数据 ...

// 清理
sane_cancel(handle);
sane_close(handle);
sane_exit();
```

---

## 相关仓库

- [print_print_fwk](https://gitee.com/openharmony/print_print_fwk) - 打印框架（扫描功能模块）

---

## 更多信息

- 上游项目: https://gitlab.com/sane-project/backends
- 上游文档: https://sane-project.gitlab.io/standard/
- OH 问题反馈: https://gitee.com/openharmony/third_party_backends/issues

---

*本文档是 OpenHarmony 第三方库适配文档的一部分，专注于记录 SANE-backends 在 OH 中的定制化内容。*
