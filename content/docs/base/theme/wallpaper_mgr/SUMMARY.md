# Wallpaper Mgr Wiki - 全站导航

> 新人阅读路线图

## 📖 必读入门

1. [README](README.md) - 文档说明与覆盖范围
2. [01_Overview.md](01_Overview.md) - 项目定位与核心概念

## 🏗️ 架构设计

3. [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构与模块职责
4. [04_Architecture.md](04_Architecture.md) - 组件图、数据流、时序图

## 🔌 接口文档

5. [03_API.md](03_API.md) - **N-API 接口清单**（重点）
   - JS API 名称、参数、返回值
   - C/C++ 入口函数与绑定位置
   - 权限、错误码、同步/异步模式

## 🛠️ 构建与实现

6. [05_Build.md](05_Build.md) - GN targets 与编译产物
7. [07_InnerAPI.md](07_InnerAPI.md) - Native 内部接口

## 🔒 安全相关

8. [06_Security.md](06_Security.md) - 安全风险评审（重点）
   - 攻击面清单
   - 信任边界
   - 可利用点与修复建议

## 🐛 故障排查

9. [08_Debug.md](08_Debug.md) - 构建/运行/调试问题定位

---

## 📋 快速跳转

### 按类型索引

| 分类 | 文档 |
|------|------|
| JS 开发者 | 03_API.md, 08_Debug.md |
| Native 开发者 | 04_Architecture.md, 07_InnerAPI.md |
| 安全审计 | 06_Security.md |
| 构建配置 | 05_Build.md |

### 按层次索引

| 层次 | 入口文档 |
|------|----------|
| JS/NAPI 层 | 03_API.md |
| Native 层 | 07_InnerAPI.md |
| Service 层 | 04_Architecture.md |
| 安全评估 | 06_Security.md |

---

## 🔗 外部链接

- [OpenHarmony 官方文档](https://www.openharmony.cn/)
- [Wallpaper Manager API 参考](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-wallpaper.md)
- [System Ability 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/system-dev/third-party-guides/sa-basic.md)

---

## 📝 更新日志

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2024-XX-XX | 初始版本，基于代码证据生成 |
