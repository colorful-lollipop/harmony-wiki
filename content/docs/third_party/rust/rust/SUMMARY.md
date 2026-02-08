# 阅读路线建议

## 快速了解 (5 分钟)

如果您只需要快速了解 Rust 工具链在 OpenHarmony 中的角色：

1. 阅读 [README.md](./README.md) - 2 分钟
2. 查看 [01_Overview.md](./01_Overview.md) - 3 分钟

## 深入理解 (30 分钟)

如果您需要全面了解 Rust 工具链的 OH 适配细节：

### 第一步：基础了解
- [README.md](./README.md) - 了解项目全貌
- [01_Overview.md](./01_Overview.md) - 理解 Rust 在 OH 中的定位

### 第二步：核心适配分析
- [02_Patches.md](./02_Patches.md) - **重点** - 理解所有 OH 特有修改
- [03_Build_Integration.md](./03_Build_Integration.md) - 理解构建系统集成

### 第三步：实际使用
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解使用场景
- [05_API_Differences.md](./05_API_Differences.md) - 如需进行底层开发

## 维护者路线 (1 小时+)

如果您需要维护或修改此工具链：

1. 完整阅读以上所有文档
2. 查看 [06_Security.md](./06_Security.md) - 了解安全注意事项
3. 参考 [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 完整的评估记录
4. 参考 [_work/NOTES.md](./_work/NOTES.md) - 详细分析过程

## 文档优先级

| 优先级 | 文档 | 必读人群 |
|--------|------|---------|
| **P0** | 02_Patches.md | 所有需要理解 OH 适配的人 |
| **P0** | 03_Build_Integration.md | 构建系统维护者 |
| **P1** | 01_Overview.md | 新用户 |
| **P1** | 04_Usage_in_OH.md | 应用开发者 |
| **P2** | 05_API_Differences.md | 底层开发者 |
| **P2** | 06_Security.md | 安全审查者 |

## 推荐阅读顺序

```
新手入门
    ↓
README.md → 01_Overview.md → 04_Usage_in_OH.md
    ↓
深入开发
    ↓
02_Patches.md → 03_Build_Integration.md → 05_API_Differences.md
    ↓
安全审查
    ↓
06_Security.md
```
