# Linux 内核 6.6 Wiki

## 概述

本 Wiki 为 OpenHarmony `kernel_linux_common_6.6` 仓库的工程文档，提供 Linux 内核 6.6 的完整技术参考。

**仓库信息**:
- 仓库名称: kernel_linux_common_6.6
- 内核版本: Linux 6.6
- 用途: OpenHarmony 的 Linux 内核基础（原生代码）
- 官方文档: https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/kernel/Readme-CN.md

## 文档覆盖范围

### 已覆盖

- ✅ 项目概览与核心概念
- ✅ 目录结构与模块职责
- ✅ 内核架构（组件/数据流/线程模型）
- ✅ 系统调用接口（对外 API）
- ✅ 内部 API（子系统接口）
- ✅ 构建系统（Kbuild/Kconfig）
- ✅ 编译产物（vmlinux/ko/dtb）
- ✅ 安全风险评审（攻击面/可利用点）

### 未覆盖

- ⚠️ 特定架构的实现细节（请参考 arch/*/README）
- ⚠️ 每个设备驱动的详细文档（请参考 drivers/*/Documentation）
- ⚠️ 内核调试工具使用（请参考 Documentation/trace/）
- ⚠️ 性能调优指南（请参考 Documentation/admin-guide/）
- ⚠️ 设备树编写规范（请参考 Documentation/devicetree/）

## 文档更新方式

### 自动化更新（推荐）

当内核代码变更时，可重新生成此 Wiki：

```bash
# 1. 更新代码库
git pull

# 2. 重新运行 Wiki 生成工具
./tools/generate-wiki.sh  # 假设有此脚本
```

### 手动更新

如需手动维护：

1. **更新路径/符号**: 确保 `grep` / `find` 结果是最新的
2. **更新代码片段**: 验证代码行号仍然正确
3. **更新时间戳**: 在文档末尾更新 `最后更新时间`
4. **更新 PLAN.md**: 记录修改内容和原因

### 版本对照表

| Wiki 版本 | 内核版本 | 生成时间 | 主要变更 |
|-----------|-----------|----------|---------|
| v1.0 | 6.6 | 2026-02-06 | 初始版本 |

## 文档约定

### 代码证据

所有关键结论必须包含代码证据：
- **文件路径**: 例如 `kernel/sched/core.c:1234`
- **符号名**: 例如 `schedule()` 函数
- **代码片段**: 最小必要片段说明逻辑

### 术语统一

| 术语 | 说明 |
|------|------|
| 系统调用 (syscall) | 用户态 → 内核态的调用接口 |
| 内核模块 (ko) | 可动态加载的内核组件 |
| 设备驱动 | 操作硬件设备的内核代码 |
| LSM | Linux Security Modules（安全框架） |

### 测试内容排除

本 Wiki **不包含**以下内容的分析：
- `test/` 目录下的测试代码
- `*_test.*` 或 `*_fuzzer.*` 文件
- `unittest/` 或 `fuzztest/` 目录

**原因**: 测试代码不是生产代码的一部分，不反映实际系统行为。

## 文档结构

```
wiki/
├── README.md              # 本文件
├── SUMMARY.md            # 全站导航与阅读路线
├── 00_Overview.md        # 项目概览与核心概念
├── 01_Directory_Structure.md   # 目录结构与模块职责
├── 02_Architecture.md         # 架构说明
├── 03_System_Calls.md         # 系统调用接口
├── 04_Internal_APIs.md        # 内部 API
├── 05_Build_System.md         # 构建系统
├── 06_Build_Artifacts.md      # 编译产物
├── 07_Security_Review.md      # 安全风险评审
├── 08_Troubleshooting.md      # 常见问题与定位路径
├── appendix/
│   ├── Callgraphs.md          # 关键调用链（可选）
│   └── Config_Flags.md       # 关键配置选项（可选）
└── _work/
    ├── NOTES.md              # 工作笔记
    └── PLAN.md             # 生成计划
```

## 阅读建议

### 新人路线

1. 从 [00_Overview.md](00_Overview.md) 开始，了解项目定位
2. 阅读 [01_Directory_Structure.md](01_Directory_Structure.md)，熟悉代码布局
3. 阅读 [02_Architecture.md](02_Architecture.md)，理解子系统交互
4. 根据需要深入：
   - 添加新功能 → [03_System_Calls.md](03_System_Calls.md)
   - 内核开发 → [04_Internal_APIs.md](04_Internal_APIs.md)
   - 构建内核 → [05_Build_System.md](05_Build_System.md)
   - 安全问题 → [07_Security_Review.md](07_Security_Review.md)

### 开发者路线

- 驱动开发: [01_Directory_Structure.md](01_Directory_Structure.md) → [02_Architecture.md](02_Architecture.md) → [04_Internal_APIs.md](04_Internal_APIs.md)
- 内核模块开发: [03_System_Calls.md](03_System_Calls.md) → [05_Build_System.md](05_Build_System.md)
- 安全审计: [07_Security_Review.md](07_Security_Review.md) → [02_Architecture.md](02_Architecture.md)

## 问题反馈

如发现文档错误或需要补充，请：
1. 记录问题（路径/行号/描述）
2. 提交 issue 或 PR 到文档仓库
3. 参考 `wiki/_work/PLAN.md` 了解文档生成流程

## 参考资料

### 官方内核文档

- **主页**: https://www.kernel.org/doc/html/latest/
- **构建文档**: Documentation/kbuild/
- **设备驱动**: Documentation/driver-api/
- **安全文档**: Documentation/security/
- **网络文档**: Documentation/networking/
- **文件系统**: Documentation/filesystems/

### OpenHarmony 文档

- **内核开发**: https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/kernel/
- **驱动开发**: https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/driver/

---

**最后更新时间**: 2026-02-06
**生成工具**: OpenHarmony Wiki Generator v1.0
**内核版本**: 6.6
