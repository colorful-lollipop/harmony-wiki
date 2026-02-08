# 04 - OpenHarmony 中的依赖与使用

## 4.1 依赖关系总览

### 组件依赖图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          OpenHarmony 系统                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                        应用层                                         │  │
│  │   (应用程序通过 Storage Manager 访问存储)                             │  │
│  └─────────────────────┬────────────────────────────────────────────────┘  │
│                        │                                                    │
│  ┌─────────────────────▼────────────────────────────────────────────────┐  │
│  │                    框架层                                             │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │  │
│  │  │ Storage      │  │ Update       │  │ DLP          │               │  │
│  │  │ Service      │  │ Service      │  │ Service      │               │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │  │
│  │         │                 │                 │                        │  │
│  │         └─────────────────┴─────────────────┘                        │  │
│  │                           │                                          │  │
│  └───────────────────────────┼──────────────────────────────────────────┘  │
│                              │                                              │
│  ┌───────────────────────────▼──────────────────────────────────────────┐  │
│  │                      系统服务层                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │  │
│  │  │ Storage      │  │ Updater      │  │ File         │               │  │
│  │  │ Daemon       │  │ (Recovery)   │  │ Manager      │               │  │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │  │
│  │         │                 │                 │                        │  │
│  │         │    ┌────────────┴─────────────────┘                        │  │
│  │         │    │                                                       │  │
│  │         │    ▼                                                       │  │
│  │         │  ┌─────────────────────────────────────────────────────┐   │  │
│  │         │  │              e2fsprogs                               │   │  │
│  │         │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │  │
│  │         │  │  │ libblkid │  │ e2fsck   │  │ e2fsdroid│           │   │  │
│  │         │  │  │ (识别)   │  │ (修复)   │  │ (制镜像) │           │   │  │
│  │         │  │  └──────────┘  └──────────┘  └──────────┘           │   │  │
│  │         │  └─────────────────────────────────────────────────────┘   │  │
│  │         │                                                            │  │
│  └─────────┼────────────────────────────────────────────────────────────┘  │
│            │                                                                │
│  ┌─────────▼────────────────────────────────────────────────────────────┐  │
│  │                        内核层                                         │  │
│  │                    (ext4 / HMFS 驱动)                                 │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 4.2 直接依赖者

根据 `bundle.json` 中的 `inner_kits` 定义，以下模块直接依赖 e2fsprogs：

### 组件使用者

| 模块 | 使用的库/工具 | 用途 | BUILD.gn 路径 |
|------|--------------|------|--------------|
| storage_service | libext2_blkid, blkid | 存储设备识别 | `//foundation/filemanagement/storage_service` |
| updater | e2fsck, resize2fs | OTA 升级、分区调整 | `//base/update/updater` |
| build_system | e2fsdroid, mke2fs | 镜像制作 | 构建脚本 |
| init | e2fsck | 启动时文件系统检查 | `//base/startup/init` |

### 依赖详情

#### Storage Service (存储服务)

```gn
# storage_service/BUILD.gn (示例)
ohos_executable("storage_daemon") {
  deps = [
    "//third_party/e2fsprogs:libext2_blkid",
    # ...
  ]
}
```

**使用场景**:
- 识别插入的 U 盘/SD 卡文件系统类型
- 获取分区 UUID 和 LABEL
- 支持 VFAT/exFAT/NTFS/ext4/HMFS 等格式

**关键 API**:
```c
blkid_get_cache(&cache, NULL);
blkid_probe_all(cache);
blkid_dev dev = blkid_get_dev(cache, device_path, BLKID_DEV_NORMAL);
const char *type = blkid_get_tag_value(cache, "TYPE", device_path);
```

#### Updater (升级服务)

```gn
# updater/BUILD.gn (示例)
ohos_executable("updater") {
  deps = [
    "//third_party/e2fsprogs:e2fsck",
    "//third_party/e2fsprogs:resize2fs",
  ]
}
```

**使用场景**:
- Recovery 模式下检查和修复文件系统
- OTA 升级时调整分区大小

**典型流程**:
```bash
# 1. 检查文件系统
e2fsck -y /dev/block/platform/.../system

# 2. 调整分区大小
resize2fs /dev/block/platform/.../system ${NEW_SIZE}K
```

#### 镜像构建系统

**使用场景**:
- 构建 system.img
- 构建 vendor.img
- 构建 userdata.img

**典型命令**:
```bash
e2fsdroid -T -1 \
    -C fs_config.txt \
    -S file_contexts \
    -s shared_library \
    -a system \
    -f system/ \
    -L system \
    -D system \
    -b 4096 \
    system.img
```

## 4.3 典型使用场景

### 场景一：存储设备热插拔

```
用户插入 U 盘
    │
    ▼
┌─────────────────┐
│ Storage Service │◀── 1. 接收到 uevent
│   (storage_daemon) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ libblkid.so     │◀── 2. 调用 blkid 识别分区
│                 │     - 读取超级块 magic
│                 │     - 识别文件系统类型
│                 │     - 获取 UUID/LABEL
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 挂载服务         │◀── 3. 根据类型选择挂载参数
│                 │     - VFAT: uid=system,gid=system
│                 │     - ext4: 默认参数
│                 │     - NTFS: 使用 fuse
└────────┬────────┘
         │
         ▼
    挂载完成，通知 UI
```

**代码示例**:
```cpp
// storage_service 中识别分区的代码片段
#include <blkid/blkid.h>

int IdentifyPartition(const char *device_path, FsInfo &info) {
    blkid_cache cache = nullptr;
    blkid_get_cache(&cache, nullptr);
    blkid_probe_all(cache);
    
    blkid_dev dev = blkid_get_dev(cache, device_path, BLKID_DEV_NORMAL);
    if (!dev) {
        return -1;
    }
    
    info.type = blkid_get_tag_value(cache, "TYPE", device_path);
    info.uuid = blkid_get_tag_value(cache, "UUID", device_path);
    info.label = blkid_get_tag_value(cache, "LABEL", device_path);
    
    blkid_put_cache(cache);
    return 0;
}
```

### 场景二：系统启动文件系统检查

```
内核启动
    │
    ▼
挂载根文件系统 (ro)
    │
    ▼
启动 init 进程
    │
    ▼
┌─────────────────────────┐
│ init.cfg 配置的检查任务  │◀── 1. 检查 fstab 中的 fsck 标志
│                         │
│ fsck /dev/block/.../vendor
│ fsck /dev/block/.../system
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ e2fsck 执行             │◀── 2. 执行文件系统检查
│                         │
│ - 检查超级块一致性      │
│ - 检查 inode 分配       │
│ - 检查目录结构          │
│ - 修复发现的问题        │
└────────┬────────────────┘
         │
         ▼
重新挂载根文件系统 (rw)
    │
    ▼
继续启动流程
```

**init.cfg 配置示例**:
```json
{
    "jobs": [{
        "name": "fsck",
        "cmds": [
            "fsck /dev/block/platform/fe310000.sdhci/by-name/vendor",
            "fsck /dev/block/platform/fe310000.sdhci/by-name/system"
        ]
    }]
}
```

### 场景三：OTA 升级流程

```
下载 OTA 包
    │
    ▼
验证签名
    │
    ▼
重启进入 Recovery
    │
    ▼
┌─────────────────────────┐
│ Updater 执行            │◀── 1. 解析 OTA 包
│                         │
│ - 检查版本兼容性        │
│ - 备份关键数据          │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ resize2fs 执行          │◀── 2. 调整分区大小(如需要)
│                         │
│ resize2fs /dev/.../system ${NEW_SIZE}
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ 应用更新包              │◀── 3. 写入新数据
│                         │
│ - 解压 payload          │
│ - 写入分区              │
│ - 验证 checksum         │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ e2fsck 执行             │◀── 4. 验证文件系统完整性
│                         │
│ e2fsck -n /dev/.../system
└────────┬────────────────┘
         │
         ▼
重启到正常系统
```

### 场景四：镜像制作流程

```
编译系统组件
    │
    ▼
收集文件到临时目录
    │
    ▼
生成 fs_config.txt
    │
    ▼
生成 file_contexts
    │
    ▼
┌─────────────────────────┐
│ e2fsdroid 执行          │◀── 制作 ext4 镜像
│                         │
│ - 创建空镜像文件        │
│ - 从目录填充文件        │
│ - 应用 DAC 配置         │
│ - 应用 SELinux 标签     │
│ - 生成稀疏镜像          │
└────────┬────────────────┘
         │
         ▼
生成 system.img/sparse.img
    │
    ▼
打包到 OTA/刷机包
```

## 4.4 安装位置

### 分区分布

| 工具/库 | system 分区 | updater 分区 | 说明 |
|--------|-------------|--------------|------|
| e2fsck | ✓ | ✓ | 运行时和升级时都需要 |
| mke2fs | ✓ | ✗ | 通常仅在系统运行时需要 |
| resize2fs | ✓ | ✓ | 升级时需要调整分区 |
| blkid | ✓ | ✗ | 运行时识别设备 |
| e2fsdroid | ✓ | ✗ | 仅构建时使用 |
| libext2_* | ✓ | ✓ | 库文件 |

### 安装路径

```
/system/bin/
├── e2fsck
├── mke2fs
├── resize2fs
├── blkid
└── e2fsdroid

/system/lib/
├── libext2_blkid.so
├── libext2_com_err.so
├── libext2_e2p.so
├── libext2fs.so
├── libext2_misc.so
├── libext2_quota.so
└── libext2_uuid.so

/updater/bin/
├── e2fsck
└── resize2fs

/updater/lib/
└── (必要的库文件)
```

## 4.5 性能考虑

### 启动时间影响

| 操作 | 时间消耗 | 优化建议 |
|------|---------|---------|
| e2fsck (干净分区) | ~100ms | 仅在异常时执行完整检查 |
| e2fsck (修复模式) | 数秒到数分钟 | 后台异步执行 |
| blkid 缓存 | ~10ms | 使用缓存避免重复探测 |

### 内存占用

| 组件 | 内存占用 | 说明 |
|------|---------|------|
| libblkid | ~100KB | 缓存结构 |
| e2fsck | 数MB到数十MB | 取决于文件系统大小 |
| resize2fs | 数MB | 临时的 inode 表 |

---

## 下一章

- **[05_API_Differences.md](./05_API_Differences.md)** - API 差异说明
