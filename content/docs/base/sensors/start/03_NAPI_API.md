# N-API 文档

**适用范围**: 本文档适用于所有需要了解 N-API 接口的人员
**目的**: 说明 N-API 接口、JS API 清单、参数验证、错误码
**关键结论**: **本组件不提供任何 N-API/JS API**

---

## N-API 概述

### 组件特点

⚠️ **重要提示**: `sensors_start` 是一个**纯配置型组件**

**不包含**:
- ❌ 无 JavaScript 绑定代码
- ❌ 无 N-API 模块定义
- ❌ 无 JS API 导出
- ❌ 无 `.ts` / `.js` 源代码

**职责**:
- ✅ 仅提供 INIT 系统启动配置文件
- ✅ 服务实现由外部组件提供（`sensors_sensor`、`sensors_miscdevice`、`msdp`）

### N-API 证据

通过仓库文件列表可以确认:

| N-API 相关元素 | 搜索结果 | 证据 |
|---------------|----------|------|
| `napi_` 函数 | 未找到 | 仓库无任何 `.cpp`/`.c` 源代码 |
| `NAPI_MODULE` 宏 | 未找到 | 仓库无任何源代码 |
| `napi_module_register` | 未找到 | 仓库无任何源代码 |
| `napi_define_properties` | 未找到 | 仓库无任何源代码 |
| `.ts` / `.js` 文件 | 未找到 | 仓库仅包含配置文件 |

---

## JS API 说明

### 本组件

**不提供任何 JS API**

传感器和 msdp 服务的 JS API 由以下组件提供（不在本仓库）:

| 组件 | JS API | 仓库 |
|------|--------|------|
| sensors_sensor | 传感器 JS API | openharmony/sensors_sensor |
| sensors_miscdevice | 震动器等 JS API | openharmony/sensors_miscdevice |
| msdp 相关组件 | msdp JS API | msdp 相关仓库 |

### 传感器 JS API (外部)

**不在本仓库，位于 `sensors_sensor` 组件**

**典型 API** (示例):
```javascript
// 订阅加速度传感器数据
import sensor from '@ohos.sensor';

sensor.on(sensor.SensorType.ACCELEROMETER, (data) => {
  console.log('x: ' + data.x + ', y: ' + data.y + ', z: ' + data.z);
});
```

**说明**: 传感器 JS API 的详细文档请查阅 `sensors_sensor` 仓库

### 震动器 JS API (外部)

**不在本仓库，位于 `sensors_miscdevice` 组件**

**典型 API** (示例):
```javascript
import vibrator from '@ohos.vibrator';

vibrator.vibrate(1000); // 震动 1 秒
```

**说明**: 震动器 JS API 的详细文档请查阅 `sensors_miscdevice` 仓库

---

## 相关跳转

- [项目概览](./00_Overview.md) - 组件定位和职责
- [内部 API](./04_Inner_API.md) - 内部 API 说明
- [外部组件 JS API]:
  - [sensors_sensor JS API](https://gitee.com/openharmony/sensors_sensor)
  - [sensors_miscdevice JS API](https://gitee.com/openharmony/sensors_miscdevice)

---

**最后更新**: 2026-02-06
