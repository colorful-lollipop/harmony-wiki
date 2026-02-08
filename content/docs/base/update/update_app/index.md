# OpenHarmony update_app 模块

> OpenHarmony 应用增量更新模块 - 提供高效、安全的应用版本更新能力

## 项目概述

update_app 是 OpenHarmony 系统中的应用更新模块，负责处理应用的版本检查、增量下载、补丁应用和完整性校验等核心功能。

### 核心能力

| 能力 | 说明 | 关键技术 |
|------|------|----------|
| 版本检查 | 检测应用最新版本 | 增量对比、版本号解析 |
| 增量下载 | 只下载变化部分 | bsdiff、bsdiff算法优化 |
| 补丁应用 | 安全应用更新包 | 签名校验、回滚机制 |
| 完整性校验 | 确保更新正确性 | SHA256、证书链验证 |

### 技术特点

- **低带宽**: 增量更新相比全量下载节省 70%+ 带宽
- **快速**: 差分算法优化，应用补丁仅需数秒
- **安全**: 多层签名校验，确保来源可靠
- **稳定**: 支持断点续传、更新回滚

## 快速入门

### 环境要求

| 依赖 | 版本要求 | 说明 |
|------|----------|------|
| OpenHarmony SDK | 4.0+ | API Level 9+ |
| Node.js | 16+ | 构建工具链 |
| GN | r155+ | 构建系统 |
| Ninja | 1.10+ | 构建执行器 |

### 构建命令

```bash
# 完整构建
./build.sh

# 仅编译
make update_core

# 运行测试
make test
```

### 首个 API 调用

```javascript
import update from '@ohos.update';

// 检查更新
let checkResult = await update.checkForUpdates('com.example.app');
console.log('最新版本:', checkResult.latestVersion);
```

## 文档导航

### 新人推荐阅读顺序

```
1. 项目定位 → 01_Project_Overview.md
2. 架构设计 → 02_Architecture.md
3. API 使用 → 03_N-API_Reference.md
4. 构建配置 → 05_Build_System.md
```

### 开发者常用链接

| 需求 | 跳转 |
|------|------|
| N-API 接口详情 | [03_N-API_Reference.md](./03_N-API_Reference.md) |
| 内部模块接口 | [04_Inner_API.md](./04_Inner_API.md) |
| GN 构建配置 | [05_Build_System.md](./05_Build_System.md) |
| 安全注意事项 | [07_Security_Review.md](./07_Security_Review.md) |
| 问题排查 | [08_Troubleshooting.md](./08_Troubleshooting.md) |

## 模块架构

```
update_app/
├── src/
│   ├── base/           # 核心业务逻辑
│   │   ├── update/     # 更新流程控制
│   │   ├── diff/       # 差分算法
│   │   ├── patch/      # 补丁应用
│   │   └── verify/     # 校验模块
│   ├── napi/           # N-API 接口层
│   │   ├── native/     # C++ 绑定
│   │   └── js/         # JS 接口定义
│   └── server/         # 系统服务
├── interfaces/         # 对外接口定义
│   ├── napi/          # N-API 声明
│   └── inner_api/     # 内部 API
├── include/           # 头文件
├── utils/             # 工具模块
└── wiki/             # 文档目录
```

## 版本信息

| 版本 | 日期 | 主要变更 |
|------|------|----------|
| v1.0.0 | 2026-02-06 | 初始版本 |

## 参与贡献

欢迎提交 Issue 和 Pull Request：

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/xxx`)
3. 提交变更 (`git commit -m 'feat: xxx'`)
4. 推送分支 (`git push origin feature/xxx`)
5. 创建 Pull Request

## 许可证

本项目采用 [Apache License 2.0](./LICENSE) 开源协议。
