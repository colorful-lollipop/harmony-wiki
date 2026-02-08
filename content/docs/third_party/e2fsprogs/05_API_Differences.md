# 05 - API 差异说明

## 5.1 概述

OpenHarmony 对 e2fsprogs 的 API 修改相对保守，主要通过 **新增功能** 而非修改现有接口来实现适配。

### 修改分类

| 类型 | 数量 | 说明 |
|------|------|------|
| 新增 API | 3 个 | DAC 配置相关 |
| 新增功能开关 | 2 个 | blkid 跳过 FS、NTFS 大簇 |
| 行为修改 | 1 个 | VFAT 中文卷标编码转换 |
| 兼容宏 | 若干 | `_GNU_SOURCE` 保护等 |

## 5.2 新增 API

### DAC 配置 API

#### LoadDacConfig

```c
int LoadDacConfig(const char* fn);
```

**功能**: 从 CSV 文件加载 DAC 配置

**参数**:
- `fn`: 配置文件路径

**返回值**:
- `0`: 成功
- `-1`: 失败

**配置文件格式**:
```csv
# 路径,模式(oct),UID,GID,Capabilities(可选)
/system/bin/init,0755,0,0,CAP_SYS_ADMIN|CAP_SYS_BOOT
/system/etc/hosts,0644,0,0,0
/system/app/*,0755,0,0,0
```

**使用示例**:
```cpp
#include "dac_config.h"

int main() {
    int ret = LoadDacConfig("/path/to/fs_config.txt");
    if (ret != 0) {
        fprintf(stderr, "Failed to load DAC config\n");
        return -1;
    }
    // ...
}
```

#### GetDacConfig

```c
void GetDacConfig(const char* path, int dir, char* targetOutPath,
                  unsigned* uid, unsigned* gid, unsigned* mode,
                  uint64_t* capabilities);
```

**功能**: 获取指定路径的 DAC 配置

**参数**:
- `path`: 文件路径
- `dir`: 是否为目录 (1=是, 0=否)
- `targetOutPath`: 保留参数，传递 NULL
- `uid`: 输出 UID
- `gid`: 输出 GID
- `mode`: 输出文件模式
- `capabilities`: 输出 Linux capabilities

**使用示例**:
```cpp
unsigned uid, gid, mode;
uint64_t caps;

GetDacConfig("/system/bin/init", 0, nullptr, 
             &uid, &gid, &mode, &caps);

printf("uid=%d, gid=%d, mode=0%o, caps=0x%lx\n",
       uid, gid, mode, caps);
```

### blkid 增强 API

#### 全局变量: g_no_check_fs

```c
extern const char *g_no_check_fs;
```

**功能**: 指定要跳过的文件系统类型

**默认值**: `"unknown"` (不跳过任何类型)

**使用方式**:
```c
#include <blkid/blkid.h>
#include <blkidP.h>

// 跳过 mdraid 检查
g_no_check_fs = "mdraid";
blkid_probe_all(cache);

// 恢复默认
g_no_check_fs = "unknown";
```

**命令行工具**:
```bash
blkid -n mdraid /dev/sda
```

## 5.3 行为差异

### blkid: VFAT 中文卷标

**标准行为**:
- 直接输出卷标字符串
- 可能导致 GBK 编码中文显示为乱码

**OH 行为**:
- 自动检测卷标编码
- GBK 编码自动转换为 UTF-8
- 转码失败时保留原始值

**实现位置**: `misc/blkid.c`

**相关函数**:
```c
// 内部函数，不对外暴露
static int is_str_utf8(const char* str);
static int code_convert(char *from_charset, char *to_charset,
                        char *inbuf, size_t inlen, 
                        char *outbuf, size_t outlen);
```

### populate-extfs.sh: 配置文件支持

**标准参数**:
```bash
populate-extfs.sh <source> <device>
```

**OH 参数**:
```bash
populate-extfs.sh <source> <device> <cfgfile>
```

**差异**:
- 增加第三个参数：配置文件路径
- 配置文件格式为 CSV
- 支持 mode/uid/gid 配置

## 5.4 文件系统支持差异

### 新增支持: HMFS

| 特性 | 上游 | OpenHarmony |
|------|------|-------------|
| **识别支持** | ❌ | ✅ |
| **probe 函数** | 无 | `probe_hmfs()` |
| **Magic** | - | `\x24\x20\xf5\xfe` |
| **UUID 读取** | - | ✅ |
| **卷标读取** | - | ✅ |

### NTFS: 大簇支持

| 特性 | 上游 | OpenHarmony |
|------|------|-------------|
| **标准簇大小** | ✅ | ✅ |
| **大簇格式 (0xF0-0xF9)** | ❌ | ✅ |

## 5.5 编译时差异

### 宏定义差异

| 宏 | 上游 | OpenHarmony | 说明 |
|---|------|-------------|------|
| `_GNU_SOURCE` | 直接定义 | 条件定义 | 避免重复定义警告 |
| `secure_getenv` | 使用原生 | 定义为 `getenv` | musl 兼容性 |
| `HAVE_SYS_TYPES_H` | 由 configure 定义 | 强制定义 | 确保类型定义 |

### 头文件差异

| 头文件 | 上游 | OpenHarmony | 说明 |
|--------|------|-------------|------|
| `config.h` | 由 configure 生成 | Patch 1002 预定义 | musl 适配 |
| `blkid.h` | 安装时生成 | Patch 1002 预定义 | 编译适配 |
| `blkid_types.h` | 安装时生成 | Patch 1002 预定义 | 类型定义 |

## 5.6 API 兼容性说明

### 向后兼容性

✅ **完全向后兼容**

- 所有标准 API 保持不变
- 新增 API 不影响现有代码
- 行为修改仅针对特定场景（VFAT 中文卷标）

### 与上游代码的兼容性

| 场景 | 兼容性 | 说明 |
|------|--------|------|
| 使用标准 libblkid API | ✅ 完全兼容 | 无差异 |
| 使用标准 libext2fs API | ✅ 完全兼容 | 无差异 |
| 使用标准工具命令行 | ✅ 兼容 | 增加可选参数 |
| 使用 e2fsdroid | ⚠️ OH 特有 | 仅存在于 OH |
| 使用 DAC 配置 | ⚠️ OH 特有 | 仅存在于 OH |

## 5.7 迁移指南

### 从上游代码迁移到 OH

#### 情况一：仅使用标准 API

**无需修改**，代码完全兼容。

#### 情况二：使用 DAC 配置

```cpp
// 原 Android 代码
#include <private/canned_fs_config.h>
load_canned_fs_config("/path/to/config");
canned_fs_config(path, dir, target_out, &uid, &gid, &mode, &capabilities);

// OpenHarmony 代码
#include "dac_config.h"
LoadDacConfig("/path/to/config");
GetDacConfig(path, dir, target_out, &uid, &gid, &mode, &capabilities);
```

#### 情况三：使用 blkid 跳过功能

```cpp
// 新增功能，上游代码无此功能
#include <blkid/blkid.h>
#include <blkidP.h>

// 跳过指定文件系统类型检查
g_no_check_fs = "mdraid";
```

### 从 OH 迁移到上游

#### 情况一：使用 DAC 配置

**需要修改**:
```cpp
// 移除 DAC 配置依赖
// 改用标准权限设置或实现自定义机制
```

#### 情况二：使用 HMFS 识别

**需要修改**:
```cpp
// 在上游代码中添加 HMFS 支持
// 参考 Patch 1006 的实现
```

## 5.8 API 使用最佳实践

### DAC 配置使用建议

```cpp
// 1. 配置文件路径使用绝对路径
LoadDacConfig("/system/etc/fs_config.txt");

// 2. 检查返回值
if (LoadDacConfig(config_path) != 0) {
    // 使用默认配置
}

// 3. 默认值处理
unsigned uid = 0, gid = 0, mode = 0644;
uint64_t caps = 0;
GetDacConfig(path, is_dir, nullptr, &uid, &gid, &mode, &caps);
// 如果配置不存在，返回默认值
```

### blkid 使用建议

```cpp
// 1. 使用缓存避免重复探测
blkid_cache cache = nullptr;
blkid_get_cache(&cache, nullptr);
blkid_probe_all(cache);

// 2. 多设备查询共享缓存
for (auto &device : devices) {
    blkid_dev dev = blkid_get_dev(cache, device.c_str(), BLKID_DEV_NORMAL);
    // ...
}

// 3. 释放缓存
blkid_put_cache(cache);
```

---

## 下一章

- **[06_Security.md](./06_Security.md)** - 安全风险分析
