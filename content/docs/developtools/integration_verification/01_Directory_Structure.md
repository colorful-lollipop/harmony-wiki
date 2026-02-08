# 目录结构与模块职责

## 顶层目录结构

```
/developtools/integration_verification/
├── cases/                       # [用例目录] 测试用例存放位置
│   ├── daily/                   # [每日构建] 每日构建相关用例
│   │   └── mini_system/         # [最小系统测试] 最小系统功能验证
│   └── smoke/                   # [门禁冒烟] 门禁测试用例
│       ├── audio/               # [音频测试] 音频功能验证
│       ├── basic/               # [基础功能] 核心功能测试
│       │   ├── screenshot32/   # [截图测试 32位] 32位系统截图测试
│       │   └── screenshot64/   # [截图测试 64位] 64位系统截图测试
│       ├── distributed/         # [分布式测试] 分布式场景端到端验证
│       └── video/               # [视频测试] 视频播放功能验证
├── tools/                       # [工具集] 公共工具和脚本
│   ├── rom_ram_analyzer/       # [资源分析] ROM/RAM 使用分析
│   ├── fotff/                  # [自动化框架] 端到端测试框架
│   ├── opensource_tools/       # [开源合规] 开源许可证检查工具
│   ├── code_access_control/    # [代码管控] 代码访问控制检查
│   ├── components_deps/        # [依赖检查] 组件依赖分析
│   ├── deps_guard/             # [依赖守护] 依赖关系守护
│   ├── gated_check_in/         # [门禁检查] 门禁配置检查
│   ├── precise_build/          # [精准构建] 精准构建工具
│   ├── startup_guard/          # [启动守护] 启动流程守护
│   ├── arkui_tools/            # [ArkUI 工具] ArkUI 相关工具
│   └── check.py                # [检查脚本] 主检查入口
├── DeployDevice/                # [设备部署] 设备部署工具和脚本
├── figures/                    # [图表资源] 文档使用的图表文件
├── test/                       # [自测试] 项目自身的测试用例
├── wiki/                       # [本文档] Wiki 文档目录
├── README.md                   # [项目说明] 项目主说明文档
├── README.en.md               # [英文说明] 英文版项目说明
├── OAT.xml                    # [OAT 配置] OpenHarmony 资产工具配置
└── LICENSE                    # [许可证] Apache 2.0 许可证
```

**证据来源**：`README.md:14-29`

## cases 目录详解

### 目录职责

`cases/` 目录是测试用例的集中存放位置，按照测试类型和功能模块进行组织。该目录下的用例分为两大类：门禁冒烟用例和每日构建用例。

### daily 子目录

**路径**：`cases/daily/`

**职责**：存放每日构建流程所需的测试用例。

**证据来源**：`README.md:16-18`

> ├── daily                   # 每日构建
> │   └── mini_system         # 最小系统测试

#### mini_system 子目录

**路径**：`cases/daily/mini_system/`

**职责**：最小系统测试用例，用于验证 OpenHarmony 在最小系统配置下的功能完整性。最小系统测试关注核心功能的正常运行，不涉及复杂的硬件依赖。

### smoke 子目录

**路径**：`cases/smoke/`

**职责**：门禁冒烟测试用例集合，是代码提交时首先执行的测试。

**证据来源**：`README.md:18-25`

> └── smoke                   # 门禁冒烟
>     ├── audio               # 音频用例
>     ├── basic               # 基础功能用例
>     │   ├── screenshot32
>     │   └── screenshot64
>     ├── distributed         # 分布式场景端到端用例
>     └── video               # 视频用例

#### audio 子目录

**路径**：`cases/smoke/audio/`

**职责**：音频功能测试用例，验证系统的音频播放、录制和处理功能。

#### basic 子目录

**路径**：`cases/smoke/basic/`

**职责**：基础功能测试用例，验证系统的核心功能模块。

##### screenshot32 子目录

**路径**：`cases/smoke/basic/screenshot32/`

**职责**：32 位系统环境下的截图功能测试。

##### screenshot64 子目录

**路径**：`cases/smoke/basic/screenshot64/`

**职责**：64 位系统环境下的截图功能测试。

#### distributed 子目录

**路径**：`cases/smoke/distributed/`

**职责**：分布式场景端到端测试用例，验证多设备协同工作的能力。

#### video 子目录

**路径**：`cases/smoke/video/`

**职责**：视频功能测试用例，验证视频播放、编解码等功能。

## tools 目录详解

### 目录职责

`tools/` 目录包含项目提供的各种公共工具集，用于支持测试、构建、分析等场景。

**证据来源**：目录列表输出

### rom_ram_analyzer 子目录

**路径**：`tools/rom_ram_analyzer/`

**职责**：ROM/RAM 使用分析工具，帮助开发者分析二进制文件的存储器和内存占用情况。

**证据来源**：`README.md:28`

> └── rom_ram_analyzer        # ROM/RAM分析工具

#### standard 子目录

**路径**：`tools/rom_ram_analyzer/standard/`

**职责**：标准系统版本的 ROM/RAM 分析工具实现。

**包含文件**：
- `ram_analyzer.py` - RAM 分析器主程序
- `rom_analyzer.py` - ROM 分析器主程序
- `pkgs/` - 工具依赖包目录

#### lite_small 子目录

**路径**：`tools/rom_ram_analyzer/lite_small/`

**职责**：轻量系统版本的 ROM/RAM 分析工具实现。

### fotff 子目录

**路径**：`tools/fotff/`

**职责**：Full OTA Flow Framework，完整的 OTA 升级流程测试框架，支持自动化端到端测试。

### opensource_tools 子目录

**路径**：`tools/opensource_tools/`

**职责**：开源合规检查工具集，包括许可证检查、README 验证等功能。

**包含文件**：
- `src/generate_readme_opensource.py` - 开源 README 生成
- `src/validate_readme_opensource.py` - 开源 README 验证
- `src/spdx_license_matcher.py` - SPDX 许可证匹配器
- `data/spdx.json` - SPDX 许可证数据

### code_access_control 子目录

**路径**：`tools/code_access_control/`

**职责**：代码访问控制检查工具，确保代码符合访问控制策略。

**包含文件**：
- `check_arkweb_hard_coded.py` - 硬编码检查
- `part_compile_build.py` - 部件编译构建检查
- `sdk_check_comment.py` - SDK 注释检查

### 其他工具目录

| 目录名 | 路径 | 职责 |
|--------|------|------|
| components_deps | `tools/components_deps/` | 组件依赖分析工具 |
| deps_guard | `tools/deps_guard/` | 依赖关系守护工具 |
| gated_check_in | `tools/gated_check_in/` | 门禁配置检查工具 |
| precise_build | `tools/precise_build/` | 精准构建工具 |
| startup_guard | `tools/startup_guard/` | 启动流程守护工具 |
| arkui_tools | `tools/arkui_tools/` | ArkUI 相关工具 |

### 主检查脚本

**路径**：`tools/check.py`

**职责**：项目的主检查入口脚本，提供统一的检查命令行接口。

## DeployDevice 目录

**路径**：`DeployDevice/`

**职责**：设备部署工具和脚本，用于将构建产物烧录到开发板或虚拟设备。

## figures 目录

**路径**：`figures/`

**职责**：项目文档中使用的图表资源文件。

## test 目录

**路径**：`test/`

**职责**：项目自身的测试用例，用于验证集成验证部件的功能正确性。

## wiki 目录

**路径**：`wiki/`

**职责**：本文档目录，存放项目的 Wiki 文档。

## 相关文档

- [概览](./index.md)
- [用例管理](./02_Test_Cases.md)
- [工具集说明](./03_Tools.md)
- [部署与使用指南](./04_Deployment.md)
