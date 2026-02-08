# OpenHarmony Peripheral Wiki

## 概述

本文档为 OpenHarmony `drivers/peripheral` 仓库的工程 Wiki，提供外设驱动的完整技术文档。

## 覆盖范围

### 包含内容
- **所有外设驱动模块**（audio, camera, display, input, sensor, usb, wlan, ril 等）
- **HDI 接口定义**（Hardware Driver Interface）
- **HAL 实现代码**
- **GN 构建配置**
- **模块依赖关系**
- **安全风险分析**

### 不包含内容
- 测试代码（`test/`, `*_test.*`, `fuzz/` 等）
- 外部关联仓库详细文档（仅提供链接）

## 文档结构

```
wiki/
├── README.md                    # 本文档
├── SUMMARY.md                   # 全站导航
├── index.md                     # 项目首页
├── 01_Overview.md              # 项目概述
├── 02_Module_Catalog.md         # 模块目录
├── 03_HDI_Interfaces.md         # HDI 接口说明
├── 04_Architecture.md           # 架构说明
├── 05_GN_Build.md               # GN 构建配置
├── 06_Security_Review.md        # 安全风险评审
├── 07_Build_Artifacts.md        # 编译产物
├── appendix/
│   ├── Callgraphs.md            # 关键调用链
│   └── Config_Flags.md          # 配置宏
└── _work/                       # 工作区
    ├── NOTES.md                 # 事实记录
    └── PLAN.md                  # 任务计划
```

## 阅读建议

### 新人快速上手
1. 阅读 `index.md` 了解项目定位
2. 阅读 `01_Overview.md` 理解系统架构
3. 根据需要查看 `03_HDI_Interfaces.md` 了解具体模块接口

### 开发者
- 接口开发：查看 `03_HDI_Interfaces.md` 和对应模块的 `interfaces/` 目录
- HAL 实现：查看各模块的 `hal/` 目录
- 构建配置：查看 `05_GN_Build.md`

## 更新方式

### 何时更新 Wiki
- 新增外设驱动模块
- 修改 HDI 接口定义
- 更改模块间依赖关系
- 发现安全风险

### 更新流程
1. 在 `_work/NOTES.md` 中记录发现
2. 更新对应 Markdown 文件
3. 确保 `SUMMARY.md` 链接一致
4. 验证文档可读性和准确性

## 版本信息

- **生成时间**: 2026-02-06
- **仓库版本**: OpenHarmony drivers/peripheral
- **文档版本**: v1.0

## 关联仓库

- [Driver Subsystem](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/驱动子系统.md)
- [drivers_framework](https://gitee.com/openharmony/drivers_framework/blob/master/README_zh.md)
- [drivers_adapter](https://gitee.com/openharmony/drivers_adapter/blob/master/README_zh.md)
- [drivers_adapter_khdf_linux](https://gitee.com/openharmony/drivers_adapter_khdf_linux/blob/master/README_zh.md)
