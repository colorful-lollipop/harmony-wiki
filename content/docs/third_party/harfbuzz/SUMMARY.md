# 文档导航与阅读路线

本文档为 HarfBuzz 在 OpenHarmony 中的 Wiki 提供阅读路线建议，帮助不同需求的读者快速找到所需信息。

## 文档结构

```
wiki/
├── README.md              # 项目概述、快速开始
├── SUMMARY.md             # 文档导航（本文档）
├── 01_Overview.md         # 原始库简介
├── 02_Patches.md          # Patch 详细分析
├── 03_Build_Integration.md # 构建适配说明
├── 04_Usage_in_OH.md      # 依赖关系与使用
└── _work/
    ├── ASSESSMENT.md      # 项目评估结果
    ├── NOTES.md           # 分析过程记录
    └── PLAN.md            # 任务进度
```

## 读者路线

### 路线 1：了解概况（5 分钟）

适合：初次接触 HarfBuzz 在 OH 中集成的开发者

**阅读顺序**:
1. `README.md` - 概览和快速开始
2. `01_Overview.md` - 了解 HarfBuzz 基本功能

**重点关注**:
- HarfBuzz 在 OH 系统中的定位
- 主要功能简介
- 快速使用示例

### 路线 2：构建集成（15 分钟）

适合：需要构建或修改 HarfBuzz 的开发者

**阅读顺序**:
1. `README.md` - 概述
2. `03_Build_Integration.md` - 详细构建配置
3. `02_Patches.md` - 了解 OH Patch

**重点关注**:
- BUILD.gn 配置详解
- 不同工具链的支持
- Patch 集成方式
- 构建选项说明

### 路线 3：深入 Patch（30 分钟）

适合：需要维护或升级 Patch 的开发者

**阅读顺序**:
1. `README.md` - 概述
2. `02_Patches.md` - 完整 Patch 分析
3. `03_Build_Integration.md` - 构建集成

**重点关注**:
- 每个 Patch 的修改内容和目的
- ICCARM 适配细节
- COLR 增强功能
- Patch 升级建议

### 路线 4：集成使用（20 分钟）

适合：将 HarfBuzz 集成到新模块的开发者

**阅读顺序**:
1. `README.md` - 概述
2. `04_Usage_in_OH.md` - 详细使用说明
3. `01_Overview.md` - API 了解

**重点关注**:
- 直接依赖者列表
- 集成方式详解
- 头文件引用
- 性能优化建议
- 故障排查

### 路线 5：全面了解（45 分钟）

适合：需要全面掌握 HarfBuzz OH 集成的开发者

**阅读顺序**:
1. `README.md` - 整体概览
2. `01_Overview.md` - 基础信息
3. `02_Patches.md` - 技术细节
4. `03_Build_Integration.md` - 构建配置
5. `04_Usage_in_OH.md` - 实际应用

## 主题索引

### 构建相关

| 主题 | 文档 | 章节 |
|-----|------|-----|
| BUILD.gn 配置 | 03_Build_Integration.md | 全部 |
| 工具链支持 | 03_Build_Integration.md | 工具链适配 |
| Patch 应用 | 03_Build_Integration.md | Patch 集成 |
| 编译选项 | 03_Build_Integration.md | 构建配置 |

### Patch 相关

| 主题 | 文档 | 章节 |
|-----|------|-----|
| Patch 概览 | 02_Patches.md | 第 1 节 |
| ICCARM 适配 | 02_Patches.md | 第 2 节 |
| COLR 增强 | 02_Patches.md | 第 3 节 |
| 升级建议 | 02_Patches.md | 第 7 节 |

### 使用相关

| 主题 | 文档 | 章节 |
|-----|------|-----|
| 依赖关系 | 04_Usage_in_OH.md | 第 2-3 节 |
| 集成示例 | 04_Usage_in_OH.md | 第 5 节 |
| 性能优化 | 04_Usage_in_OH.md | 第 6 节 |
| 故障排查 | 04_Usage_in_OH.md | 第 9 节 |

## 快速参考

### 常用命令

```bash
# 构建 HarfBuzz
hb build

# 运行测试
hb test

# 查看依赖
hb deps
```

### 配置变量

| 变量 | 说明 | 默认值 |
|-----|------|-------|
| ENABLE_ICCARM | 启用 ICCARM 支持 | false |
| HB_TINY | 启用精简模式 | false |
| HB_CUSTOM_MALLOC | 使用自定义内存分配 | false |
| HAVE_PTHREAD | 启用线程支持 | true |

### API 快速参考

```cpp
// 核心类型
hb_buffer_t*     // 塑形缓冲区
hb_font_t*       // 字体对象
hb_face_t*       // 字体字形数据
hb_blob_t*       // 二进制数据

// 主要函数
hb_buffer_create()           // 创建缓冲区
hb_shape()                   // 执行塑形
hb_ft_font_create()          // 创建 FreeType 字体
hb_ot_tag_from_language()    // 从语言获取 OpenType 标签
```

## 版本对应

| 文档版本 | HarfBuzz 版本 | 更新日期 |
|---------|--------------|---------|
| 1.0 | 11.0.0 | 2024年 |

## 反馈与贡献

如有以下问题，请联系维护者：

- 文档错误或遗漏
- 需要新增文档内容
- 技术问题咨询
- Patch 贡献建议

## 相关链接

- [上游 HarfBuzz](https://github.com/harfbuzz/harfbuzz)
- [HarfBuzz 官方文档](https://harfbuzz.github.io/)
- [OpenHarmony 文档](https://docs.openharmony.cn/)
- [IAR ICCARM 文档](https://www.iar.com/iccarm/)

---

*本文档最后更新: 2024年*
