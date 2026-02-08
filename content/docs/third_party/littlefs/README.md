# littlefs - OpenHarmony 集成文档

> littlefs 是一个专为微控制器设计的小型故障安全文件系统，在 OpenHarmony 中主要用于 LiteOS-M 和 UniProton 内核的嵌入式文件系统支持。

---

## 快速导航

| 文档 | 说明 | 适用场景 |
|------|------|----------|
| **[01_Overview](./01_Overview.md)** | 库概览、OH 适配概述 | 了解 littlefs 在 OH 中的定位 |
| **[02_Patches](./02_Patches.md)** | Patch 详细分析 | 了解 OH 特定修改（如有）|
| **[03_Build_Integration](./03_Build_Integration.md)** | OH 构建适配 | 集成 littlefs 到 OH 项目 |
| **[04_Usage_in_OH](./04_Usage_in_OH.md)** | 依赖关系与使用 | 了解谁在使用 littlefs |
| **[05_API_Differences](./05_API_Differences.md)** | API/接口差异 | 了解 OH 特有 API |
| **[06_Security](./06_Security.md)** | 安全风险分析 | 了解安全特性和 CVE 状态 |

---

## littlefs 在 OpenHarmony 中的定位

### 核心作用

littlefs 在 OpenHarmony 中作为**嵌入式 MCU 文件系统**提供关键支持：

1. **LiteOS-M 内核标准文件系统**
   - 为轻量级内核提供可靠的文件系统支持
   - 通过 VFS 层适配，与 POSIX 兼容
   - 适用于需要持久化存储的 IoT 设备

2. **UniProton RTOS 文件系统**
   - 为实时操作系统提供文件系统支持
   - 条件编译控制，可按需启用

3. **芯片厂商深度集成**
   - 海思 WS63V100 星闪芯片 SDK 内置 littlefs
   - Rockchip RK2206 芯片 HAL 层适配
   - QEMU 模拟器全覆盖支持

### 适用场景

| 场景 | 描述 |
|------|------|
| **IoT 设备数据持久化** | 配置文件、日志、用户数据存储 |
| **固件更新** | 版本信息、更新状态管理 |
| **数据采集** | 传感器数据缓存和上传 |
| **开发调试** | QEMU 模拟器环境测试 |

---

## OH 适配概述

### 适配方式特点

| 特点 | 说明 |
|------|------|
| **无代码 Patch** | littlefs 核心代码与上游完全一致 |
| **适配层隔离** | 通过 VFS 适配层（lfs_adapter.c）实现集成 |
| **配置外置** | 编译选项由引用方控制（littlefs.gni + lfs_conf.h）|
| **版本同步** | OH 版本与上游最新版本保持一致（v2.11.2）|

### 关键适配文件

| 文件 | 用途 |
|------|------|
| **littlefs.gni** | GN 构建系统适配（源文件列表 + 头文件路径）|
| **lfs_adapter.c** | LiteOS-M VFS 适配层实现 |
| **lfs_conf.h** | OH 配置参数定义 |

---

## 技术特点

### 三大核心特性

1. **Power-loss resilience（断电恢复）**
   - 所有文件操作都有强 copy-on-write 保证
   - 断电后会回退到最后已知良好状态
   - 适合嵌入式设备的不可靠电源环境

2. **Dynamic wear leveling（动态磨损均衡）**
   - 专为 Flash 存储设计
   - 动态块磨损均衡，延长 Flash 寿命
   - 可检测坏块并绕过

3. **Bounded RAM/ROM（内存边界限制）**
   - RAM 使用量严格受限，不随文件系统增长而变化
   - 无无界递归，适合资源受限的 MCU

### 资源占用

| 资源类型 | 占用量 | 说明 |
|---------|--------|------|
| **ROM** | 56KB | 代码空间占用 |
| **RAM** | 112KB | 运行时内存占用 |
| **适用系统** | mini | 轻量级系统 |

---

## 版本信息

| 项目 | 版本 |
|------|------|
| **上游版本** | v2.11.2 (LFS_VERSION: 0x0002000b) |
| **磁盘版本** | lfs2.1 (LFS_DISK_VERSION: 0x00020001) |
| **OH 组件版本** | 3.1 |
| **许可证** | BSD 3-Clause |
| **上游地址** | https://github.com/littlefs-project/littlefs |

**版本状态**: ✅ OH 版本与上游最新版本保持一致，无需升级。

---

## 快速开始

### 在 LiteOS-M 中使用 littlefs

```c
#include "lfs.h"
#include "lfs_adapter.h"

// 挂载 littlefs 文件系统
ret = mount(NULL, "/data", "littlefs", 0, &littlefsConfig);
if (ret != 0) {
    printf("mount littlefs failed: %d\n", ret);
    return -1;
}

// 创建文件
int fd = open("/data/config.txt", O_CREAT | O_WRONLY, 0644);
if (fd >= 0) {
    write(fd, "hello littlefs", 15);
    close(fd);
}

// 读取文件
fd = open("/data/config.txt", O_RDONLY);
if (fd >= 0) {
    char buf[64];
    int len = read(fd, buf, sizeof(buf));
    printf("read: %.*s\n", len, buf);
    close(fd);
}

// 卸载文件系统
ret = umount("/data");
```

### 编译选项配置

在 BUILD.gn 中启用 littlefs 支持：

```gn
import("//third_party/littlefs/littlefs.gni")

// 定义编译选项
config("littlefs_config") {
  defines = [
    "LFS_THREADSAFE=1",  # 启用线程安全
    "LFS_NO_DEBUG=1",    # 禁用调试日志
  ]
}

// 引用 littlefs 源文件
sources = LITTLEFS_SRC_FILES_FOR_KERNEL_MODULE + [
  "lfs_adapter.c",  # OH 适配层
]

include_dirs = LITTLEFS_INCLUDE_DIRS + [ "." ]
```

---

## 相关资源

### 上游资源

- [littlefs GitHub 仓库](https://github.com/littlefs-project/littlefs)
- [littlefs 官方文档](https://github.com/littlefs-project/littlefs/blob/master/README.md)
- [DESIGN.md - 设计原理](https://github.com/littlefs-project/littlefs/blob/master/DESIGN.md)
- [SPEC.md - 磁盘格式规范](https://github.com/littlefs-project/littlefs/blob/master/SPEC.md)

### OH 相关资源

- [OH 文件系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/file/README.md)
- [LiteOS-M VFS 文档](https://gitee.com/openharmony/kernel_liteos_m/blob/master/README.md)

### 依赖模块

- [kernel/liteos_m/components/fs/littlefs](../../../oh/kernel/liteos_m/components/fs/littlefs) - LiteOS-M 适配层
- [kernel/uniproton](../../../oh/kernel/uniproton) - UniProton 内核

---

## 常见问题

### Q: littlefs 与 FATfs 有什么区别？

**A**: littlefs 是专为嵌入式设备设计的故障安全文件系统，主要优势：
- 断电恢复：即使突然断电也能保证文件系统完整性
- 磨损均衡：延长 Flash 寿命
- 无需 FAT 表：减少写入次数，适合小容量 Flash

FATfs 更适合需要与 PC 互操作的场景（如 SD 卡）。

### Q: OH 版本是否有代码修改？

**A**: 没有。littlefs 核心代码（lfs.c, lfs.h, lfs_util.c/h）与上游完全一致，通过适配层（lfs_adapter.c）实现集成。

### Q: 如何启用 littlefs？

**A**: 在 LiteOS-M 配置中启用 `LOSCFG_FS_LITTLEFS`，然后在内核编译时选择 littlefs 文件系统支持。

### Q: littlefs 支持的最大文件大小？

**A**: 默认为 2GB，可通过修改 `LFS_FILE_MAX` 宏调整。

---

## 贡献与反馈

如有问题或建议，请联系：
- **维护者**: wangmihu@huawei.com
- **Issues**: [Gitee littlefs](https://gitee.com/openharmony/third_party_littlefs/issues)

---

**最后更新时间**: 2026-02-08
**OH 版本**: 3.1
**littlefs 版本**: v2.11.2
