# hievent_lite Wiki

> OpenHarmony DFX 子系统 - 轻量级事件日志组件 Wiki

## 文档覆盖范围

本 Wiki 涵盖 hievent_lite 仓库的完整技术文档：

### ✅ 已覆盖内容

| 分类 | 说明 |
|------|------|
| **项目概览** | 定位、边界、核心能力、运行环境 |
| **架构设计** | 组件图、数据流、线程模型、关键时序 |
| **API 参考** | C API 清单、参数说明、错误码 |
| **构建系统** | GN targets、依赖关系、编译产物 |
| **安全评审** | 攻击面分析、风险点、修复建议 |
| **故障排查** | 常见问题、定位路径、调试方法 |

### ❌ 未覆盖内容

| 分类 | 说明 |
|------|------|
| **N-API / JS API** | 本仓库为纯 C 库，无 JS 绑定 |
| **测试代码** | 按规范不引用测试代码作为证据 |
| **运行时行为** | 依赖 hiview_lite 的部分实现细节 |

## 文档更新方式

### 何时更新 Wiki

当发生以下变更时，应同步更新 Wiki：

1. **新增 API** - 在 `02_API_Reference.md` 添加接口说明
2. **修改构建配置** - 更新 `03_Build_System.md` 的 targets 清单
3. **发现安全风险** - 在 `04_Security_Review.md` 添加新风险项
4. **变更架构** - 更新 `01_Architecture.md` 的数据流图

### 更新步骤

```bash
# 1. 克隆仓库
git clone https://gitee.com/openharmony/hiviewdfx_hievent_lite.git

# 2. 编辑对应文档
# 3. 确保所有结论有代码证据（路径 + 符号）
# 4. 提交变更
git add wiki/
git commit -m "docs: update wiki for [变更说明]"
```

## 文档结构

```
wiki/
├── README.md              # 本文档，覆盖范围与更新方式
├── SUMMARY.md             # 全站导航索引
├── 00_Overview.md         # 项目概览
├── 01_Architecture.md      # 架构设计
├── 02_API_Reference.md    # API 参考
├── 03_Build_System.md     # 构建系统
├── 04_Security_Review.md  # 安全评审
├── 05_Troubleshooting.md  # 故障排查
└── _work/                 # 工作目录（生成过程中使用）
    ├── NOTES.md           # 事实记录
    └── PLAN.md           # 任务计划
```

## 阅读建议

### 新人阅读顺序

1. **00_Overview.md** - 快速了解项目定位
2. **01_Architecture.md** - 理解核心架构
3. **02_API_Reference.md** - 掌握 API 使用
4. **05_Troubleshooting.md** - 常见问题速查

### 开发者阅读顺序

1. **02_API_Reference.md** - 查找接口定义
2. **03_Build_System.md** - 了解构建配置
3. **01_Architecture.md** - 理解实现细节
4. **04_Security_Review.md** - 安全注意事项

## 代码证据规范

本 Wiki 所有关键结论均标注代码证据：

```markdown
**证据**: `path:line` - 符号名
```

示例：
- **事件类型定义**: `interfaces/native/innerkits/hiview_event.h:27-38`
- **事件创建函数**: `frameworks/hiview_event.c:73` - `HiEventCreate()`
- **缓存管理**: `frameworks/hiview_output_event.c:35-46`

## 术语表

| 术语 | 说明 |
|------|------|
| **DFX** | Diagnostics, Fault-tolerance, eXtensibility - 可诊断性、容错、可扩展性 |
| **HiEvent** | 事件对象结构体 |
| **TLV** | Type-Length-Value - 类型-长度-值编码格式 |
| **LiteOS-M** | OpenHarmony 轻量级内核 |
| **mini 系统** | 轻量级设备系统 |

## 相关链接

- **代码仓库**: https://gitee.com/openharmony/hiviewdfx_hievent_lite
- **README**: [项目原始说明](../README_zh.md)
- **DFX 子系统**: https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md

---

*最后更新: 2026-02-06*
