# mtdev 库概览

## 原始库信息

### 基本信息
- **库名称**: mtdev (Multitouch Protocol Translation Library)
- **版本**: 1.1.7
- **许可证**: MIT License
- **上游地址**: http://bitmath.org/git/mtdev

### 功能描述
mtdev 是一个独立的库，用于将内核各种变体的多点触控（MT）事件转换为 Type B 插槽协议。该库可以处理来自任何 MT 设备的事件，具体包括：
- Type A 设备（无触点跟踪）
- Type A 设备（带触点跟踪）
- Type B 设备（带触点跟踪）

mtdev 的核心代码自 2008 年起作为 Multitouch X Driver 的一部分存在。通过此包，手指跟踪和无缝 MT 协议处理以自由许可证的形式提供。

### 目录结构
```
mtdev/
├── include/           # API 头文件
├── src/               # 封装层实现
│   ├── caps.c         # 设备能力检测
│   ├── core.c         # 核心逻辑
│   ├── iobuf.c        # 输入缓冲区
│   ├── match.c        # 事件匹配
│   └── match_four.c   # 四点匹配
├── test/              # 测试代码
└── doc/               # 文档
```

### 官方 API 文档
- API 参考: http://bitmath.org/code/mtdev/

---

## OpenHarmony 中的定位

### 组件信息
- **OH 组件名称**: @ohos/mtdev
- **版本**: 3.1
- **所属子系统**: thirdparty (多模态输入子系统 multimodalinput)
- **所属 Part**: input
- **资源占用**:
  - ROM: 400KB
  - RAM: 800KB

### 在 OH 系统中的作用

mtdev 是 OpenHarmony 多模态输入系统（MMI）的**核心基础库**，位于输入事件处理链的底层：

```
内核触摸驱动 → mtdev (协议转换) → libinput (设备抽象) → MMI 服务 (事件分发) → 应用层
```

### 核心职责

1. **协议标准化**: 将不同硬件触点设备产生的各种 MT 事件统一转换为标准的 Type B 协议
2. **设备兼容性**: 支持各种类型的触摸硬件，无需上层关心底层协议差异
3. **事件过滤与处理**: 提供触摸事件的基础过滤和处理逻辑（OH 中部分禁用）

### 关键特性（OH 版本）

| 特性 | 说明 | OH 适配状态 |
|-----|------|-----------|
| 多点触控支持 | 支持最多 10 点触控 | ✅ 完整支持 |
| 协议转换 | Type A → Type B 转换 | ✅ 完整支持 |
| 事件过滤 | 数据去重和平滑处理 | ⚠️ 禁用 (`DISABLE_FILTER`) |
| 设备能力检测 | 自动检测设备支持的 MT 事件类型 | ✅ 完整支持 |
| 硬件兼容 | 支持各种触摸屏、触摸板、触摸笔 | ✅ 完整支持 |

---

## 适用场景

### 支持的设备类型
- 智能手机触摸屏
- 平板电脑触摸屏
- 可穿戴设备触摸屏
- 触摸板（如笔记本触摸板）
- 数字化仪/绘图板

### 不支持的设备类型
- 单点触控设备（无 MT 事件）
- 非触摸输入设备（键盘、鼠标等）

---

## 快速开始

### 1. 头文件引用
```c
#include <mtdev.h>
```

### 2. BUILD.gn 依赖添加
```gn
deps = [
    "//third_party/mtdev:libmtdev-third-mmi",
]
```

### 3. 基本使用示例
```c
#include <mtdev.h>
#include <fcntl.h>
#include <unistd.h>

void process_mt_events(int fd) {
    struct mtdev dev;
    struct input_event ev;

    // 打开 mtdev 设备
    int ret = mtdev_open(&dev, fd);
    if (ret < 0) {
        // 错误处理
        return;
    }

    // 获取设备能力
    if (mtdev_has_mt_event(&dev, ABS_MT_POSITION_X)) {
        // 设备支持 X 坐标
    }

    // 读取并处理事件
    while (!mtdev_idle(&dev, fd, 5000)) {
        while (mtdev_get(&dev, fd, &ev, 1) > 0) {
            // 处理每个事件
            switch (ev.type) {
                case EV_ABS:
                    // 绝对轴事件
                    break;
                case EV_SYN:
                    // 同步事件
                    break;
            }
        }
    }

    mtdev_close(&dev);
}
```

---

## 相关文档

- [02_Patches.md](02_Patches.md) - OpenHarmony 特定 Patch 详细分析
- [03_Build_Integration.md](03_Build_Integration.md) - GN 构建系统适配说明
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 在 OpenHarmony 中的使用和依赖关系

---

## 参考资源

- 上游仓库: http://bitmath.org/git/mtdev
- 官方文档: http://bitmath.org/code/mtdev/
- 内核 MT 协议规范: Linux kernel documentation (Multitouch Protocol)
