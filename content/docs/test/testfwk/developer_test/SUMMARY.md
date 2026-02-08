# 导航与阅读路线

## 新人阅读路线（推荐顺序）

```
1️⃣ 先了解项目定位
   └─→ [01_Overview.md](01_Overview.md)

2️⃣ 再看目录结构
   └─→ [03_Directory_Structure.md](03_Directory_Structure.md)

3️⃣ 理解系统架构
   └─→ [02_Architecture.md](02_Architecture.md)

4️⃣ 学习使用指南
   └─→ [06_Usage_Guide.md](06_Usage_Guide.md)

5️⃣ 根据需要查阅
   ├─→ [04_Configuration.md](04_Configuration.md)
   ├─→ [05_Build_System.md](05_Build_System.md)
   ├─→ [07_Examples.md](07_Examples.md)
   └─→ [08_Security_Review.md](08_Security_Review.md)
```

## 全文档列表

### 快速入门

| 文档 | 说明 |
|------|------|
| [README.md](README.md) | Wiki 使用说明 |
| [SUMMARY.md](SUMMARY.md) | 本导航文档 |

### 核心文档

| 文档 | 说明 |
|------|------|
| [01_Overview.md](01_Overview.md) | 项目定位、核心能力、运行环境、关键概念 |
| [02_Architecture.md](02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [03_Directory_Structure.md](03_Directory_Structure.md) | 目录结构与模块职责 |

### 使用指南

| 文档 | 说明 |
|------|------|
| [04_Configuration.md](04_Configuration.md) | 配置项详解（user_config.xml, framework_config.xml） |
| [05_Build_System.md](05_Build_System.md) | GN targets、编译产物、构建配置 |
| [06_Usage_Guide.md](06_Usage_Guide.md) | 使用指南、命令参考、执行流程 |
| [07_Examples.md](07_Examples.md) | 测试用例示例（calculator, app_info, detector等） |

### 安全与评审

| 文档 | 说明 |
|------|------|
| [08_Security_Review.md](08_Security_Review.md) | 安全风险评审、攻击面分析、修复建议 |

## 命令速查

```bash
# 启动测试框架
./start.sh          # Linux
start.bat          # Windows

# 运行测试
run -t UT          # 单元测试
run -t PERF        # 性能测试
run -t FUZZ        # 模糊测试
run -t ACTS        # 活动测试

# 查看帮助
help
show productlist   # 查看支持的产品形态
show typelist      # 查看支持的测试类型
```

## 关键文件速查

| 路径 | 说明 |
|------|------|
| `src/main/__main__.py` | 框架入口 |
| `src/core/command/console.py` | 控制台交互 |
| `src/core/command/run.py` | 测试执行 |
| `src/core/build/build_manager.py` | 构建管理 |
| `src/core/driver/drivers.py` | 设备驱动 |
| `config/user_config.xml` | 用户配置 |
| `config/framework_config.xml` | 框架配置 |
| `BUILD.gn` | GN 构建入口 |

## 反馈与贡献

如发现文档错误或遗漏，请：
1. 提交 Issue 到对应仓库
2. 或直接修改 Wiki 后提交 PR
