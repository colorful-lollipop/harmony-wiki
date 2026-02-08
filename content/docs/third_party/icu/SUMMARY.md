# SUMMARY - 阅读路线建议

## 根据角色的阅读路线

### 路线 1: 快速了解 (5分钟)
适合想快速了解 ICU 在 OH 中概况的读者

1. [README.md](./README.md) - 查看快速信息
2. [01_Overview.md](./01_Overview.md) - 了解 OH 中的定位
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看使用场景

### 路线 2: 开发者集成 (15分钟)
适合需要在项目中集成或使用 ICU 的开发者

1. [01_Overview.md](./01_Overview.md) - 了解功能范围
2. [03_Build_Integration.md](./03_Build_Integration.md) - 查看构建配置
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 了解如何依赖
4. [05_API_Differences.md](./05_API_Differences.md) - 查看新增 API

### 路线 3: 系统适配 (30分钟)
适合需要理解或修改 ICU 适配的系统工程师

1. [01_Overview.md](./01_Overview.md) - 了解整体架构
2. [02_Patches.md](./02_Patches.md) - 深入理解 OH 适配
3. [03_Build_Integration.md](./03_Build_Integration.md) - 查看构建细节
4. [06_Security.md](./06_Security.md) - 了解安全考量

### 路线 4: 升级维护 (45分钟)
适合需要升级 ICU 版本或维护适配的工程师

1. [02_Patches.md](./02_Patches.md) - 理解所有修改
2. [03_Build_Integration.md](./03_Build_Integration.md) - 检查构建配置
3. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 参考评估报告
4. [06_Security.md](./06_Security.md) - 检查 CVE 和安全问题

## 文档依赖关系

```
README.md
    ├── 01_Overview.md
    │       ├── 02_Patches.md
    │       └── 05_API_Differences.md
    ├── 03_Build_Integration.md
    ├── 04_Usage_in_OH.md
    └── 06_Security.md
```

## 关键章节索引

| 主题 | 主要文档 |
|------|----------|
| 农历功能 | [02_Patches.md#2-中国农历日历支持](./02_Patches.md) |
| 数据裁剪 | [03_Build_Integration.md#数据裁剪配置](./03_Build_Integration.md) |
| NDK 使用 | [04_Usage_in_OH.md#ndk-封装层](./04_Usage_in_OH.md) |
| 构建配置 | [03_Build_Integration.md#buildgn-结构](./03_Build_Integration.md) |
| 符号导出 | [05_API_Differences.md#ndk-导出-api](./05_API_Differences.md) |

## 版本历史

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-02-08 | v1.0 | 初始版本，基于 ICU 74.2 |
