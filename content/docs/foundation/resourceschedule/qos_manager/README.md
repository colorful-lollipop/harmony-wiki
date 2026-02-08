# resourceschedule_qos_manager Wiki

> 本 Wiki 基于 OpenHarmony qos_manager 仓库代码自动生成

## 文档覆盖范围

### 已覆盖内容

| 模块 | 描述 | 证据来源 |
|------|------|----------|
| **项目概览** | 定位、边界、核心能力、运行环境 | `README_zh.md`, `bundle.json` |
| **目录结构** | 模块职责划分（不含测试） | 根目录结构扫描 |
| **架构说明** | 组件图、数据流、线程模型 | `services/`, `frameworks/` 代码分析 |
| **NDK API** | C 语言接口清单、参数、错误码 | `interfaces/kits/c/qos.h` |
| **内部 API** | C++ 接口、模块依赖方向 | `interfaces/inner_api/` |
| **GN 构建** | targets 列表、依赖、产物 | `BUILD.gn` 文件分析 |
| **编译产物** | .so 文件、安装路径、加载关系 | 构建配置推导 |
| **安全评审** | 攻击面、信任边界、风险清单 | 代码安全模式分析 |

### 未覆盖内容

| 模块 | 原因 | 建议 |
|------|------|------|
| **JS/ArkTS API** | 本模块为纯 Native 实现，无 N-API 层 | 需在 arkui/ace_engine 仓库查找 JS 封装 |
| **libtask_controller.z.so** | 实现代码在外部仓库 | 需关联查看 resource_schedule_service 仓库 |
| **内核驱动实现** | 属于内核子系统 | 需查看 kernel 子系统相关仓库 |
| **libgewu_client.z.so** | GEWU AI 推理模块 | 需查看对应仓库 |

## 文档更新方式

### 触发条件

当代码发生以下变更时，需要同步更新 Wiki：

| 变更类型 | 更新文档 | 更新内容 |
|----------|----------|----------|
| 新增/删除/修改 NDK API | `02_NDK_API.md` | API 清单、参数、错误码 |
| 新增/删除模块 | `00_Overview.md`, `01_Architecture.md` | 架构图、模块职责 |
| 新增/修改 BUILD.gn | `04_Build_Targets.md` | target 配置、依赖关系 |
| 新增安全相关代码 | `05_Security.md` | 攻击面、风险项 |
| 新增系统能力配置 | `appendix/Config_Flags.md` | feature flags、宏定义 |

### 更新流程

```bash
# 1. 克隆 wiki 仓库（如果独立管理）
git clone wiki_url

# 2. 在代码仓库执行文档生成脚本（如有）
./scripts/generate_wiki.sh

# 3. 手动同步关键变更
# 编辑对应的 wiki/*.md 文件

# 4. 提交 Wiki 变更
git add .
git commit -m "docs: sync with code change <commit_hash>"
git push
```

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航 + 阅读路线
├── 00_Overview.md        # 项目概览
├── 01_Architecture.md    # 架构说明
├── 02_NDK_API.md         # NDK 接口（重点）
├── 03_Inner_API.md       # 内部 API
├── 04_Build_Targets.md   # GN 构建配置
├── 05_Security.md        # 安全风险评审
└── appendix/
    ├── Callgraphs.md     # 关键调用链
    └── Config_Flags.md   # 配置参数
```

## 阅读建议

### 新人快速上手

1. **先读** `00_Overview.md` - 了解项目定位和核心能力
2. **再看** `01_Architecture.md` - 理解整体架构和数据流
3. **重点** `02_NDK_API.md` - 掌握对外接口使用方式

### 开发者深入

1. **开发新功能**：先读 `04_Build_Targets.md` 了解构建约束
2. **修改安全相关**：必读 `05_Security.md` 了解现有安全措施
3. **调试问题**：参考 `appendix/Callgraphs.md` 追踪调用链

## 元信息

- **文档生成时间**: 2026-02-06
- **代码版本**: qos_manager v3.1
- **OpenHarmony 版本**: 5.0+
- **代码仓库**: `foundation/resourceschedule/qos_manager`
- **生成工具**: OpenHarmony Wiki Generator Agent

## 反馈与贡献

- **文档问题**: 在代码仓库提 Issue
- **代码问题**: 在代码仓库提 Issue 或 PR
- **Wiki 改进**: 联系文档维护团队

## 相关链接

- [OpenHarmony 官方文档](https://docs.openharmony.cn)
- [qos_manager 源码](https://gitee.com/openharmony/foundation/tree/master/resourceschedule/qos_manager)
- [FFRT 并发框架](https://gitee.com/openharmony/appexecfwk_framework/tree/master/ffrt)
- [frame_aware_sched 帧感知调度](https://gitee.com/openharmony/drivers_peripheral/tree/master/frame_aware_sched)
