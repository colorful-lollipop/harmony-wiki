# 02 - Patch 详细分析

## 2.1 Patch 清单表

| Patch 文件 | 修改文件 | 修改类型 | OH 需求 | 优先级 |
|-----------|---------|---------|--------|-------|
| 1001-image-make.patch | populate-extfs.sh | 功能增强 | 镜像制作配置文件支持 | 高 |
| 1002-add-header-file-to-musl-compile-mk2efs.patch | 5+ 个头文件 | 编译适配 | musl C 库适配 | 高 |
| 1003-add-dac-config.patch | dac_config.cpp/h, perms.c/h | 功能增强 | 自定义 DAC 配置机制 | 高 |
| 1004-modify-code-to-compile.patch | 3 个源文件 | 编译适配 | 编译兼容性修复 | 中 |
| 1005-read-vfat-chinese-label.patch | blkid.c | 功能增强 | 中文卷标支持 | 中 |
| 1006-add-hmfs-for-blkid.patch | probe.c/h | 功能增强 | HMFS 文件系统识别 | 中 |
| 1007-blkid-support-skip-specified-filesystem.patch | probe.c, blkidP.h, blkid.c | 功能增强 | 跳过指定文件系统检查 | 低 |
| 1008-blkid-enlarge-cluster-for-ntfs.patch | probe.c | Bugfix | NTFS 大簇支持 | 中 |

## 2.2 详细 Patch 分析

### Patch 1001: image-make.patch

**基本信息**
- **修改文件**: `contrib/populate-extfs.sh`
- **修改大小**: ~80 行
- **修改类型**: 功能增强

**原始问题**
原生的 populate-extfs.sh 脚本不支持从配置文件读取文件权限和属主信息，无法满足 OpenHarmony 镜像制作对精细化权限控制的需求。

**修改内容**

1. **增加配置文件参数支持**
```bash
# 修改前
Usage: populate-extfs.sh <source> <device>

# 修改后
Usage: populate-extfs.sh <source> <device> <cfgfile>
```

2. **新增配置解析函数**
```bash
declare -a CONFIG_ARRAY

# 解析 CSV 格式配置文件
do_parsecfgfile () {
    while read _BINFILE_ _FILETYPE_ _FILEMODE_ _FILEUID_ _FILEGID_
    do
        CONFIG_ARRAY[${i}]=${_BINFILE_}
        # ... 存储到数组
    done < ${CFGFILE}
}
```

3. **配置文件格式**
```
# 格式: path,type,mode,uid,gid
/system/bin/init,f,755,0,0
/system/etc/passwd,f,644,0,0
/system/app,d,755,0,0
```

4. **文件权限查询函数**
```bash
do_getfilecfgmode () {
    # 从 CONFIG_ARRAY 查找匹配的文件配置
    # 支持精确匹配和通配符匹配
}
```

**OH 需求**
- 支持从 CSV 格式配置文件读取权限
- 支持模式、UID、GID 三要素配置
- 保持与 Android 的 populate-extfs.sh 接口兼容

**回归风险**
- **风险等级**: 中
- **风险点**: populate-extfs.sh 脚本变更可能影响镜像制作流程
- **缓解措施**: 
  - 升级时检查脚本接口变更
  - 验证配置文件格式兼容性

---

### Patch 1002: add-header-file-to-musl-compile-mk2efs.patch

**基本信息**
- **修改文件**: 
  - `lib/blkid/blkid.h` (新增, 110 行)
  - `lib/blkid/blkid_types.h` (新增, 174 行)
  - `lib/config.h` (新增, 906 行)
  - `lib/dirpaths.h` (新增, 10 行)
  - `lib/ext2fs/crc32c_table.h` (新增, 1044 行)
- **修改大小**: ~2200 行
- **修改类型**: 编译适配

**原始问题**
OpenHarmony 使用 musl C 库而非 glibc，缺少编译 e2fsprogs 所需的头文件和配置定义。

**修改内容**

1. **blkid.h** - blkid 库公共头文件
```c
#ifndef _BLKID_BLKID_H
#define _BLKID_BLKID_H

#include <sys/types.h>
#include <blkid/blkid_types.h>

// 版本信息
#define BLKID_VERSION	"1.0.0"
#define BLKID_DATE	"12-Feb-2003"

// 类型定义
typedef struct blkid_struct_dev *blkid_dev;
typedef struct blkid_struct_cache *blkid_cache;
typedef __s64 blkid_loff_t;

// API 声明 (cache, dev, probe, tag 等)
...
#endif
```

2. **blkid_types.h** - 类型定义头文件
```c
#ifndef _BLKID_TYPES_H
#define _BLKID_TYPES_H

// 内核类型定义
#ifndef HAVE___U8
typedef unsigned char __u8;
#endif

#ifndef HAVE___U16
typedef unsigned short __u16;
#endif

#ifndef HAVE___U32
typedef unsigned int __u32;
#endif

#ifndef HAVE___U64
typedef unsigned long long __u64;
#endif

// 以及 __s8, __s16, __s32, __s64
...
#endif
```

3. **config.h** - 编译配置
```c
// 功能开关
#define HAVE_ALLOCA_H 1
#define HAVE_FALLOCATE 1
#define HAVE_EXT2_IOCTLS 1

// 路径配置
#include <dirpaths.h>

// 线程支持
#define USE_POSIX_THREADS 1

// 大量 HAVE_* 宏定义...
```

4. **dirpaths.h** - 路径配置
```c
#define LOCALEDIR "/path/to/locale"
#define ROOT_SYSCONFDIR "/path/to/etc"
```

5. **crc32c_table.h** - CRC32C 查找表
```c
// 预计算的 CRC32C 查找表
static const uint32_t crc32table_be[8][256] = {{...}};
```

**OH 需求**
- 提供 musl 环境编译所需的所有头文件
- 定义适当的类型和配置宏
- 确保与 OpenHarmony 内核版本兼容

**回归风险**
- **风险等级**: 低
- **风险点**: 上游版本升级时配置可能变化
- **缓解措施**: 
  - 升级时对比上游 config.h.in
  - 使用 configure 脚本重新生成

---

### Patch 1003: add-dac-config.patch

**基本信息**
- **修改文件**:
  - `contrib/android/dac_config.cpp` (新增, 246 行)
  - `contrib/android/dac_config.h` (新增, 32 行)
  - `contrib/android/perms.c`
  - `contrib/android/perms.h`
- **修改类型**: 功能增强 (核心 Patch)

**原始问题**
Android 使用 `fs_config` 和 `canned_fs_config` 机制配置文件权限，依赖于 Android 特有的头文件 (`android_filesystem_config.h`)。OpenHarmony 需要独立的权限配置机制。

**修改内容**

1. **dac_config.h** - 头文件
```c
#ifndef __DAC_CONFIG
#define __DAC_CONFIG
#include <stdint.h>

#ifdef __cplusplus
extern "C" {
#endif

int LoadDacConfig(const char* fn);
void GetDacConfig(const char* path, int dir, char* targetOutPath,
        unsigned* uid, unsigned* gid, unsigned* mode,
        uint64_t* capabilities);

#ifdef __cplusplus
}
#endif
#endif
```

2. **dac_config.cpp** - DAC 配置解析
```cpp
#include <unordered_map>
#include <vector>
#include <linux/capability.h>

namespace {
struct DacConfig {
    unsigned int uid;
    unsigned int gid;
    unsigned int mode;
    uint64_t capabilities;
    string path;
    
    DacConfig() : uid(0), gid(0), mode(0), capabilities(0), path("") {}
};

unordered_map<string, DacConfig> g_configMap;

// 能力名到数值的映射
unordered_map<string, unsigned int> g_capStrCapNum = {
    { "CAP_CHOWN", CAP_CHOWN },
    { "CAP_DAC_OVERRIDE", CAP_DAC_OVERRIDE },
    // ... 更多能力
};
} // namespace

extern "C" {
    int LoadDacConfig(const char* fn) {
        // 读取 CSV 格式配置文件
        // 格式: path,mode,uid,gid,capabilities
        // 解析并存入 g_configMap
    }
    
    void GetDacConfig(const char* path, int dir, char* targetOutPath,
            unsigned* uid, unsigned* gid, unsigned* mode,
            uint64_t* capabilities) {
        // 查找文件路径对应的配置
        // 支持精确匹配和通配符匹配 (e.g., /system/bin/*)
        // 未找到时返回默认值
    }
}
```

3. **perms.c/h 修改** - 条件编译
```c
// perms.h
#if defined(__ANDROID__)
#  include <private/android_filesystem_config.h>
#  include <private/canned_fs_config.h>
#  include <private/fs_config.h>
#else /* !__ANDROID__ */
#  include "dac_config.h"
#endif

// perms.c
if (fs_config_file) {
#if defined(__ANDROID__)
    retval = load_canned_fs_config(fs_config_file);
    fs_config_func = canned_fs_config;
#else
    retval = LoadDacConfig(fs_config_file);
    fs_config_func = GetDacConfig;
#endif
}
```

**OH 需求**
- 完全替代 Android 的 fs_config 机制
- 使用 CSV 格式配置文件（人类可读）
- 支持 Linux capabilities
- 支持通配符匹配

**配置文件格式示例**
```csv
# 路径,模式,UID,GID,能力
/system/bin/init,0755,0,0,CAP_SYS_ADMIN|CAP_SYS_BOOT
/system/etc/hosts,0644,0,0,0
/system/app/*,0755,0,0,0
```

**回归风险**
- **风险等级**: 高
- **风险点**: 
  - 这是核心功能 Patch，与镜像制作强相关
  - 权限配置错误会导致系统无法启动
  - 配置文件格式变化需要同步修改
- **缓解措施**:
  - 升级时必须完整测试镜像制作流程
  - 验证权限配置解析的正确性
  - 保持配置文件格式向后兼容

**测试建议**
```bash
# 测试 DAC 配置加载
e2fsdroid -C fs_config.txt -f system/ -L system system.img

# 验证镜像中的权限
debugfs system.img -R 'stat /system/bin/init'
```

---

### Patch 1004: modify-code-to-compile.patch

**基本信息**
- **修改文件**: 
  - `contrib/android/block_range.c`
  - `contrib/android/e2fsdroid.c`
  - `contrib/android/perms.c`
- **修改类型**: 编译适配

**原始问题**
编译时出现警告和错误：
1. `_GNU_SOURCE` 宏重复定义警告
2. 缺少 `linux/capability.h` 头文件

**修改内容**

1. **_GNU_SOURCE 宏保护**
```c
// block_range.c, e2fsdroid.c
#ifndef _GNU_SOURCE
#define _GNU_SOURCE
#endif
```

2. **添加 capability 头文件**
```c
// perms.c
#include <linux/capability.h>
```

**OH 需求**
- 消除编译警告
- 确保在 OpenHarmony 编译环境下无错误

**回归风险**
- **风险等级**: 低
- **风险点**: 几乎无风险
- **升级建议**: 可尝试推向上游

---

### Patch 1005: read-vfat-chinese-label.patch

**基本信息**
- **修改文件**: `misc/blkid.c`
- **修改类型**: 功能增强

**原始问题**
Windows 格式化的 VFAT 分区使用 GBK/GB2312 编码存储中文卷标，直接读取会显示乱码。

**修改内容**

1. **添加 UTF-8 检测函数**
```c
static int is_str_utf8(const char* str) {
    unsigned int nBytes = 0;
    unsigned char chr = *str;
    int bAllAscii = 1;
    
    for (unsigned int i = 0; str[i] != '\0'; ++i) {
        chr = *(str + i);
        if ((chr & 0x80) != 0) bAllAscii = 0;
        if(nBytes == 0 && ((chr & 0x80) != 0)) {
            // 计算多字节字符长度
            while((chr & 0x80) != 0) {
                chr <<= 1;
                nBytes++;
            }
            if((nBytes < 2) || (nBytes > 6)) return 0;
            nBytes--;
        } else if (nBytes != 0) {
            if((chr & 0xc0) != 0x80) return 0;
            nBytes--;
        }
    }
    return (nBytes == 0 || bAllAscii);
}
```

2. **添加编码转换函数**
```c
static int code_convert(char *from_charset, char *to_charset,
                       char *inbuf, size_t inlen, 
                       char *outbuf, size_t outlen) {
    iconv_t cd = iconv_open(to_charset, from_charset);
    if (cd == 0) return -1;
    if (memset_s(outbuf, outlen, 0, outlen) != EOK) return -1;
    if (iconv(cd, &inbuf, &inlen, &outbuf, &outlen) == (size_t)-1) return -1;
    iconv_close(cd);
    return 0;
}
```

3. **修改卷标输出逻辑**
```c
static void print_tags(blkid_dev dev, char *show[], int numtag, int output) {
    // ...
    if (output & OUTPUT_VALUE_ONLY) {
        // 针对 VFAT 文件系统的中文卷标转码
        if (!strncmp(type, "LABEL", 5) && 
            !strncmp(dev->bid_type, "vfat", 4) && 
            !is_str_utf8(value)) {
            char outbuf[255];
            int res = code_convert("gbk", "utf-8", 
                                  (char *)value, strlen(value), 
                                  outbuf, 255);
            if (!res) {
                fputs(outbuf, stdout);
            } else {
                fputs(value, stdout);
            }
        } else {
            fputs(value, stdout);
        }
    }
    // ...
}
```

**OH 需求**
- 正确显示 Windows 格式化的 VFAT 分区中文卷标
- 自动检测并转换 GBK 编码到 UTF-8

**回归风险**
- **风险等级**: 低
- **风险点**: 
  - iconv 库依赖
  - 转码失败时的 fallback 逻辑
- **缓解措施**: 
  - 转码失败时输出原始值
  - 使用 securec 安全函数

---

### Patch 1006: add-hmfs-for-blkid.patch

**基本信息**
- **修改文件**: 
  - `lib/blkid/probe.c`
  - `lib/blkid/probe.h`
- **修改类型**: 功能增强

**原始问题**
blkid 无法识别华为移动文件系统 (HMFS)。

**修改内容**

1. **probe.h 中添加 HMFS 超级块定义**
```c
// HMFS 与 F2FS 结构相同
typedef struct f2fs_super_block hmfs_super_block;
```

2. **probe.c 中添加 HMFS 探测函数**
```c
static int probe_hmfs(struct blkid_probe *probe,
                      struct blkid_magic *id __BLKID_ATTR((unused)),
                      unsigned char *buf) {
    hmfs_super_block *bs = NULL;
    if (buf == NULL) return 1;
    
    bs = (hmfs_super_block *)buf;
    set_uuid(probe->dev, bs->uuid, 0);
    
    if (bs->volume_name[0] != 0) {
        unsigned char vol_name_utf8[513] = {0};
        unicode_16le_to_utf8(vol_name_utf8, 512, 
                            (const unsigned char*)bs->volume_name, 
                            512 * sizeof(__u16));
        blkid_set_tag(probe->dev, "LABEL", vol_name_utf8, 512);
    }
    return 0;
}
```

3. **注册 HMFS 到类型数组**
```c
static struct blkid_magic type_array[] = {
    // ...
    { "f2fs", 1, 0, 4, "\x10\x20\xf5\xf2", probe_f2fs },
    { "hmfs", 1, 0, 4, "\x24\x20\xf5\xfe", probe_hmfs },  // 新增
    // ...
};
```

**HMFS Magic Number**
- HMFS: `\x24\x20\xf5\xfe` (0xFE524020 小端)
- F2FS: `\x10\x20\xf5\xf2` (0xF2520010 小端)

**OH 需求**
- 支持识别华为移动文件系统
- 读取 HMFS 的 UUID 和卷标

**回归风险**
- **风险等级**: 低
- **风险点**: 几乎无风险
- **说明**: 纯新增功能，不影响现有功能

---

### Patch 1007: blkid-support-skip-specified-filesystem.patch

**基本信息**
- **修改文件**:
  - `lib/blkid/blkidP.h`
  - `lib/blkid/probe.c`
  - `misc/blkid.c`
- **修改类型**: 功能增强

**原始问题**
某些场景下需要跳过特定文件系统类型的检查（如避免误判 RAID 设备）。

**修改内容**

1. **blkidP.h - 添加全局变量声明**
```c
extern const char *g_no_check_fs;
```

2. **probe.c - 实现跳过逻辑**
```c
const char *g_no_check_fs = "unknown";

// 在探测函数中
for (id = type_array; id->bim_type; id++) {
    // 跳过指定的文件系统类型
    if (!strcmp(g_no_check_fs, id->bim_type))
        continue;
    // ...
}

// 对 mdraid 特殊处理
if (strcmp(g_no_check_fs, "mdraid") && check_mdraid(probe.fd, uuid) == 0) {
    // ...
}
```

3. **blkid.c - 添加命令行选项**
```c
static void usage(int error) {
    // ...
    "\t-n\tskip specified filesystem check\n"
    // ...
}

int main(int argc, char **argv) {
    // ...
    while ((c = getopt (argc, argv, "c:f:ghlLo:s:t:w:n:v")) != EOF)
        switch (c) {
        // ...
        case 'n':
            g_no_check_fs = optarg;
            break;
        // ...
        }
}
```

**使用示例**
```bash
# 跳过 mdraid 文件系统检查
blkid -n mdraid /dev/sda

# 跳过 vfat 检查
blkid -n vfat /dev/sdb1
```

**OH 需求**
- 提供更灵活的文件系统探测控制
- 解决特定存储设备的识别问题

**回归风险**
- **风险等级**: 低
- **风险点**: 
  - 默认值 "unknown" 确保向后兼容
  - 不影响正常使用

---

### Patch 1008: blkid-enlarge-cluster-for-ntfs.patch

**基本信息**
- **修改文件**: `lib/blkid/probe.c`
- **修改类型**: Bugfix

**原始问题**
NTFS 文件系统支持大于 128 扇区的簇大小，原代码无法正确处理这种大簇格式。

**修改内容**

1. **扩展 sectors_per_cluster 计算逻辑**
```c
static int probe_ntfs(struct blkid_probe *probe, ...) {
    // ...
    bytes_per_sector = ns->bios_parameter_block[0] +
            (ns->bios_parameter_block[1] << 8);
    
    // 原代码: 直接使用 bpb[2] 作为簇大小
    // sectors_per_cluster = ns->bios_parameter_block[2];
    
    // 新代码: 处理大簇格式
    switch (ns->bios_parameter_block[2]) {
        case 1:
        case 2:
        case 4:
        case 8:
        case 16:
        case 32:
        case 64:
        case 128:
            sectors_per_cluster = ns->bios_parameter_block[2];
            break;
        default:
            // 0xF0-0xF9 表示大簇: cluster = 2^(256 - value)
            if ((ns->bios_parameter_block[2] < 240) ||
                (ns->bios_parameter_block[2] > 249))
                return 1;
            sectors_per_cluster = 1 << (256 - ns->bios_parameter_block[2]);
    }
    // ...
}
```

**NTFS 簇大小编码**
- 1-128: 直接作为簇大小（扇区数）
- 240-249 (0xF0-0xF9): 大簇格式
  - 0xF0 (240): 2^16 = 65536 扇区
  - 0xF1 (241): 2^15 = 32768 扇区
  - ...
  - 0xF9 (249): 2^7 = 128 扇区

**OH 需求**
- 正确识别使用大簇格式的 NTFS 分区
- 兼容 Windows 10/11 创建的新型 NTFS 分区

**回归风险**
- **风险等级**: 低
- **风险点**: 几乎无风险
- **升级建议**: 可尝试推向上游

---

## 2.3 Patch 维护建议

### 推向上游的可能性评估

| Patch | 推向上游可能性 | 理由 |
|-------|--------------|------|
| 1001 | 低 | OH 特有的镜像制作流程 |
| 1002 | 中 | musl 支持可能有社区需求 |
| 1003 | 低 | OH 特有的 DAC 机制 |
| 1004 | 高 | 通用编译修复 |
| 1005 | 中 | 本地化功能，社区可能接受 |
| 1006 | 低 | HMFS 是华为特有文件系统 |
| 1007 | 中 | 通用功能增强 |
| 1008 | 高 | 标准 Bugfix |

### 升级上游版本时的检查清单

#### 步骤 1: 应用编译适配 Patch
- [ ] 1002 - 检查 config.h 是否需要更新
- [ ] 1004 - 验证编译是否通过

#### 步骤 2: 应用功能增强 Patch
- [ ] 1001 - 检查 populate-extfs.sh 是否有冲突
- [ ] 1003 - **重点测试 DAC 配置功能**
- [ ] 1005 - 验证中文卷标转码
- [ ] 1006 - 验证 HMFS 识别
- [ ] 1007 - 验证跳过 FS 功能

#### 步骤 3: 应用 Bugfix Patch
- [ ] 1008 - 检查是否已在上游修复

#### 步骤 4: 回归测试
- [ ] 镜像制作流程
- [ ] blkid 识别功能
- [ ] 文件系统检查

### Patch 自动化维护建议

```bash
#!/bin/bash
# patch-check.sh - Patch 健康检查脚本

E2FSPROGS_VERSION="1.47.2"

echo "=== e2fsprogs Patch 检查 ==="

# 1. 检查 Patch 文件存在
echo "[1/4] 检查 Patch 文件..."
for patch in 100{1..8}-*.patch; do
    if [ -f "$patch" ]; then
        echo "  ✓ $patch"
    else
        echo "  ✗ $patch 缺失!"
    fi
done

# 2. 验证 Patch 格式
echo "[2/4] 验证 Patch 格式..."
for patch in *.patch; do
    if patch -p1 --dry-run < "$patch" 2>/dev/null; then
        echo "  ✓ $patch 格式正确"
    else
        echo "  ✗ $patch 需要更新"
    fi
done

# 3. 检查上游 CVE
echo "[3/4] 检查安全更新..."
echo "  TODO: 查询 CVE 数据库"

# 4. 功能测试
echo "[4/4] 运行功能测试..."
echo "  TODO: 运行测试套件"

echo "=== 检查完成 ==="
```

---

## 下一章

- **[03_Build_Integration.md](./03_Build_Integration.md)** - BUILD.gn 构建适配详解
