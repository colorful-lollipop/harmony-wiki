# 02_Patches.md - Patch 分析

## Patch 文件清单

### 分析结果

**本库无任何 Patch 文件**

```bash
# 搜索结果
$ find . -name "*.patch" -o -name "patches" -type d
# (无输出)

$ git log --all --full-history -- '*.patch'
# (无输出)
```

## 为什么没有 Patch？

### 原因 1：纯头文件库

OpenMAX IL 是一个**接口定义库**，只包含 C 头文件：

```
api/1.1.2/
├── OMX_*.h        # 16 个标准头文件
├── codec_omx_ext.h # OH 扩展头文件（新增，非 Patch）
└── *.zip          # 原始分发包
```

- 无 `.c`/`.cpp` 源文件需要修改
- 无逻辑代码需要修复
- 只有宏定义、结构体、枚举类型

### 原因 2：标准接口应保持兼容

OpenMAX IL 是 Khronos Group 制定的**行业标准**：

| 考虑因素 | 说明 |
|---------|------|
| **兼容性** | 修改标准头文件会破坏与芯片厂商 OMX 实现的兼容性 |
| **生态** | 主流芯片厂商（Rockchip、HiSilicon、Qualcomm）均按标准实现 |
| **升级** | 保持原样便于跟随上游版本升级 |

### 原因 3：OH 采用扩展机制

OH 通过独立的扩展文件添加功能，而非修改原文件：

```c
// 不推荐：修改上游头文件 ❌
// OMX_Video.h (修改)
#define OMX_VIDEO_CodingHEVC 0x7F000001  // 新增 HEVC 支持

// 推荐：扩展文件 ✅
// codec_omx_ext.h (新增)
enum CodecVideoExType {
    CODEC_OMX_VIDEO_CodingHEVC = 11,
    CODEC_OMX_VIDEO_CodingVVC = 0x7F000007,
};
```

**扩展文件优势**：
- 与上游完全隔离
- 版本升级无冲突
- 变更历史清晰

## 维护策略

### 无 Patch 意味着

| 方面 | 影响 |
|------|------|
| **升级成本** | 极低 - 直接替换头文件即可 |
| **维护负担** | 极低 - 无 Patch 需要同步 |
| **安全风险** | 低 - 接口定义无运行时逻辑漏洞 |
| **兼容风险** | 低 - 与上游保持 100% 兼容 |

### 升级上游版本流程

假设要从 1.1.2 升级到 1.2.0：

```bash
# 1. 备份当前版本
mv api/1.1.2 api/1.1.2.bak

# 2. 放置新版本
mv openmax_il_1_2_0 api/1.2.0

# 3. 保留扩展文件
cp api/1.1.2.bak/codec_omx_ext.h api/1.2.0/

# 4. 更新 BUILD.gn 中的 include 路径
# include_dirs = [ "api/1.2.0" ]

# 5. 检查 codec_omx_ext.h 兼容性
# 可能需要更新扩展定义以适配新版本类型
```

### 需要避免的修改

以下操作**禁止**执行：

```diff
# ❌ 不要修改标准头文件中的定义
- #define OMX_VERSION_MAJOR 1
+ #define OMX_VERSION_MAJOR 2

# ❌ 不要删除或重命名已有枚举
- OMX_CommandStateSet,
+ OMX_CommandStateSet_DEPRECATED,

# ❌ 不要改变结构体布局
  struct OMX_BUFFERHEADERTYPE {
-     OMX_U32 nSize;
      OMX_VERSIONTYPE nVersion;
+     OMX_U32 nSize;
  };
```

### 正确的扩展方式

```c
// ✅ codec_omx_ext.h - 独立扩展文件
#ifndef CODEC_OMX_EXT_H
#define CODEC_OMX_EXT_H

#include <OMX_Core.h>
#include <OMX_Video.h>

// 使用 Vendor 保留区域定义扩展
enum OmxIndexCodecExType {
    OMX_IndexExtBufferTypeStartUnused = OMX_IndexKhronosExtensions + 0x00a00000,
    OMX_IndexParamSupportBufferType,
    // ...
};

// 定义新的结构体
struct CodecVideoPortFormatParam {
    uint32_t size;
    union OMX_VERSIONTYPE version;
    // ... 自定义字段
};

#endif
```

## Patch 替代方案对比

| 方案 | 适用场景 | 本库采用情况 |
|------|---------|-------------|
| **Patch 文件** | 修改上游源码 | ❌ 未采用 |
| **独立扩展文件** | 新增功能而不修改原文件 | ✅ `codec_omx_ext.h` |
| **包装层** | 封装标准接口提供 OH 风格 API | ✅ Codec HDI |
| **配置宏** | 条件编译 | ✅ BUILD.gn defines |

## 历史变更分析

通过 `git log` 分析，`codec_omx_ext.h` 演进历史：

```
2025-01: 增加结构体成员，实现上层可配置
2024-12: 新增 codec error type (参数集非法/丢失)
2024-11: 新增无线低时延码控
2024-10: 统一命名风格
2024-09: 新增无线低时延动静状态上报
2024-08: 新增 HEVC/VVC profile/level 扩展
...
```

**观察**：所有变更都集中在扩展文件，未触碰标准头文件。

## 质量检查清单

### Patch 相关检查项

- [x] 确认无 Patch 文件需要分析
- [x] 确认无 Patch 是设计决策而非遗漏
- [x] 确认扩展文件 `codec_omx_ext.h` 机制健全
- [x] 确认升级路径清晰

### 升级风险评估

| 风险项 | 等级 | 说明 |
|--------|------|------|
| Patch 迁移遗漏 | 无 | 无 Patch |
| 扩展 API 冲突 | 低 | 使用 Vendor 保留区域 |
| 上游 API 变更 | 低 | 接口标准稳定 |
| 厂商实现兼容 | 中 | 需确认芯片厂商支持新版本 |

## 结论

OpenMAX IL 库**无需 Patch**，这是由库的特性决定的：

1. **纯接口定义**：无逻辑代码需要修复
2. **行业标准**：保持与 Khronos 标准兼容
3. **扩展机制完善**：通过 `codec_omx_ext.h` 满足 OH 定制化需求

**维护建议**：
- 继续保持无 Patch 策略
- 功能扩展统一添加到 `codec_omx_ext.h`
- 版本升级前检查扩展文件兼容性
