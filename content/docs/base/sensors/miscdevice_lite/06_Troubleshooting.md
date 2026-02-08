# sensors_miscdevice_lite 常见问题与排查

## ⚠️ 重要声明

**本文档基于标准实现和通用故障模式推断**。

当前仓库**不包含源代码**，因此部分问题定位需参考主实现仓库：
- **主实现仓库**: https://github.com/openharmony/sensors_miscdevice

---

## 6.1 构建问题

### 6.1.1 构建失败：找不到依赖

**问题描述**:
```
error: cannot find dependency '//base/sensors/miscdevice_lite/interfaces/native:miscddevice_inner_kit'
```

**可能原因**:
1. 子模块未初始化
2. 构建配置不完整
3. 依赖项路径错误

**排查步骤**:

```bash
# 1. 检查子模块状态
git submodule status

# 2. 初始化子模块
git submodule update --init --recursive

# 3. 清理构建缓存
rm -rf out/
hb build -f
```

**解决方案**:

```bash
# 重新同步仓库
git fetch origin
git checkout develop
git pull

# 或检查依赖声明
cat bundle.json | grep -A5 "deps"
```

---

### 6.1.2 Ninja 构建超时

**问题描述**:
```
ninja: fatal: waiting for input: pipe closed
```

**可能原因**:
1. 磁盘空间不足
2. 内存不足
3. 进程被杀死

**排查步骤**:

```bash
# 1. 检查磁盘空间
df -h

# 2. 检查内存
free -h

# 3. 清理构建产物
rm -rf out/linaro64_release/
```

**解决方案**:

```bash
# 增加并行度（减少内存占用）
ninja -C out/... -j2

# 或使用更小的编译集群
hb set  # 选择 memory 优化配置
```

---

## 6.2 运行时问题

### 6.2.1 振动无响应

**问题描述**: 调用 `startVibration()` 但设备无振动

**可能原因**:
| 原因 | 检查方法 |
|------|----------|
| 设备不支持振动 | `hdc shell "ls /dev/ vibration*"` |
| 振动器忙碌 | 检查是否有其他应用占用 |
| 权限未授予 | 检查 `ohos.permission.VIBRATE` |
| 驱动未加载 | `hdc shell "lsmod | grep vibrator"` |

**排查步骤**:

```bash
# 1. 检查设备节点
hdc shell ls -la /dev/vibrator*

# 2. 检查内核模块
hdc shell lsmod | grep -i vibr

# 3. 检查服务状态
hdc shell "bm dump -a" | grep -i vibr

# 4. 查看日志
hdc shell "hilog | grep -i vibrator"
```

**解决方案**:

```bash
# 重新加载驱动
hdc shell "rmmod vibrator_driver"
hdc shell "insmod /vendor/driver/vibrator_driver.ko"

# 或重启设备
hdc shell reboot
```

---

### 6.2.2 LED 灯不亮

**问题描述**: 调用 LED 控制 API 但 LED 无响应

**可能原因**:
| 原因 | 检查方法 |
|------|----------|
| LED 硬件不存在 | 检查设备原理图 |
| LED 驱动未加载 | `hdc shell "lsmod | grep led"` |
| GPIO 配置错误 | 检查设备树配置 |
| LED 亮度为 0 | 检查亮度值 |

**排查步骤**:

```bash
# 1. 检查 LED 节点
hdc shell ls -la /sys/class/leds/

# 2. 检查 LED 亮度
hdc shell "cat /sys/class/leds/*/brightness"

# 3. 手动测试 LED
hdc shell "echo 255 > /sys/class/leds/*/brightness"

# 4. 查看内核日志
hdc shell "dmesg | grep -i led"
```

**解决方案**:

```bash
# 检查设备树 LED 配置
cat /vendor/etc/device_tree/led.dts

# 或使用 sysfs 直接控制
hdc shell "echo 1 > /sys/class/leds/led_name/brightness"
```

---

### 6.2.3 权限被拒绝

**问题描述**: 调用 API 返回权限错误 (错误码 14600103)

**可能原因**:
1. 应用未声明权限
2. 权限未授予
3. 权限级别不匹配

**排查步骤**:

```bash
# 1. 检查应用权限声明
hdc shell "cat /data/app/package_name/config.json" | grep -A5 "requestPermissions"

# 2. 检查权限授予状态
hdc shell " dumpsys app permission <package_name>"

# 3. 检查系统能力
hdc shell "param get const.product.name"  # 确认设备支持
```

**解决方案**:

```json
// 1. 在 module.json5 中添加权限声明
{
    "module": {
        "requestPermissions": [
            {
                "name": "ohos.permission.VIBRATE",
                "reason": "Need vibration for notifications"
            }
        ]
    }
}

// 2. 动态申请权限（API 9+）
import { abilityAccessCtrl, bundleManager } from '@kit.AbilityKit';
```

---

## 6.3 API 调用问题

### 6.3.1 振动效果无效

**问题描述**: 使用预设效果 ID 返回"不支持"错误

**可能原因**:
1. 设备不支持该效果
2. 效果 ID 拼写错误
3. 效果配置文件缺失

**排查步骤**:

```typescript
// 1. 先检查效果是否支持
vibrator.isSupportEffect('haptic.clock.timer', (err, supported) => {
    if (supported) {
        // 效果支持
    } else {
        // 效果不支持
    }
});

// 2. 列出可用效果
// （如果有相关 API）
```

**解决方案**:

```typescript
// 使用通用定时振动替代
vibrator.startVibration(
    { type: 'time', duration: 500 },
    { usage: { scenario: 'notification' } }
);

// 或使用其他预设效果
const PRESET_EFFECTS = [
    'haptic.clock.timer',
    'haptic.feedback.click',
    'haptic.feedback.dragStart'
];
```

---

### 6.3.2 振动无法停止

**问题描述**: 调用 `stopVibration()` 后振动继续

**可能原因**:
1. 调用时机问题
2. 多个振动会话冲突
3. 系统级振动队列

**排查步骤**:

```bash
# 1. 查看当前振动状态
hdc shell "cat /sys/class/timed_output/vibrator/enable"
```

**解决方案**:

```typescript
// 1. 使用强制停止模式
vibrator.stopVibration(vibrator.VibratorStopMode.VIBRATOR_STOP_MODE_ALL);

// 2. 等待振动自然结束
// 3. 检查是否有其他应用同时振动
```

---

## 6.4 性能问题

### 6.4.1 振动启动延迟

**问题描述**: 调用 `startVibration()` 后振动有延迟

**可能原因**:
| 原因 | 影响 |
|------|------|
| IPC 调用延迟 | 跨进程通信开销 |
| 驱动初始化 | 首次调用需要初始化 |
| 系统负载高 | 资源竞争 |

**优化建议**:

```typescript
// 1. 预初始化（如果有相关 API）
// vibrator.preInit();

// 2. 使用短延时振动
vibrator.startVibration({ type: 'time', duration: 50 }, {});

// 3. 避免频繁启停
```

---

### 6.4.2 电池消耗过快

**问题描述**: 振动功能导致电池快速耗尽

**可能原因**:
1. 应用滥用振动 API
2. 振动时长过长
3. 振动频率过高

**检测方法**:

```bash
# 查看振动使用统计
hdc shell "dumpsys batterystats | grep -i vibrator"
```

**建议**:
- 限制单次振动时长（建议 < 5 秒）
- 降低振动频率
- 使用轻量级振动效果

---

## 6.5 日志收集

### 6.5.1 关键日志标签

| 标签 | 用途 |
|------|------|
| `Vibrator` | 振动相关日志 |
| `LED` | LED 控制日志 |
| `HDF` | 驱动框架日志 |
| `MiscDevice` | 综合日志 |

### 6.5.2 日志收集命令

```bash
# 收集所有相关日志
hdc shell "hilog > /data/logs/vibrator.log"

# 过滤特定日志
hdc shell "hilog | grep -E 'Vibrator|LED'"

# 实时日志
hdc shell "hilog -T Vibrator"
```

---

## 6.6 调试工具

### 6.6.1 命令行工具

| 工具 | 用途 |
|------|------|
| `hdc` | 设备连接调试 |
| `hilog` | 日志查看 |
| `bm` | 包管理调试 |
| `param` | 系统参数查看 |

### 6.6.2 调试命令速查

```bash
# 设备连接状态
hdc connect

# 查看系统能力
hdc shell "param get const.product.name"

# 查看安装的应用
hdc shell "bm list -a"

# 查看系统服务
hdc shell "bm dump -a"

# 内核日志
hdc shell "dmesg"
```

---

## 6.7 相关文档

### 官方资源
- [Vibrator API 参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-vibrator)
- [OpenHarmony 开发指南](https://gitee.com/openharmony/docs)
- [hdc 工具使用](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hdc-0000002052)

### 本地文档
- [03_API.md](./03_API.md) - API 文档
- [04_Build.md](./04_Build.md) - 构建配置
- [05_Security.md](./05_Security.md) - 安全评审
