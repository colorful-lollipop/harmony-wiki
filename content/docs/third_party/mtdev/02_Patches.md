# mtdev Patch 详细分析

## Patch 概述

OpenHarmony 对 mtdev 库进行了少量但关键的修改，主要通过 **1 个 Patch 文件**实现。这些修改主要针对触摸事件的处理逻辑，以满足 OpenHarmony 多模态输入系统的特定需求。

### Patch 统计

| 维度 | 数量 | 说明 |
|-----|------|-----|
| Patch 文件数量 | 1 | `mtdev_0000.diff` |
| 修改的源文件 | 1 | `src/core.c` |
| 修改的函数数量 | 2 | `push_slot_changes()`, `apply_typeA_changes()` |
| 修改的代码行数 | ~15 行 | 新增条件判断和宏包装 |
| 引入的编译宏 | 1 | `DISABLE_FILTER` |

---

## Patch 详细分析

### Patch 文件信息

- **文件路径**: `patch/diff_libmtdev_mmi/mtdev/mtdev_0000.diff`
- **版权**: Copyright (C) 2021-2023 Huawei Device Co., Ltd.
- **许可证**: Apache License, Version 2.0
- **修改类型**: 功能适配 + 行为变更

---

## Patch 1: mtdev_0000.diff

### 修改文件
- `src/core.c`

### 修改点 1: 强制发送 X/Y 坐标事件

#### 位置
`push_slot_changes()` 函数，第 252-258 行（原代码）

#### 原始代码
```c
static void push_slot_changes(struct mtdev_state *state,
			      const struct mtdev_slot *data, bitmask_t prop,
			      int slot, const struct input_event *syn)
{
	struct input_event ev;
	int i, count = 0;
	foreach_bit(i, prop)
		if (get_sval(&state->data[slot], i) != get_sval(data, i))
			count++;
	// ...
```

#### 修改后代码
```c
static void push_slot_changes(struct mtdev_state *state,
			      const struct mtdev_slot *data, bitmask_t prop,
			      int slot, const struct input_event *syn)
{
	struct input_event ev;
	int i, count = 0;
	foreach_bit(i, prop)
		if (mtdev_mt2abs(i) == ABS_MT_POSITION_X || mtdev_mt2abs(i) == ABS_MT_POSITION_Y ||
			get_sval(&state->data[slot], i) != get_sval(data, i))
			count++;
	// ...
```

#### 修改说明
**原始逻辑**: 仅当属性值发生变化时（`!=` 比较）才计入变化数量。

**修改后逻辑**: 如果是 X 或 Y 坐标属性，无论值是否变化，都计入变化数量。

**影响**: 即使 X/Y 坐标值未发生实际变化，也会被标记为"需要发送"的事件。

---

### 修改点 2: 强制输出 X/Y 事件

#### 位置
`push_slot_changes()` 函数，第 268-276 行（原代码）

#### 原始代码
```c
	foreach_bit(i, prop) {
		ev.code = mtdev_mt2abs(i);
		ev.value = get_sval(data, i);
		if (get_sval(&state->data[slot], i) != ev.value) {
			evbuf_put(&state->outbuf, &ev);
			set_sval(&state->data[slot], i, ev.value);
		}
	}
```

#### 修改后代码
```c
	foreach_bit(i, prop) {
		ev.code = mtdev_mt2abs(i);
		ev.value = get_sval(data, i);
		if (ev.code == ABS_MT_POSITION_X || ev.code == ABS_MT_POSITION_Y ||
			get_sval(&state->data[slot], i) != ev.value) {
			evbuf_put(&state->outbuf, &ev);
			set_sval(&state->data[slot], i, ev.value);
		}
	}
```

#### 修改说明
**原始逻辑**: 仅当属性值发生变化时，才将事件添加到输出缓冲区。

**修改后逻辑**: 如果是 X 或 Y 坐标事件（`ev.code == ABS_MT_POSITION_X || ev.code == ABS_MT_POSITION_Y`），无论值是否变化，都会：
1. 将事件添加到输出缓冲区 (`evbuf_put`)
2. 更新内部状态 (`set_sval`)

**影响**: **强制输出所有 X/Y 坐标事件**，即使坐标值未发生实际变化。

---

### 修改点 3: 条件禁用数据过滤

#### 位置
`apply_typeA_changes()` 函数，第 297-310 行（原代码）

#### 原始代码
```c
	foreach_bit(slot, state->used) {
		if (state->data[slot].tracking_id != id)
			continue;
		filter_data(state, dev, &data[i], prop[i], slot);
		push_slot_changes(state, &data[i], prop[i], slot, syn);
		SETBIT(used, slot);
		id = MT_ID_NULL;
		break;
	}
```

#### 修改后代码
```c
	foreach_bit(slot, state->used) {
		if (state->data[slot].tracking_id != id)
			continue;
#ifdef DISABLE_FILTER
		(void)filter_data;
#else
		filter_data(state, dev, &data[i], prop[i], slot);
#endif
		push_slot_changes(state, &data[i], prop[i], slot, syn);
		SETBIT(used, slot);
		id = MT_ID_NULL;
		break;
	}
```

#### 修改说明
**原始逻辑**: 在推送事件前，调用 `filter_data()` 对数据进行过滤处理（去抖动、平滑等）。

**修改后逻辑**:
- 如果定义了 `DISABLE_FILTER` 宏，则**不执行** `filter_data()` 调用
- 否则，保持原有的过滤逻辑

**OpenHarmony 配置**: 在 `BUILD.gn` 中定义了该宏：
```gn
config("libmtdev-third_config") {
  cflags = [
    "-DDISABLE_FILTER",  # 禁用数据过滤
  ]
}
```

**影响**: **完全禁用了数据过滤功能**，所有原始事件直接传递，不进行平滑或去抖动处理。

---

## 修改目的分析

### 为什么需要强制发送 X/Y 事件？

**推测原因**：

1. **触摸事件完整性**
   - OpenHarmony 的上层输入子系统可能需要收到完整的触摸事件流（包括每个 SYN_REPORT 周期的所有 X/Y 坐标）
   - 某些手势识别算法依赖于周期性的位置更新

2. **驱动层兼容性**
   - 某些硬件驱动可能存在事件丢失或时序不稳定的问题
   - 强制发送可以增加冗余，提高鲁棒性

3. **触摸延迟优化**
   - 通过避免条件判断，减少事件处理的延迟（但会增加事件量）

4. **调试和诊断**
   - 便于上层调试触摸事件流，了解每个采样周期的完整状态

### 为什么禁用数据过滤？

**推测原因**：

1. **保持原始精度**
   - 某些应用场景（如精密绘图、签名识别）需要原始的触摸坐标
   - 过滤算法可能引入额外的延迟或精度损失

2. **上层已有处理**
   - OpenHarmony 的 MMI 层或应用层可能已有更高级的平滑、去抖动逻辑
   - 底层不过滤，让上层根据具体应用场景灵活处理

3. **性能考虑**
   - 避免重复的计算开销

**潜在风险**:
- 可能增加事件噪声
- 上游的 EWMA（指数加权移动平均）过滤可能用于特定硬件的抖动抑制

---

## OH 需求关联

### 多模态输入子系统（MMI）需求

根据 OpenHarmony 的输入系统设计，mtdev 的修改可能与以下需求相关：

| OH 需求 | 对应的 Patch 修改 | 推测原因 |
|---------|-----------------|---------|
| 触摸事件实时性 | 强制发送 X/Y 事件 | 确保每个采样周期都有位置事件 |
| 手势识别准确性 | 禁用过滤 | 保持原始坐标精度，避免平滑导致手势特征丢失 |
| 多种硬件兼容性 | 强制发送 X/Y 事件 | 应对不同驱动的时序行为差异 |
| 调试和故障诊断 | 修改点 1 & 2 | 提供完整的事件流，便于定位问题 |

### 对应的 OH 模块

Patch 修改直接影响以下 OH 模块：

1. **libinput** (`third_party/libinput`)
   - 直接依赖 mtdev，接收其转换后的事件
   - Patch 导致 libinput 收到更频繁的 X/Y 事件

2. **MMI 服务** (`foundation/multimodalinput/input/service`)
   - 间接依赖 mtdev（通过 libinput）
   - 触摸事件规范化模块 (`touch_event_normalize`) 会收到原始未过滤的事件

3. **手势识别模块**
   - 如果依赖周期性的位置更新，Patch 改动可能提升识别准确率

---

## 回归风险

### 升级上游版本时的注意事项

| 风险点 | 说明 | 缓解措施 |
|-------|------|---------|
| **函数签名变化** | `push_slot_changes()` 或 `apply_typeA_changes()` 函数原型可能变化 | 升级后检查函数签名，重新适配 Patch |
| **内联或优化** | 上游可能重构核心逻辑，使 Patch 失效 | 逐行对比源码，找到等效修改点 |
| **新过滤逻辑** | 上游可能引入新的过滤算法 | 评估是否需要在新版本中也禁用 |
| **性能回归** | 强制发送 X/Y 可能增加事件量和 CPU 占用 | 监控性能指标，必要时调整策略 |

### 功能影响评估

| 影响维度 | 评估 | 说明 |
|---------|------|-----|
| **性能影响** | 中等 | 事件量增加可能导致 CPU 占用上升 |
| **功耗影响** | 低 | 事件处理增加的功耗较小 |
| **延迟影响** | 低-中 | 跳过过滤可能降低延迟，但事件量增加可能相反 |
| **精度影响** | 无 | 仅修改事件发送逻辑，不改变坐标值 |
| **兼容性** | 中 | 可能不兼容依赖原始过滤行为的上层代码 |

### 测试建议

在以下场景下验证 Patch 的有效性：

1. **性能测试**
   - 监控触摸事件处理频率
   - 测量 CPU 占用率（使用 `top` 或性能分析工具）
   - 检查事件延迟（从硬件事件到应用层的时延）

2. **功能测试**
   - 测试多点触控（2-10 点）稳定性
   - 测试手势识别（滑动、捏合、旋转）
   - 测试触摸精度（绘图应用）

3. **压力测试**
   - 快速连续触摸
   - 边缘触摸
   - 湿手指或低质量触摸屏测试

---

## 升级建议

### Patch 维护策略

| Patch 部分 | 是否建议向上游贡献 | 原因 |
|----------|-----------------|------|
| 强制发送 X/Y 事件 | **不建议** | 这是 OH 特有的行为，可能不是通用需求 |
| 禁用过滤 (`DISABLE_FILTER`) | **不建议** | 可通过编译选项控制，不影响上游 |

### 未来优化方向

1. **可配置行为**
   ```c
   // 建议添加配置选项
   struct mtdev_config {
       int force_xy_updates;    // 是否强制发送 X/Y
       int enable_filter;       // 是否启用过滤
   };
   ```

2. **性能优化**
   - 仅在特定场景下强制发送 X/Y（如手势识别模式）
   - 提供运行时切换接口

3. **文档补充**
   - 在 README 中说明 OH 特定的修改原因
   - 提供性能测试数据作为修改依据

---

## Patch 文件清单总结

| 序号 | Patch 文件 | 修改文件 | 修改函数 | 关键修改 |
|-----|-----------|---------|---------|---------|
| 1 | `mtdev_0000.diff` | `src/core.c` | `push_slot_changes()` | 强制发送 X/Y 事件 |
| 2 | `mtdev_0000.diff` | `src/core.c` | `apply_typeA_changes()` | 条件禁用过滤 |

---

## 相关文档

- [01_Overview.md](01_Overview.md) - mtdev 库概览
- [03_Build_Integration.md](03_Build_Integration.md) - 构建系统和 Patch 应用机制
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 在 OpenHarmony 中的使用
