# 02 Patch 详细分析

> **核心文档**: 本页详细分析 OpenHarmony 对 gptfdisk 的所有修改。

---

## 2.1 Patch 清单概览

### 传统 Patch 文件

| 搜索结果 | 数量 | 状态 |
|---------|------|------|
| `*.patch` 文件 | 0 | 未找到 |
| `patches/` 目录 | 0 | 未找到 |

**结论**: gptfdisk **没有使用传统 Patch 文件**，修改直接在源码中进行。

### 修改汇总表

| 修改位置 | 修改类型 | 影响范围 | OH 特有 |
|---------|---------|---------|---------|
| `sgdisk.cc` | 功能扩展 | 新增 `--ohos-dump` | ✅ 是 |
| `BUILD.gn` | 新增文件 | 构建配置 | ✅ 是 |
| `bundle.json` | 新增文件 | 组件元数据 | ✅ 是 |

---

## 2.2 核心修改: `--ohos-dump` 功能

### 修改位置

**文件**: `third_party/gptfdisk/sgdisk.cc`

### 新增代码 (完整)

```cpp
/*
 * Dump partition details in a machine readable format:
 *
 * DISK[mbr|gpt][guid]
 * PART[n][type][guid]
 */
static int ohos_dump(char* device) {
   BasicMBRData mbrData;
   GPTData gptData;
   GPTPart partData;
   int numParts = 0;

   if (!mbrData.ReadMBRData((string)device)) {
     cerr << "Failed to read MBR" << endl;
     return 8;
   }

   switch(mbrData.GetValidity()) {
     case mbr:
       cout << "DISK mbr" << endl;
       for (int i = 0; i < MAX_MBR_PARTS; i++) {
         if(mbrData.GetLength(i) > 0) {
           cout << "PART " << (i + 1) << " " << hex << (int)mbrData.GetType(i) << dec << endl;
         }
       }
       break;
     case hybrid:
     case gpt:
       gptData.JustLooking();
       if(!gptData.LoadPartitions((string)device)) {
         cerr << "Failed to read GPT" << endl;
         return 9;
       }

       cout << "DISK gpt " << gptData.GetDiskGUID() << endl;
       numParts = gptData.GetNumParts();
       for (int i = 0; i < numParts; i++) {
         partData = gptData[i];
         if (partData.GetFirstLBA() > 0) {
           cout << "PART " << (i + 1) << " " << partData.GetType() << " " << partData.GetUniqueGUID() << " "
               << partData.GetDescription() << endl;
         }
       }
       break;
     default:
       cerr << "Unknown partition table" << endl;
       return 10;
   }

   return 0;
}
```

### main() 函数修改

```cpp
int main(int argc, char *argv[]) {
   // OH 特有: 处理 --ohos-dump 参数
   for (int i = 0; i < argc; i++) {
     if (!strcmp("--ohos-dump", argv[i])) {
       if (i + 1 >= argc) {
        return -1;
       }
       return ohos_dump(argv[i + 1]);
     }
   }
   
   // 原有逻辑
   GPTDataCL theGPT;
   return theGPT.DoOptions(argc, argv);
}
```

---

## 2.3 `--ohos-dump` 详细分析

### 功能描述

**`--ohos-dump` 是一个 OpenHarmony 特有的命令行选项**，用于以机器可读的格式导出磁盘分区表信息。

### 输出格式

#### MBR 磁盘输出

```
DISK mbr
PART 1 0c
PART 2 83
```

**格式说明**:
- `DISK mbr`: 表示 MBR 分区表
- `PART n type`: 分区号 + 十六进制类型码

#### GPT 磁盘输出

```
DISK gpt 12345678-1234-1234-1234-123456789abc
PART 1 EBD0A0A2-B9E5-4433-87C0-68B6B72699C7 11111111-1111-1111-1111-111111111111 EFI System Partition
PART 2 0FC63DAF-8483-4772-8E79-3D69D8477DE4 22222222-2222-2222-2222-222222222222 Linux filesystem
```

**格式说明**:
- `DISK gpt [disk_guid]`: GPT 分区表 + 磁盘 GUID
- `PART n type_guid part_guid description`: 分区号 + 类型 GUID + 分区 GUID + 描述

### 代码逻辑分析

```
┌─────────────────┐
│   ohos_dump()   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ ReadMBRData()   │ ← 读取保护性 MBR
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ GetValidity()   │ ← 判断分区表类型
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌───────┐  ┌────────┐
│  MBR  │  │  GPT   │
└───┬───┘  └───┬────┘
    │          │
    ▼          ▼
┌─────────┐  ┌──────────────┐
│遍历4个  │  │JustLooking() │ ← 只读模式
│主分区   │  │LoadPartitions│
└─────────┘  └──────────────┘
                  │
                  ▼
             ┌─────────┐
             │遍历所有  │
             │GPT分区  │
             └─────────┘
```

### OH 需求关联

**为什么需要这个功能?**

Storage Service 需要：
1. **标准化输出**: 易于解析的格式，而非 sgdisk 原生的多行文本
2. **完整信息**: 同时需要类型 GUID 和分区 GUID
3. **统一接口**: MBR 和 GPT 输出格式一致
4. **机器可读**: 适合 C++ 代码直接解析

**使用代码位置**:

```cpp
// foundation/filemanagement/storage_service/services/storage_daemon/disk/src/disk_info.cpp
constexpr const char *SGDISK_PATH = "/system/bin/sgdisk";
constexpr const char *SGDISK_DUMP_CMD = "--ohos-dump";

// 调用示例:
// sgdisk --ohos-dump /dev/block/sda
```

---

## 2.4 修改分类

### 按性质分类

| 修改 | 类型 | 说明 |
|------|------|------|
| `ohos_dump()` | 功能新增 | 新增导出函数 |
| `--ohos-dump` | 接口扩展 | 新增命令行选项 |

### 按影响分类

| 修改 | 影响 | 风险 |
|------|------|------|
| `ohos_dump()` | 新增代码路径 | 低 (独立函数) |
| `--ohos-dump` | 新增参数处理 | 低 (前置检查) |

### 上游兼容性

| 检查项 | 结果 | 说明 |
|-------|------|------|
| 原有功能 | ✅ 保留 | 未修改原 main 逻辑 |
| 参数冲突 | ❌ 无 | `--ohos-dump` 非上游参数 |
| 返回值 | ✅ 兼容 | 遵循 sgdisk 错误码约定 |

---

## 2.5 回归风险分析

### 升级上游版本时

| 风险点 | 等级 | 缓解措施 |
|-------|------|---------|
| `sgdisk.cc` 重构 | 中 | 对比 main() 函数结构 |
| 类接口变更 | 低 | BasicMBRData/GPTData 已稳定 |
| 错误码变更 | 低 | 使用独立错误码 (8-10) |

### 建议的升级流程

1. **备份当前修改**
   ```bash
   cp sgdisk.cc sgdisk.cc.ohos.bak
   ```

2. **应用上游更新**
   ```bash
   # 合并或替换文件
   ```

3. **重新应用 OH 修改**
   - 添加 `ohos_dump()` 函数
   - 在 `main()` 开头添加 `--ohos-dump` 处理

4. **验证构建**
   ```bash
   # 确保 BUILD.gn 未变更
   ```

5. **功能测试**
   ```bash
   sgdisk --ohos-dump /dev/block/sda
   ```

---

## 2.6 代码质量评估

### 优点

| 方面 | 评价 |
|------|------|
| **封装性** | 独立函数，不影响原有代码 |
| **可读性** | 逻辑清晰，switch-case 结构 |
| **错误处理** | 有错误码返回和错误输出 |
| **资源管理** | 使用栈对象，无内存泄漏风险 |

### 潜在改进

| 方面 | 现状 | 建议 |
|------|------|------|
| **返回值** | 魔法数字 (8,9,10) | 使用枚举常量 |
| **错误信息** | cerr 输出 | 可考虑 syslog |
| **路径处理** | 直接使用 argv | 可添加路径验证 |

---

## 2.7 与上游协作建议

### 是否适合推向上游?

| 评估项 | 结论 | 原因 |
|-------|------|------|
| **通用性** | 中 | 机器可读格式有价值 |
| **复杂性** | 低 | 功能独立，易于维护 |
| **维护成本** | 低 | 代码量少 |

**建议**: 可考虑向上游提交 PR，但需：
1. 改为更通用的选项名 (如 `--machine-readable`)
2. 提供详细的文档和测试用例
3. 等待上游反馈

### 当前策略

**维持 OH 特有修改**，原因：
- 上游接受 PR 需要时间
- 选项名 (`--ohos-dump`) 包含 OH 品牌，不适合通用
- 功能满足当前需求，无需推广

---

## 2.8 Patch 信息汇总

```markdown
### Patch: OHOS-DUMP-FEATURE

**修改文件**: `sgdisk.cc`

**修改摘要**: 
- 新增 `ohos_dump()` 函数，支持机器可读格式导出分区表
- 新增 `--ohos-dump` 命令行选项

**OH 需求**: 
为 Storage Service 提供标准化的分区表信息导出功能，便于 C++ 代码解析。

**关键代码变更**:
```cpp
// 新增函数
static int ohos_dump(char* device);

// main() 前置处理
if (!strcmp("--ohos-dump", argv[i])) {
    return ohos_dump(argv[i + 1]);
}
```

**升级建议**: 
此修改为 OH 特有功能，升级上游版本时需手动合并。建议保留备份文件。
```
