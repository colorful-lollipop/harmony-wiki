# 01 - 原始库简介

## 1.1 库基本信息

| 属性 | 详情 |
|------|------|
| **全称** | SANE-backends (Scanner Access Now Easy) |
| **版本** | 1.4.0 |
| **发布日期** | 2025-05-11 |
| **许可证** | GPL v2 |
| **上游维护者** | SANE 项目团队 |
| **上游地址** | https://gitlab.com/sane-project/backends |

## 1.2 原始功能描述

SANE 是一个提供标准化访问光栅图像扫描仪硬件的应用程序编程接口(API)。其设计目标是：

- **标准化接口**：为各种扫描仪硬件提供统一的访问接口
- **硬件抽象**：应用开发者无需了解具体硬件细节
- **后端架构**：通过动态加载后端(driver)支持不同设备
- **网络支持**：支持网络扫描仪访问

### 支持的设备类型
- 平板扫描仪 (Flatbed scanners)
- 手持扫描仪 (Hand-held scanners)
- 自动文档进纸器 (ADF - Automatic Document Feeder)
- 幻灯片扫描仪 (Film/slide scanners)
- 视频采集设备 (Video and still-cameras)
- 多功能一体机 (MFP - Multi-Function Printers)

### 主要后端支持
| 后端名称 | 厂商/设备类型 |
|----------|--------------|
| `pixma` | Canon PIXMA 系列 |
| `escl` | eSCL/AirPrint 设备 (通用) |
| `epson2` | Epson 扫描仪 |
| `epsonds` | Epson DS/PX/WF 系列 |
| `fujitsu` | Fujitsu 扫描仪 |
| `avision` | Avision 扫描仪 |
| `genesys` | 多款基于 Genesys 芯片的设备 |
| `hpaio` | HP 一体机 |
| `xerox_mfp` | Xerox/Samsung/Dell 一体机 |

## 1.3 在 OpenHarmony 中的定位

### 业务需求背景

在 OpenHarmony 南向生态发展过程中，需要兼容存量市场的扫描仪设备：

1. **市场存量大**：传统扫描仪设备数量庞大，用户期望在 OH 设备上继续使用
2. **协议标准化**：SANE 是 Linux/Unix 系统上最成熟的扫描仪支持方案
3. **驱动复用**：可直接使用现有的 SANE 后端，无需重新开发驱动

### 系统架构位置

```
┌─────────────────────────────────────────┐
│           应用层 (Application)           │
│    扫描应用 / OCR应用 / 文档管理应用      │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│         打印框架 (print_print_fwk)       │
│            扫描服务模块                   │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│      SANE-backends (third_party)        │
│    ┌──────────────────────────────┐     │
│    │    dll.c (动态加载器)         │     │
│    │    + 线程池并发优化           │     │
│    │    + HiLog 日志集成           │     │
│    └──────────────────────────────┘     │
│    ┌──────────────────────────────┐     │
│    │    后端驱动 (Backends)        │     │
│    │    escl, pixma, epsonds...    │     │
│    └──────────────────────────────┘     │
└──────────────┬──────────────────────────┘
               │
┌──────────────▼──────────────────────────┐
│         系统服务层                        │
│    libusb (USB设备通信)                  │
│    hilog (日志服务)                       │
│    c_utils (线程池)                       │
└─────────────────────────────────────────┘
```

### 关键使用场景

| 场景 | 说明 |
|------|------|
| **文档数字化** | 将纸质文档扫描为 PDF/JPEG 文件 |
| **照片扫描** | 扫描老照片进行数字化保存 |
| **条码识别** | 扫描条码/二维码进行信息录入 |
| **OCR 输入** | 扫描文档后进行文字识别 |
| **网络扫描** | 访问网络扫描仪设备 |

## 1.4 OH 与上游的差异

### 核心差异总结

| 方面 | 上游 SANE | OpenHarmony 适配版 |
|------|-----------|-------------------|
| **构建系统** | Autotools (configure/make) | GN + Python |
| **日志系统** | 标准错误输出/系统日志 | HiLog 统一日志 |
| **设备发现** | 串行扫描各后端 | 线程池并发扫描 |
| **路径结构** | FHS 标准路径 (/etc, /usr/lib) | OH 沙箱路径 |
| **驱动加载** | 固定命名规范 | 支持动态搜索和压缩格式 |

### 新增功能

1. **线程池并发设备发现**
   - 使用 OHOS::ThreadPool 实现
   - 最大 20 线程并发
   - 显著减少多后端设备发现时间

2. **HiLog 日志集成**
   - 统一接入 OH 日志系统
   - 支持日志级别管理
   - 便于问题诊断

3. **扫描服务适配**
   - 支持 OH 沙箱目录结构
   - 动态配置加载
   - 适配扫描服务架构

### 移除/禁用功能

- 移除了前端工具（scanimage, saned 等）- 由 OH 扫描服务替代
- 禁用了部分需要特殊权限的功能
- 移除了文档生成（由 Wiki 替代）

## 1.5 API 兼容性

OpenHarmony 适配版保持了与上游 SANE API 的完全兼容：

### 核心 API 函数
```c
// 初始化和清理
SANE_Status sane_init(SANE_Int *version_code, SANE_Auth_Callback authorize);
void sane_exit(void);

// 设备枚举
SANE_Status sane_get_devices(const SANE_Device ***device_list, SANE_Bool local_only);

// 设备操作
SANE_Status sane_open(SANE_String_Const devicename, SANE_Handle *handle);
void sane_close(SANE_Handle handle);

// 扫描控制
SANE_Status sane_start(SANE_Handle handle);
SANE_Status sane_read(SANE_Handle handle, SANE_Byte *data, SANE_Int max_length, SANE_Int *length);
void sane_cancel(SANE_Handle handle);
```

### 状态码
所有 `SANE_Status` 状态码与上游保持一致，确保应用层无需修改即可迁移。

---

## 参考链接

- [SANE 官方网站](http://sane-project.org/)
- [SANE 标准文档](https://sane-project.gitlab.io/standard/)
- [上游 GitLab 仓库](https://gitlab.com/sane-project/backends)
- [OpenHarmony 打印框架](https://gitee.com/openharmony/print_print_fwk)
