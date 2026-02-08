# 05 API/接口差异

> 分析 OpenHarmony 版本与上游版本的接口差异

---

## 5.1 接口差异概述

### 差异类型

| 类型 | 数量 | 说明 |
|------|------|------|
| **新增接口** | 1 | `--ohos-dump` 命令行选项 |
| **修改接口** | 0 | 无 |
| **废弃接口** | 3 | gdisk, cgdisk, fixparts 未编译 |

### 差异对比表

| 接口 | 上游 | OH 版本 | 差异 |
|------|------|---------|------|
| `sgdisk` 主程序 | ✅ | ✅ | 相同 |
| `--ohos-dump` 选项 | ❌ | ✅ | **OH 特有** |
| `gdisk` 程序 | ✅ | ❌ | 未编译 |
| `cgdisk` 程序 | ✅ | ❌ | 未编译 |
| `fixparts` 程序 | ✅ | ❌ | 未编译 |

---

## 5.2 新增接口: `--ohos-dump`

### 接口定义

**命令格式**:
```bash
sgdisk --ohos-dump <device>
```

**参数**:
| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `--ohos-dump` | 标志 | 是 | 启用 OH 导出模式 |
| `<device>` | 字符串 | 是 | 块设备路径，如 `/dev/block/sda` |

**返回值**:
| 值 | 含义 |
|-----|------|
| 0 | 成功 |
| 8 | 读取 MBR 失败 |
| 9 | 读取 GPT 失败 |
| 10 | 未知分区表类型 |
| -1 | 参数错误 |

### 输出格式

#### MBR 磁盘

```
DISK mbr
PART <n> <type>
```

**示例**:
```
DISK mbr
PART 1 0c
PART 2 83
```

**字段说明**:
| 字段 | 说明 |
|------|------|
| `DISK mbr` | 表示 MBR 分区表 |
| `PART n` | 分区号 (1-4) |
| `type` | 十六进制类型码 |

#### GPT 磁盘

```
DISK gpt <disk_guid>
PART <n> <type_guid> <part_guid> <description>
```

**示例**:
```
DISK gpt 12345678-1234-1234-1234-123456789abc
PART 1 EBD0A0A2-B9E5-4433-87C0-68B6B72699C7 11111111-1111-1111-1111-111111111111 EFI System Partition
```

**字段说明**:
| 字段 | 说明 |
|------|------|
| `DISK gpt` | 表示 GPT 分区表 |
| `disk_guid` | 磁盘 GUID |
| `PART n` | 分区号 (1-128) |
| `type_guid` | 分区类型 GUID |
| `part_guid` | 分区唯一 GUID |
| `description` | 分区描述/名称 |

### 与上游选项对比

| 上游选项 | OH 选项 | 差异 |
|---------|---------|------|
| `-p / --print` | `--ohos-dump` | 输出格式不同 |
| `-i / --info` | `--ohos-dump` | 信息详细程度不同 |

**主要区别**:
- `--print`: 人类可读，多行文本，格式复杂
- `--ohos-dump`: 机器可读，结构化，易于解析

---

## 5.3 废弃接口

### 未编译的程序

由于 OH 仅编译 `sgdisk`，以下程序在 OH 中不可用：

#### gdisk

**上游功能**: 交互式文本模式分区工具

**OH 状态**: ❌ 未编译

**替代方案**: 使用 `sgdisk` 的命令行选项

**示例转换**:
```bash
# 上游: 使用 gdisk 交互式创建分区
gdisk /dev/sda
# (交互式菜单操作)

# OH: 使用 sgdisk 命令行
gdisk --new=1:0:+100M --typecode=1:8300 /dev/sda
```

#### cgdisk

**上游功能**: curses 图形界面分区工具

**OH 状态**: ❌ 未编译

**原因**: 嵌入式系统无图形界面需求

#### fixparts

**上游功能**: MBR 分区修复工具

**OH 状态**: ❌ 未编译

**原因**: OH 使用 GPT 分区，不需要 MBR 修复

---

## 5.4 行为一致的接口

### sgdisk 标准选项

以下 sgdisk 选项在 OH 版本中行为与上游完全一致：

| 选项 | 功能 | OH 支持 |
|------|------|---------|
| `-n / --new` | 创建分区 | ✅ |
| `-d / --delete` | 删除分区 | ✅ |
| `-t / --typecode` | 设置类型码 | ✅ |
| `-c / --change-name` | 修改分区名 | ✅ |
| `-g / --mbrtogpt` | MBR 转 GPT | ✅ |
| `-z / --zap-all` | 清除分区表 | ✅ |
| `-p / --print` | 打印分区表 | ✅ |
| `-i / --info` | 显示分区详情 | ✅ |
| `-v / --verify` | 验证分区表 | ✅ |
| `-h / --help` | 帮助信息 | ✅ |

### 使用示例

#### 创建分区 (与上游相同)

```bash
# 创建 100MB 的 EFI 系统分区
sgdisk --new=1:0:+100M --typecode=1:ef00 /dev/block/sda

# 使用剩余空间创建数据分区
sgdisk --new=2:0:0 --typecode=2:8300 /dev/block/sda
```

#### 删除分区 (与上游相同)

```bash
# 删除分区 1
sgdisk --delete=1 /dev/block/sda
```

#### 查看分区表 (与上游相同)

```bash
# 标准输出 (人类可读)
sgdisk --print /dev/block/sda
```

---

## 5.5 内部 API 差异

### C++ 类接口

内部 C++ 类接口与上游保持一致：

| 类 | 状态 | 说明 |
|-----|------|------|
| `GPTData` | ✅ 一致 | GPT 操作核心类 |
| `GPTPart` | ✅ 一致 | GPT 分区类 |
| `BasicMBRData` | ✅ 一致 | MBR 操作类 |
| `GUIDData` | ✅ 一致 | GUID 处理类 |

### 新增内部函数

**`sgdisk.cc` 新增**:

```cpp
// OH 特有函数
static int ohos_dump(char* device);
```

**函数签名**:
- 输入: 设备路径字符串
- 输出: stdout 打印分区信息
- 返回: 整数状态码

---

## 5.6 迁移指南

### 从上游到 OH

如果您有基于上游 gptfdisk 的脚本，迁移到 OH 时：

#### 场景 1: 使用 sgdisk 脚本

```bash
# 上游脚本 (可直接使用)
sgdisk --zap-all /dev/sda
sgdisk --new=1:0:+500M --typecode=1:ef00 /dev/sda

# OH 兼容: ✅ 无需修改
```

#### 场景 2: 使用 gdisk 交互式

```bash
# 上游 (需要交互)
gdisk /dev/sda

# OH 替代 (命令行)
sgdisk --new=1:0:+500M /dev/sda
```

#### 场景 3: 使用 --ohos-dump

```bash
# OH 特有 (上游不支持)
sgdisk --ohos-dump /dev/block/sda

# 上游替代 (输出格式不同)
sgdisk --print /dev/sda
```

---

## 5.7 版本兼容性矩阵

### 命令兼容性

| 命令/选项 | 上游 1.0.10 | OH 3.1 | 兼容性 |
|-----------|-------------|--------|--------|
| sgdisk --new | ✅ | ✅ | 100% |
| sgdisk --delete | ✅ | ✅ | 100% |
| sgdisk --zap-all | ✅ | ✅ | 100% |
| sgdisk --print | ✅ | ✅ | 100% |
| sgdisk --ohos-dump | ❌ | ✅ | OH 特有 |
| gdisk | ✅ | ❌ | 不可用 |
| cgdisk | ✅ | ❌ | 不可用 |
| fixparts | ✅ | ❌ | 不可用 |

---

## 5.8 文档约定

### 命令行语法

本文档使用以下约定：

| 符号 | 含义 |
|------|------|
| `<arg>` | 必需参数 |
| `[arg]` | 可选参数 |
| `...` | 可重复 |
| `|` | 或 |

### 示例设备名

| 名称 | 说明 |
|------|------|
| `/dev/sda` | Linux 标准设备名 (上游示例) |
| `/dev/block/sda` | OH 设备路径 |
| `/dev/block/disk-8-0` | OH 动态设备节点 |
