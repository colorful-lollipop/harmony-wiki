# 02 Patch 详细分析

> **核心文档** - 本章详细记录 OpenHarmony 对 libevdev 的所有 Patch 适配

---

## 2.1 Patch 清单总表

| 序号 | Patch 文件 | 修改文件数 | 修改类型 | OH 适配分类 | 可推向上游 |
|------|-----------|-----------|---------|-------------|-----------|
| 1 | `libevdev_0000.diff` | 5 | 综合补丁 | OH 特有/Bug 修复 | 部分 |

### Patch 文件位置

```
third_party/libevdev/
└── patch/
    └── diff_libevdev_mmi/
        └── libevdev/
            └── libevdev_0000.diff
```

### 修改文件概览

| 文件路径 | 修改行数 | 主要变更 |
|---------|---------|---------|
| `include/linux/freebsd/input-event-codes.h` | +15 | 新增键值定义 |
| `include/linux/linux/input-event-codes.h` | +15 | 新增键值定义 |
| `include/linux/linux/input.h` | +6 | 添加 BUS_SDW、修改头文件保护 |
| `libevdev/libevdev-uinput.c` | +12 | 添加 FDSAN 安全注解 |
| `libevdev/libevdev-util.h` | +6 | 位操作使用 1ULL |
| `libevdev/libevdev.c` | +35 | 多点触控同步逻辑修复 |

---

## 2.2 Patch 详细分析

### Patch: libevdev_0000.diff (综合补丁)

#### 元信息

| 属性 | 值 |
|------|-----|
| **Patch 文件** | `libevdev_0000.diff` |
| **修改文件数** | 6 |
| **创建日期** | 2025 (根据版权年份) |
| **Patch 性质** | 综合补丁 |

---

#### 2.2.1 新增键值定义

**修改文件**：
- `include/linux/freebsd/input-event-codes.h`
- `include/linux/linux/input-event-codes.h`

**修改摘要**：
新增一系列 OH 特有的键值定义，用于支持新的交互场景。

**新增的键值**：

```c
// 通讯相关
#define KEY_LINK_PHONE         0x1bf   /* AL Phone Syncing */

// 手柄/游戏控制相关
#define BTN_GRIPL              0x224
#define BTN_GRIPR              0x225
#define BTN_GRIPL2             0x226
#define BTN_GRIPR2             0x227

// 显示相关
#define KEY_REFRESH_RATE_TOGGLE 0x232  /* Display refresh rate toggle */

// 辅助功能相关
#define KEY_ACCESSIBILITY       0x24e  /* Toggles system accessibility UI/command (HUTRR116) */
#define KEY_DO_NOT_DISTURB     0x24f  /* Toggles system-wide Do Not Disturb control (HUTRR94) */

// 性能/游戏模式
#define KEY_PERFORMANCE         0x2bd  /* Performance Boost key / G-Mode key (Alienware/Dell) */
```

**OH 需求**：

这些键值定义支持以下 OpenHarmony 特性：

1. **KEY_LINK_PHONE** - 支持与手机的协同互联功能
2. **BTN_GRIPL/R** 系列 - 支持游戏手柄的握持按键
3. **KEY_REFRESH_RATE_TOGGLE** - 支持屏幕刷新率切换
4. **KEY_ACCESSIBILITY** - 无障碍功能快捷键
5. **KEY_DO_NOT_DISTURB** - 勿扰模式切换
6. **KEY_PERFORMANCE** - 性能模式/游戏模式切换

**回归风险**：

| 风险项 | 等级 | 说明 |
|-------|------|------|
| 上游合并 | 低 | 需与上游协商键值分配 |
| 冲突风险 | 低 | 使用 0x1bf-0x2bd 范围内未使用值 |
| 兼容性 | 高 | 不影响现有功能 |

**升级建议**：

1. **短期**：保留该 Patch，直接使用这些键值
2. **长期**：推动上游接受 KEY_LINK_PHONE、KEY_ACCESSIBILITY、KEY_DO_NOT_DISTURB
3. **替代方案**：若上游不接受，可考虑在内核头文件中定义，而非 libevdev

---

#### 2.2.2 BUS_SDW 总线类型支持

**修改文件**：
- `include/linux/linux/input.h`

**修改摘要**：
添加 SoundWire (SDW) 总线类型定义。

**关键代码变更**：

```c
// 在 BUS 类型枚举中添加
#define BUS_CEC                0x1E
#define BUS_INTEL_ISHTP        0x1F
#define BUS_AMD_SFH            0x20
#define BUS_SDW                0x21   /* SoundWire Bus */
```

**OH 需求**：

SoundWire 是一种用于移动设备的串行总线协议，主要用于音频编解码器。添加 BUS_SDW 支持：

1. **音频设备识别**：支持通过 libevdev 识别 SoundWire 音频设备
2. **设备分类**：正确分类 SoundWire 输入设备
3. **兼容性**：与 Linux 内核 5.4+ 版本保持一致

**技术背景**：

```c
/* 
 * 总线类型 (bus) 定义了设备的物理连接方式
 * 常见类型：BUS_USB, BUS_BLUETOOTH, BUS_I2C, BUS_SPI
 * 新增 BUS_SDW 用于 SoundWire 音频设备
 */
```

**回归风险**：

| 风险项 | 等级 | 说明 |
|-------|------|------|
| 上游合并 | 中 | 需确认上游是否已有类似定义 |
| 冲突风险 | 低 | 使用 0x21，与现有定义不冲突 |

**升级建议**：

1. **首选方案**：从 Linux 内核头文件同步 BUS_SDW 定义
2. **备选方案**：保持本地定义，等待上游更新
3. **验证方式**：检查内核版本是否支持 SoundWire

---

#### 2.2.3 FDSAN 文件描述符安全注解

**修改文件**：
- `libevdev/libevdev-uinput.c`

**修改摘要**：
为 uinput 设备创建和管理添加 FDSAN (File Descriptor Sanitizer) 注解，防止文件描述符 Use-After-Free 漏洞。

**FDSAN 背景**：

FDSAN 是 OpenHarmony 引入的文件描述符安全机制，通过为每个文件描述符分配唯一标签（tag），检测以下问题：

- **Use-After-Close**：关闭后使用
- **Double-Close**：重复关闭
- **Close-Wrong-Tag**：错误标签关闭

**关键代码变更**：

```c
// 新增常量定义
static const uint64_t FDSAN_NEW_TAG = 0xD002800;

// 在文件打开时分配标签
fd = open("/dev/uinput", O_RDWR|O_CLOEXEC);
if (fd < 0)
    goto error;
fdsan_exchange_owner_tag(fd, 0, FDSAN_NEW_TAG);  // 分配新标签

// 在文件关闭时验证标签
fdsan_close_with_tag(fd, FDSAN_NEW_TAG);  // 使用带标签的关闭

// 在现有文件操作时交换标签
fdsan_exchange_owner_tag(fd, 0, FDSAN_NEW_TAG);
```

**FDSAN API 详解**：

| API | 功能 |
|-----|------|
| `fdsan_exchange_owner_tag(fd, old_tag, new_tag)` | 交换文件描述符的所有权标签 |
| `fdsan_close_with_tag(fd, tag)` | 使用标签关闭文件描述符 |
| `fdsan_create_owner_tag(pid, random)` | 创建新的所有者标签 |

**OH 价值**：

1. **安全性提升**
   - 检测 uinput 设备文件描述符的异常使用
   - 防止资源管理错误导致的漏洞
   - 符合 OpenHarmony 安全基线要求

2. **调试支持**
   - 当检测到问题时输出详细的调用栈
   - 快速定位文件描述符泄漏

3. **合规性**
   - 满足 OpenHarmony 安全审计要求

**代码位置分析**：

| 位置 | 操作 | 修改类型 |
|------|------|---------|
| `alloc_uinput_device()` | 打开设备文件 | 添加 FDSAN 注解 |
| `fetch_syspath_and_devnode()` | 读取设备信息 | 替换 close() 为 fdsan_close_with_tag() |
| `libevdev_uinput_create_from_device()` | 创建设备 | 添加 FDSAN 注解 |
| `libevdev_uinput_destroy()` | 销毁设备 | 替换 close() 为 fdsan_close_with_tag() |

**回归风险**：

| 风险项 | 等级 | 说明 |
|-------|------|------|
| 上游合并 | 不建议 | OH 特有安全机制 |
| 兼容性 | 高 | 不影响功能，仅添加安全检查 |
| 性能影响 | 低 | 标签操作开销极小 |

**升级建议**：

1. **保留**：该 Patch 是 OH 安全加固的核心部分，必须保留
2. **验证**：确保所有文件描述符操作都经过 FDSAN 验证
3. **扩展**：考虑将 FDSAN 推广到 libevdev 的其他文件描述符操作

---

#### 2.2.4 位操作 Bug 修复

**修改文件**：
- `libevdev/libevdev-util.h`

**修改摘要**：
修复位操作中使用 `1LL` 改为 `1ULL`，确保在 64 位系统上的正确性。

**问题背景**：

在 C 语言中：
- `1LL` 是 long long 类型的字面量（至少 32 位）
- `1ULL` 是 unsigned long long 类型的字面量（至少 32 位，推荐 64 位）

**关键代码变更**：

```c
// 修改前
static inline int
bit_is_set(const unsigned long *array, int bit)
{
    return !!(array[bit / LONG_BITS] & (1LL << (bit % LONG_BITS)));
                                     ^^^^^
                                     错误：可能溢出

static inline void
set_bit(unsigned long *array, int bit)
{
    array[bit / LONG_BITS] |= (1LL << (bit % LONG_BITS));
                                     ^^^^^
                                     错误：可能溢出
}

// 修改后
static inline int
bit_is_set(const unsigned long *array, int bit)
{
    return !!(array[bit / LONG_BITS] & (1ULL << (bit % LONG_BITS)));
                                     ^^^^^^
                                     正确：明确使用 64 位无符号

static inline void
set_bit(unsigned long *array, int bit)
{
    array[bit / LONG_BITS] |= (1ULL << (bit % LONG_BITS));
                                     ^^^^^^
                                     正确：明确使用 64 位无符号
}
```

**问题分析**：

```c
/* 
 * 当 bit >= 32 时，问题显现：
 * 
 * 假设 bit = 50, LONG_BITS = 64 (64位系统)
 * 
 * 使用 1LL (32位):
 * 1LL << 50 = 0x4000000000000  (实际被截断为 0)
 * 结果：错误地认为位未设置
 * 
 * 使用 1ULL (64位):
 * 1ULL << 50 = 0x4000000000000  (正确移位)
 * 结果：正确识别位设置状态
 */
```

**OH 需求**：

1. **正确性**：确保 64 位系统上的位操作正确
2. **兼容性**：支持 Linux 64 位内核驱动
3. **健壮性**：避免潜在的位操作错误

**回归风险**：

| 风险项 | 等级 | 说明 |
|-------|------|------|
| 上游合并 | **高** | 这是一个明确的 Bug 修复 |
| 兼容性 | 高 | 不影响现有功能 |
| 回归风险 | 低 | 修复正确性问题 |

**升级建议**：

⭐ **强烈建议推向上游**

1. **提交方式**：向上游提交 Bug 修复 Patch
2. **附加说明**：解释 64 位系统的位操作问题
3. **测试用例**：添加针对 32 位以上位的测试用例

---

#### 2.2.5 多点触控同步逻辑修复

**修改文件**：
- `libevdev/libevdev.c`

**修改摘要**：
修复 `push_mt_sync_events()` 函数中的多点触控同步逻辑，避免重复发送 ABS_MT_TRACKING_ID = -1 事件。

**问题背景**：

在多点触控场景中，当触摸槽被终止时（tracking ID = -1），需要在 `terminate_slots` 中先发送终止事件。如果 `push_mt_sync_events` 再次发送相同的事件，会导致：

1. **重复事件**：同一触摸事件被发送两次
2. **状态不一致**：应用层收到错误的触摸状态
3. **竞态条件**：极端情况下可能导致触摸卡顿

**关键代码变更**：

```c
// 修改前
static int
push_mt_sync_events(struct libevdev *dev, ...)
{
    for (int slot = 0; slot < dev->num_slots; slot++) {
        // 问题：总是先发送 ABS_MT_SLOT 事件
        if (changes[slot].state == TOUCH_STOPPED ||
            !bit_is_set(changes[slot].axes, ABS_MT_SLOT))
            continue;

        queue_push_event(dev, EV_ABS, ABS_MT_SLOT, slot);  // ← 总是发送
        last_reported_slot = slot;

        for (int axis = ABS_MT_MIN; axis <= ABS_MT_MAX; axis++) {
            // 问题：对已终止的 tracking ID 再次发送 -1
            if (bit_is_set(changes[slot].axes, axis)) {
                queue_push_event(dev, EV_ABS, axis, *slot_value(dev, slot, axis));
            }
        }
    }
}

// 修改后
static int
push_mt_sync_events(struct libevdev *dev, ...)
{
    for (int slot = 0; slot < dev->num_slots; slot++) {
        bool have_slot_event = false;  // ← 新增标记

        if (!bit_is_set(changes[slot].axes, ABS_MT_SLOT))
            continue;

        for (int axis = ABS_MT_MIN; axis <= ABS_MT_MAX; axis++) {
            if (axis == ABS_MT_SLOT ||
                !libevdev_has_event_code(dev, EV_ABS, axis))
                continue;

            if (bit_is_set(changes[slot].axes, axis)) {
                // 跳过已发送的 tracking ID = -1
                if (axis == ABS_MT_TRACKING_ID &&
                    *slot_value(dev, slot, axis) == -1)
                    continue;

                // 仅在实际有触摸事件时才发送 SLOT
                if (!have_slot_event) {
                    queue_push_event(dev, EV_ABS, ABS_MT_SLOT, slot);
                    have_slot_event = true;
                }

                queue_push_event(dev, EV_ABS, axis, *slot_value(dev, slot, axis));
            }
        }
    }
}
```

**修改逻辑详解**：

```
修改前流程：
1. 对每个有变化的槽位
2. 总是先发送 ABS_MT_SLOT 事件
3. 发送所有轴的值（包括 tracking ID = -1）

修改后流程：
1. 对每个有变化的槽位
2. 检查是否需要发送 slot 事件（至少有一个有效轴）
3. 跳过已终止的 tracking ID = -1
4. 发送其他有效的触摸数据
```

**OH 需求**：

1. **触摸体验**：消除多点触控的重复事件问题
2. **触摸精度**：确保触摸位置的准确性
3. **系统稳定性**：避免触摸事件导致的系统卡顿

**回归风险**：

| 风险项 | 等级 | 说明 |
|-------|------|------|
| 上游合并 | **高** | 这是重要的 Bug 修复 |
| 兼容性 | 中 | 可能影响依赖原有行为的应用 |
| 测试覆盖 | 建议 | 需要完整的触摸测试 |

**升级建议**：

⭐ **强烈建议推向上游**

1. **提交方式**：向上游提交 Bug 修复和测试用例
2. **附加说明**：详细解释修改前后的行为差异
3. **测试建议**：测试多点触控手势（捏合、缩放、旋转等）

---

## 2.3 Patch 升级策略

### 可推向上游的 Patch

| Patch 部分 | 优先级 | 理由 |
|-----------|--------|------|
| 位操作修复 (1ULL) | **高** | 明确的 Bug 修复 |
| 多点触控同步修复 | **高** | 重要的 Bug 修复 |

### OH 特有的 Patch

| Patch 部分 | 优先级 | 理由 |
|-----------|--------|------|
| FDSAN 注解 | **必须保留** | OH 安全机制 |
| 新键值定义 | 中 | 可协商推向上游 |
| BUS_SDW | 低 | 可从内核头文件同步 |

### 版本升级检查清单

```
版本升级时需要检查：
□ 确认位操作修复是否已在上游版本中修复
□ 确认多点触控同步是否已在上游版本中修复
□ 重新应用 FDSAN 注解
□ 重新添加 OH 特有键值定义（若上游未接受）
□ 验证 BUS_SDW 定义是否与内核一致
□ 运行完整的触摸测试用例
□ 运行多点触控手势测试
```

---

## 2.4 安全性分析

### FDSAN 安全价值

| 方面 | 评估 |
|------|------|
| **漏洞类型** | 防止 Use-After-Free / Double-Close |
| **防护效果** | 高 |
| **性能影响** | 可忽略 |
| **维护成本** | 低 |

### 位操作修复的安全影响

| 方面 | 评估 |
|------|------|
| **漏洞类型** | 潜在的位操作溢出 |
| **触发条件** | bit >= 32 时触发 |
| **风险等级** | 低 |
| **修复效果** | 消除潜在错误 |

---

## 下一章

下一章将介绍 **OpenHarmony 构建适配**，详细说明 BUILD.gn 配置和构建系统的适配细节。

👉 **[03_Build_Integration.md](./03_Build_Integration.md)** →
