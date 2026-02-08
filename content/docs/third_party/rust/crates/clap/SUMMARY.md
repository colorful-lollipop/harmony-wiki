# Wiki 阅读路线

## 推荐阅读顺序

### 快速了解 (5 分钟)
1. [README.md](./README.md) - 概览与导航
2. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 谁在用这个库

### 深入了解 (15 分钟)
1. [01_Overview.md](./01_Overview.md) - 库的基本介绍
2. [03_Build_Integration.md](./03_Build_Integration.md) - GN 构建细节
3. [06_Security.md](./06_Security.md) - 安全与升级建议

### 完整阅读 (30 分钟)
按文档编号顺序 01 → 06 阅读

## 按角色阅读

### 如果你是...

#### 系统构建工程师
推荐阅读: 03_Build_Integration.md → 04_Usage_in_OH.md
- 了解 GN 构建配置
- 掌握依赖关系
- 学习升级操作方法

#### 安全工程师
推荐阅读: 06_Security.md → 02_Patches.md
- 了解当前版本安全状态
- 评估升级风险

#### 第三方库维护者
推荐阅读: 全部文档
- 理解 OH 适配模式
- 作为零 Patch 集成的参考案例

#### 新加入的开发者
推荐阅读: 01_Overview.md → 04_Usage_in_OH.md
- 快速了解库的作用
- 知道在哪里使用它

## 文档分类索引

### 必读文档
| 文档 | 阅读理由 |
|------|---------|
| README.md | 入口导航 |
| 03_Build_Integration.md | GN 构建是 OH 集成的核心 |
| 04_Usage_in_OH.md | 理解库在系统中的位置 |

### 参考文档
| 文档 | 阅读理由 |
|------|---------|
| 02_Patches.md | 本库无 Patch，了解为什么 |
| 05_API_Differences.md | 本库无 API 差异 |
| 06_Security.md | 安全评估与升级建议 |

## 相关资源

- [OpenHarmony 构建系统文档](https://gitee.com/openharmony/build)
- [clap 官方文档](https://docs.rs/clap)
- [Rust FFI 工作组](https://github.com/rust-cli/team/)

---

*文档版本: v1.0*
