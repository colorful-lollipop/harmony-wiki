# ucollection - 统一采集

## 1. 概述

### 1.1 模块定位

ucollection（unified collection）通过 ioctl 方式向设备节点 `/dev/ucollection` 发送命令，快速获取进程 CPU 维测数据。

**证据来源**: `ucollection/README_zh.md:1-6`

```
ucollection/README_zh.md:1-6
当前系统需要获取进程的CPU维测数据，通过遍例所有进程的结点虽然可以获取所有进程的CPU使用率，
但是该方法性能相对较差，为了提升获取效率，因此开辟了内核的设备结点，通过结点可能快速获取CPU的数据。
```

---

## 2. 目录结构

```
/Volumes/lexar/code/d/work/oh/kernel/linux/common_modules/ucollection/
├── unified_collection_driver.c  # 设备驱动 (90行)
├── unified_collection_data.h    # 数据结构
├── ucollection_process_cpu.c/h  # 进程 CPU 数据 (339行)
├── Kconfig                       # 配置: CONFIG_UNIFIED_COLLECTION
├── Makefile                      # 构建2个目标文件
├── figures/                      # 架构图
├── README_zh.md                 # 文档
└── apply_ucollection.sh          # 部署脚本
```

**证据来源**: `ucollection/README_zh.md:8-22`

---

## 3. 核心接口

### 3.1 设备接口

| 函数 | 功能 |
|------|------|
| `unified_collection_ioctl()` | 主 ioctl 处理器 |
| `unified_collection_open()` | 打开设备 |
| `unified_collection_release()` | 关闭设备 |

### 3.2 IOCTL 命令

| 命令 | 功能 |
|------|------|
| `IOCTRL_COLLECT_ALL_PROC_CPU` | 采集所有进程 CPU 数据 |
| `IOCTRL_COLLECT_THE_PROC_CPU` | 采集指定进程 CPU |
| `IOCTRL_COLLECT_PROC_COUNT` | 获取进程数量 |
| `IOCTRL_COLLECT_THREAD_COUNT` | 获取进程线程数 |
| `IOCTRL_COLLECT_APP_THREAD_COUNT` | 采集自进程线程数 |
| `IOCTRL_COLLECT_THE_THREAD` | 采集指定线程 CPU |

### 3.3 数据结构

| 结构 | 说明 |
|------|------|
| `ucollection_process_cpu_entry` | 进程 CPU 数据容器 |
| `ucollection_process_cpu_item` | 单个进程指标 |
| `ucollection_thread_cpu_entry` | 线程 CPU 数据容器 |
| `ucollection_thread_cpu_item` | 单个线程指标 |

---

## 4. 相关文档

| 文档 | 说明 |
|------|------|
| [05_Troubleshooting.md](../05_Troubleshooting.md) | 调试与故障排查 |
