# 02 - Patch 详细分析

## 2.1 Patch 清单总览

### 独立 Patch 文件

经过全面搜索，**f2fs-tools 没有独立的 .patch 文件**。所有修改都直接在源代码中维护。

### 代码级修改分类

| 类型 | 数量 | 说明 |
|-----|------|------|
| `WITH_OHOS` 条件编译 | 3 处 | 控制 OH 特有代码路径 |
| 新增源文件 | 3 个 | libf2fs_log.c, libf2fs_dmd.c, extra_fsck.c |
| 新增头文件 | 5 个 | f2fs_log.h, f2fs_dmd.h 等 |
| config.h 定义 | 1 处 | 定义 WITH_OHOS 宏 |

---

## 2.2 WITH_OHOS 宏条件编译分析

### Patch 1: 禁用 SCSI INQUIRY 命令

**位置**: `lib/libf2fs.c:899-903` 和 `lib/libf2fs.c:1001-1016`

#### 原始问题
上游代码使用 SCSI INQUIRY 命令查询磁盘型号：
```c
#if !defined(WITH_OHOS) && defined(__linux__)
sg_io_hdr_t io_hdr;
unsigned char reply_buffer[96] = {0};
unsigned char model_inq[6] = {MODELINQUIRY};
// ... SCSI 命令发送代码
#endif
```

#### 修改内容
```c
// lib/libf2fs.c:899-903
#if !defined(WITH_OHOS) && defined(__linux__)
    sg_io_hdr_t io_hdr;
    unsigned char reply_buffer[96] = {0};
    unsigned char model_inq[6] = {MODELINQUIRY};
#endif
```

```c
// lib/libf2fs.c:1001-1016
#if !defined(WITH_OHOS) && defined(__linux__)
    /* Send INQUIRY command */
    memset(&io_hdr, 0, sizeof(sg_io_hdr_t));
    io_hdr.interface_id = 'S';
    io_hdr.dxfer_direction = SG_DXFER_FROM_DEV;
    io_hdr.dxfer_len = sizeof(reply_buffer);
    io_hdr.dxferp = reply_buffer;
    io_hdr.cmd_len = sizeof(model_inq);
    io_hdr.cmdp = model_inq;
    io_hdr.timeout = 1000;

    if (!ioctl(fd, SG_IO, &io_hdr)) {
        MSG(0, "Info: [%s] Disk Model: %.16s\n",
                dev->path, reply_buffer+16);
    }
#endif
```

#### OH 需求
- **原因**: OpenHarmony 运行环境可能不完全支持标准 Linux SCSI 命令
- **场景**: 某些设备（如虚拟化环境、特定硬件）可能不支持 SG_IO ioctl
- **影响**: 禁用后不再显示磁盘型号信息，但不影响核心功能

#### 回归风险
- **风险等级**: 低
- **说明**: 此功能仅为信息展示，不影响 F2FS 格式化和检查的核心逻辑

---

### Patch 2: 禁用交互式文件恢复

**位置**: `fsck/fsck.c:3815-3830`

#### 原始问题
上游 fsck 在发现孤立 inode 时会提示用户是否恢复到 `./lost_found/`：
```c
#ifndef WITH_OHOS
if (nr_unref_nid && !c.ro) {
    char ans[255] = {0};
    int res;

    printf("\nDo you want to restore lost files into ./lost_found/? [Y/N] ");
    res = scanf("%s", ans);
    // ... 文件恢复逻辑
}
#endif
```

#### 修改内容
```c
#ifndef WITH_OHOS
    if (nr_unref_nid && !c.ro) {
        char ans[255] = {0};
        int res;

        printf("\nDo you want to restore lost files into ./lost_found/? [Y/N] ");
        res = scanf("%s", ans);
        ASSERT(res >= 0);
        if (!strcasecmp(ans, "y")) {
            for (i = 0; i < fsck->nr_nat_entries; i++) {
                if (f2fs_test_bit(i, fsck->nat_area_bitmap))
                    dump_node(sbi, i, 1);
            }
        }
    }
#endif
```

#### OH 需求
- **原因**: OpenHarmony 启动环境中通常没有 TTY 交互界面
- **场景**: init 服务在启动阶段调用 fsck，无法等待用户输入
- **影响**: 禁用后跳过文件恢复提示，孤儿 inode 按默认方式处理

#### 回归风险
- **风险等级**: 中
- **说明**: 数据恢复功能被禁用，极端情况下可能导致数据不可恢复
- **缓解**: 生产环境中应确保正常关机，减少需要 fsck 的场景

---

### Patch 3: config.h 中定义 WITH_OHOS

**位置**: `config.h:213`

#### 修改内容
```c
#define WITH_OHOS 1
```

#### OH 需求
- 启用上述所有 WITH_OHOS 条件编译代码路径
- 作为全局开关，统一管理 OH 特有行为

---

## 2.3 新增源文件分析

### 文件 1: libf2fs_log.c

**文件路径**: `lib/libf2fs_log.c`
**行数**: 528 行
**创建时间**: 2023-11-24
**作者**: Huawei Technologies Co., Ltd.

#### 功能概述
增强 f2fs-tools 的日志系统，支持：
1. **内核日志 (kmsg)**: 通过 `/dev/kmsg` 写入内核日志缓冲区
2. **文件日志**: 持久化日志到 `/log/f2fs-tools/` 或 `/dev/f2fs-tools/`
3. **分类日志**: 支持 fsck.log, mkfs.log, dump.log, defrag.log, resize.log

#### 关键接口
```c
// 初始化日志系统
int SlogInit(int funcType);

// 写入系统日志（kmsg）
void KlogWrite(int level, const char* fmt, ...);

// 写入文件日志
int SlogWrite(const char *fmt, ...);

// 关闭日志系统
void SlogExit(void);
```

#### 日志级别
```c
#define KLOG_ERROR_LEVEL        3
#define KLOG_INFO_LEVEL         6
#define KLOG_DEFAULT_LEVEL      KLOG_INFO_LEVEL
```

#### 日志存储
- **主路径**: `/log/f2fs-tools/`
- **备用路径**: `/dev/f2fs-tools/`
- **文件权限**: 0664 (用户可读写，组可读)
- **目录权限**: 0775
- **所有者**: root:system

#### OH 价值
- **问题诊断**: 持久化日志便于现场问题分析
- **生产环境**: kmsg 日志可通过 `dmesg` 查看
- **性能分析**: 记录操作耗时和结果

---

### 文件 2: libf2fs_dmd.c

**文件路径**: `lib/libf2fs_dmd.c`
**行数**: 261 行
**创建时间**: 2024-03-04
**作者**: Huawei Technologies Co., Ltd.

#### 功能概述
DFX（Design For X）诊断模块，实现：
1. **错误统计**: 分类统计文件系统检查中发现的错误
2. **性能监控**: 监控 fsck 执行时间，识别超时场景
3. **设备状态**: 读取设备锁定状态（Bootloader 锁定）
4. **错误上报**: 通过 `/dev/storage` 接口上报到系统

#### 关键数据结构
```c
struct DmdReport {
    uint64_t errBitmap[NR_ERR_BITMAP_UINT64];  // 错误位图
    unsigned int propBitmap;                    // 属性标志
    unsigned long usedSpace;                    // 已用空间 (MB)
    unsigned long freeSpace;                    // 空闲空间 (MB)
    unsigned int features;                      // F2FS 特性
    unsigned short segsPerSec;                  // 每段扇区数
    unsigned short secsPerZone;                 // 每区段数
    unsigned long totalFsSectors;               // 总扇区数
    unsigned long ckptVersion;                  // Checkpoint 版本
    unsigned int ckptState;                     // Checkpoint 状态
    unsigned long costTime;                     // 执行耗时
    char msg[FSCK_REPORT_MSG_SIZE];             // 详细消息
};
```

#### 关键接口
```c
// 上报诊断信息
int DmdReport(void);

// 记录错误
void DmdInsertError(int type, unsigned int err, const char *func, int line);

// 检查耗时是否超限
void DmdCheckCostTime(const char *func, int line);
```

#### 错误位图定义
```c
// 来自 f2fs_dmd_errno.h
#define PR_SUCCESS              0
#define PR_ERROR                1
#define PR_FATAL_ERROR          2
#define PR_MISMATCH             3
// ... 更多错误码
```

#### 设备状态检测
```c
// 检测 Bootloader 锁定状态
static void ReadDeviceState(void)
{
    const char cmdlinePath[] = "/proc/cmdline";
    const char matchStr[] = "ohos.boot.hvb.device_state=locked";
    // ... 读取并解析 cmdline
}
```

#### 上报接口
```c
#define HIEVENT_DRIVER_NODE  "/dev/storage"
#define EVENT_REPORT_FSCK_CMD _IOW('S', 1, struct DmdReport)
```

#### OH 价值
- **质量监控**: 收集文件系统健康度数据
- **故障预警**: 识别频繁出现的错误模式
- **性能基线**: 建立 fsck 执行时间基线
- **安全分析**: 结合设备锁定状态分析攻击面

---

### 文件 3: extra_fsck.c

**文件路径**: `lib/extra_fsck.c`
**功能**: 额外的 fsck 辅助函数，支持 sload.f2fs 功能

---

## 2.4 新增头文件分析

| 文件 | 功能 |
|-----|------|
| `include/f2fs_log.h` | 日志系统接口定义 |
| `include/f2fs_dmd.h` | DFX 诊断接口定义，包含宏和辅助函数 |
| `include/f2fs_dmd_cfg.h` | DFX 配置常量（DMD_OK/DMD_ERR） |
| `include/f2fs_dmd_errno.h` | DFX 错误码枚举 |
| `include/f2fs_dfx_common.h` | DFX 通用类型定义（LogType 枚举等） |

---

## 2.5 Git 历史中的 Bugfix 分析

### 已合并的关键 Bugfix

| Commit | 描述 | 状态 | 分支 |
|--------|------|------|------|
| `8d94938` | fix malloc not free in libf2fs_zoned.c | ✅ 已合并 | 5.0.2 |
| `983c441` | fix malloc not free in libf2fs_zoned.c | ✅ 已合并 | 4.1 |
| `86ed66c` | fix malloc not free in libf2fs_zoned.c | ✅ 已合并 | 5.0.3 |
| `92cf264` | Fix xattr max value len bug | ✅ 已合并 | master |
| `a30faae` | Fix casefold dlen bug | ✅ 已合并 | master |
| `fbdf052` | add permissive mode for fsck | ✅ 已合并 | master |

### Bugfix 详情

#### 1. 内存泄漏修复
**问题**: `libf2fs_zoned.c` 中 malloc 分配的内存未释放
**影响**: 长时间运行可能导致内存泄漏
**修复**: 在函数退出前添加 free 调用

#### 2. xattr 最大长度 Bug
**问题**: 扩展属性值长度检查不正确
**影响**: 可能导致越界访问
**修复**: 修正长度验证逻辑

#### 3. casefold 长度计算 Bug
**问题**: 大小写折叠文件名长度计算错误
**影响**: 使用大小写不敏感特性时可能出错
**修复**: 修正 dlen 计算

#### 4. fsck 宽容模式
**问题**: fsync 数据错误导致 fsck 失败
**影响**: 某些场景下无法正常完成文件系统检查
**修复**: 添加宽容模式，允许跳过特定错误

---

## 2.6 Patch 升级建议

### 可以推向上游的修改

目前所有 `WITH_OHOS` 修改都是 OH 特有的，不适合推向上游。

### 需要持续维护的修改

| 修改 | 维护策略 |
|-----|---------|
| WITH_OHOS 宏 | 每次升级需检查新增代码是否需要条件编译 |
| libf2fs_log.c | 与上游日志系统独立，需独立维护 |
| libf2fs_dmd.c | OH 特有功能，需独立维护 |

### 升级检查清单

升级上游版本时需检查：
- [ ] `lib/libf2fs.c` 中是否有新增 SCSI 相关代码
- [ ] `fsck/fsck.c` 中是否有新增交互式功能
- [ ] 新增源文件是否需要 WITH_OHOS 保护
- [ ] 新增的 .h 文件是否需要添加到 BUILD.gn
- [ ] bounds_checking_function 的 API 兼容性
