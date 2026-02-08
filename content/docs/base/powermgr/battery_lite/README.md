# OpenHarmony Lite Battery Manager 工程 Wiki 生成

**生成时间**: 2026-02-06
**项目**: powermgr_battery_lite
**版本**: 3.1

## 项目概述

Lite Battery Manager 是 OpenHarmony 电源管理子系统的轻量级电池服务组件，为 mini 和 small 系统提供电池信息查询、充放电状态监测、电池健康状态监控以及充电指示灯控制等核心功能。

### 核心能力

1. **电池信息查询**: 获取电池电量(SOC)、电压、温度、技术型号
2. **充放电状态监测**: 实时监测充电状态、连接器类型
3. **电池健康监控**: 健康状态评估（过热、过压、亏电等）
4. **充电指示灯控制**: LED 颜色和开关控制

### 适用系统

- **mini 系统**: 基于 LiteOS-M 内核的轻量设备
- **small 系统**: 基于更高性能芯片的小型设备

---

## 文档覆盖范围

| 文档 | 状态 | 说明 |
|------|------|------|
| README.md | ✅ | 本文档，覆盖范围和更新方式 |
| SUMMARY.md | ✅ | 全站导航和新人阅读顺序 |
| 00_Overview.md | ✅ | 项目定位、核心能力、运行环境 |
| 01_Directory_Structure.md | ✅ | 目录结构和模块职责 |
| 02_Architecture.md | ✅ | 组件图、数据流、线程模型 |
| 03_N_API.md | ✅ | JS API 接口清单和绑定 |
| 04_Inner_API.md | ✅ | 内部模块接口和依赖 |
| 05_GN_Build.md | ✅ | GN Targets 和编译产物 |
| 06_Security_Review.md | ✅ | 安全风险评审 |
| appendix/Callgraphs.md | 🔄 | 关键调用链（可选） |
| appendix/Config_Flags.md | 🔄 | 关键配置开关（可选） |

---

## 代码证据来源

本文档所有关键结论均基于以下代码证据：

### 核心头文件
- `services/include/battery_device.h`: 电池设备特征 API 定义
- `services/include/ibattery.h`: 电池接口定义
- `services/include/battery_manage_service.h`: 电池管理服务
- `services/include/battery_manage_feature.h`: 电池管理特征

### 接口定义
- `interfaces/kits/battery_info.h`: 电池信息枚举和公共接口
- `interfaces/kits/js/@system.battery.d.ts`: JS API 类型定义

### 框架层
- `frameworks/native/include/battery_framework.h`: Native 框架接口
- `frameworks/native/include/battery_mgr.h`: 电池管理器常量
- `frameworks/native/include/batterymgr_intf_define.h`: 接口定义宏

### JS 绑定
- `frameworks/js/builtin/include/battery_module.h`: JS 模块接口
- `frameworks/js/builtin/src/battery_module.cpp`: JS 模块实现

### 构建配置
- `BUILD.gn`: 根构建入口
- `batterymgr.gni`: 构建配置参数
- `bundle.json`: 组件配置

---

## 更新方式

### 手动更新
当代码发生变更时，需同步更新对应文档：

1. **新增 API**: 在 `03_N_API.md` 添加 API 清单表
2. **修改架构**: 在 `02_Architecture.md` 更新组件图
3. **变更构建**: 在 `05_GN_Build.md` 更新 Targets 列表
4. **发现风险**: 在 `06_Security_Review.md` 添加风险条目

### 版本对照

| Wiki 版本 | 代码版本 | 更新内容 |
|-----------|----------|----------|
| 1.0 | 3.1 | 初始版本，完整文档覆盖 |

---

## 快速索引

### 新人阅读顺序

1. **README.md**: 了解文档覆盖范围
2. **00_Overview.md**: 理解项目定位
3. **01_Directory_Structure.md**: 熟悉代码结构
4. **02_Architecture.md**: 掌握架构设计
5. **03_N_API.md**: 学习 API 使用
6. **06_Security_Review.md**: 了解安全边界

### API 使用速查

```c
// Native API 调用示例
int32_t soc = GetBatSoc();
BatteryChargeState state = GetChargingStatus();
```

```javascript
// JS API 调用示例
battery.BatterySOC({
    success: (data) => { console.log(data.batterySoc); }
});
```

---

## 相关资源

- **OpenHarmony 电源管理子系统**: [docs](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/电源管理子系统.md)
- **完整电池管理器**: powermgr_battery_manager
- **电源管理器**: powermgr_power_manager
- **显示管理器**: powermgr_display_manager
- **温控管理器**: powermgr_thermal_manager
- **电池统计**: powermgr_battery_statistics
