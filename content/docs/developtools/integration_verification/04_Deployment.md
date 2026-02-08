# 部署与使用指南

## 概述

本文档介绍集成验证部件的部署方法和基本使用流程。集成验证部件作为 OpenHarmony CI/CD 流程的一部分，需要正确配置运行环境才能发挥作用。

## 环境要求

### 硬件环境

| 组件 | 最低要求 | 推荐配置 |
|------|----------|----------|
| 开发板 | 支持 OpenHarmony 的标准开发板 | dayu200 或同等规格 |
| 存储空间 | 10GB 可用空间 | 50GB 以上 |
| 内存 | 8GB RAM | 16GB RAM |
| 网络 | 稳定互联网连接 | 高速网络 |

### 软件环境

| 依赖项 | 版本要求 | 说明 |
|--------|----------|------|
| Python | 3.x | 工具脚本运行环境 |
| Git | 最新版本 | 代码版本控制 |
| OpenHarmony SDK | 最新稳定版 | 编译和测试依赖 |

## 环境配置

### Python 环境配置

```bash
# 检查 Python 版本
python3 --version

# 安装依赖（如果有 requirements.txt）
pip3 install -r requirements.txt

# 或者使用 setup.py 安装
python3 setup.py install
```

### OpenHarmony SDK 配置

1. 下载 OpenHarmony SDK
2. 设置环境变量
3. 验证 SDK 安装

```bash
# 设置 SDK 路径（根据实际路径修改）
export OHOS_SDK_PATH=/path/to/sdk

# 验证 SDK
echo $OHOS_SDK_PATH
```

## 门禁测试部署

### 步骤 1：获取代码

```bash
# 克隆仓库
git clone https://gitee.com/openharmony/developtools_integration_verification.git

# 进入目录
cd developtools/integration_verification
```

### 步骤 2：配置用例关联

编辑 `cases/smoke/repo_cases_matrix.csv` 文件，配置仓库与测试用例的映射关系。

**证据来源**：`README.md:35-41`

```bash
# 使用文本编辑器打开配置文件
vim cases/smoke/repo_cases_matrix.csv
```

### 步骤 3：执行门禁测试

```bash
# 执行门禁测试（具体命令根据实际工具确定）
python3 tools/check.py --mode smoke

# 或者执行特定测试
python3 tools/fotff/run.py --test smoke
```

## 每日构建部署

### 步骤 1：配置定时任务

每日构建通常通过 CI 系统定时触发，可以使用 cron 或 CI 平台的定时功能。

```bash
# 编辑 crontab 配置定时任务
crontab -e

# 添加每日构建任务（示例：每天凌晨 2 点执行）
0 2 * * * cd /path/to/integration_verification && python3 tools/fotff/run.py --mode daily
```

### 步骤 2：配置版本归档

配置构建产物的存储路径和归档规则。

```bash
# 创建归档目录
mkdir -p /path/to/archive/daily_build

# 配置归档路径
export ARCHIVE_PATH=/path/to/archive/daily_build
```

### 步骤 3：执行每日构建

```bash
# 手动触发每日构建
python3 tools/fotff/run.py --mode daily --branch master

# 执行最小系统测试
python3 tools/fotff/run.py --mode mini_system
```

## 工具使用

### ROM/RAM 分析工具

#### ROM 分析

```bash
# 进入标准版工具目录
cd tools/rom_ram_analyzer/standard/

# 执行 ROM 分析
python3 rom_analyzer.py --input /path/to/elf/file --output report.xlsx
```

#### RAM 分析

```bash
# 执行 RAM 分析
python3 ram_analyzer.py --input /path/to/elf/file --output report.xlsx
```

### 开源合规检查

```bash
# 进入开源工具目录
cd tools/opensource_tools/src/

# 生成开源软件 README
python3 generate_readme_opensource.py

# 验证合规性
python3 validate_readme_opensource.py --input /path/to/project
```

### 主检查工具

```bash
# 查看帮助信息
python3 tools/check.py --help

# 执行所有检查
python3 tools/check.py --all

# 执行特定检查
python3 tools/check.py --tool rom_ram_analyzer
python3 tools/check.py --tool opensource_tools
python3 tools/check.py --tool code_access_control
```

## 设备部署

### DeployDevice 工具

**位置**：`DeployDevice/`

**功能**：将构建产物烧录到开发板。

```bash
# 进入部署目录
cd DeployDevice/

# 查看使用帮助
python3 deploy.py --help

# 执行设备部署
python3 deploy.py --device dayu200 --image /path/to/image
```

### 常见部署场景

#### 场景 1：本地开发板部署

```bash
# 连接开发板
# 确保 USB 连接正常

# 烧录系统镜像
python3 DeployDevice/flash_image.py --board dayu200 --image system.img
```

#### 场景 2：虚拟环境测试

```bash
# 启动虚拟设备
python3 DeployDevice/start_emulator.py --type standard
```

## 常见问题

### Q1：门禁测试失败怎么办？

1. 检查测试用例配置是否正确
2. 确认开发板连接状态
3. 查看详细日志排查具体原因
4. 联系相关模块负责人确认功能状态

### Q2：ROM/RAM 分析报告为空

可能原因：
- 输入文件路径错误
- ELF 文件格式不支持
- 缺少必要的依赖库

解决方法：
- 检查输入文件是否存在
- 确认使用正确版本的 Python
- 安装缺失的依赖包

### Q3：每日构建超时

可能原因：
- 测试用例数量过多
- 网络环境不稳定
- 硬件资源不足

解决方法：
- 优化测试用例选择策略
- 检查网络连接
- 增加硬件资源

### Q4：设备无法识别

可能原因：
- USB 驱动未安装
- 设备模式未开启
- USB 线缆质量问题

解决方法：
- 安装对应的 USB 驱动
- 确认开发板已设置为正确的设备模式
- 更换 USB 线缆

## 故障排查

### 日志查看

测试执行过程中产生的日志通常保存在以下位置：

| 日志类型 | 默认路径 |
|----------|----------|
| 门禁日志 | `logs/smoke/` |
| 每日构建日志 | `logs/daily/` |
| 部署日志 | `logs/deploy/` |

```bash
# 查看最近的门禁日志
ls -la logs/smoke/

# 查看特定日志文件
cat logs/smoke/smoke_test_20260101.log
```

### 调试模式

```bash
# 以调试模式运行
python3 tools/check.py --debug --mode smoke

# 启用详细输出
python3 tools/check.py --verbose --mode smoke
```

## 最佳实践

### 代码提交前

1. 确保本地测试通过
2. 检查代码质量检查结果
3. 验证开源合规状态

### 日常开发

1. 定期运行 ROM/RAM 分析
2. 关注代码复杂度指标
3. 保持测试用例的更新

### 版本发布

1. 执行完整的每日构建测试
2. 进行架构合规性检查
3. 生成完整的测试报告

## 相关文档

- [概览](./index.md)
- [目录结构与模块职责](./01_Directory_Structure.md)
- [用例管理](./02_Test_Cases.md)
- [工具集说明](./03_Tools.md)
- [架构设计](./05_Architecture.md)
