# device_board_hihope Wiki

## 文档覆盖范围

本文档仓库是 **OpenHarmony HiHope 设备板级配置仓**，包含多个开发板的硬件配置、外设驱动、构建配置等。

### 覆盖的模块

| 模块 | 描述 | 状态 |
|------|------|------|
| Neptune100 | 基于联盛德 W800 的 Wi-Fi & 蓝牙双模开发板 | ✅ 完整 |
| DAYU200 (rk3568) | 基于瑞芯微 RK3568 的 AI 开发板 | ✅ 完整 |
| DAYU210 | 基于瑞芯微 RK3588 的高性能 AI 开发板 | ✅ 完整 |
| nearlink_dk_3863 | 近连接开发板 | ✅ 完整 |
| shields | 板级 shield 配置 | ✅ 完整 |
| hcs | HDF 硬件描述配置 | ✅ 完整 |

### 未覆盖范围

- **N-API / JS API**: 本仓库为板级配置，不包含应用层 N-API 接口
- **内核源码**: 位于 `kernel/liteos_m` 或 `device/soc/rockchip` 仓库
- **系统服务**: 位于 `base/` 仓库
- **厂商驱动**: 位于 `device/soc/` 相关仓库

---

## 证据追溯原则

本文档所有技术结论均基于代码证据支撑，遵循以下规范：

1. **文件引用**: 使用 `文件路径:行号` 格式标注代码来源
2. **符号定义**: 关键函数、类、宏定义需标注完整路径
3. **调用链追溯**: 架构结论需提供调用链路径
4. **TODO 标记**: 无法确认处标注 `TODO(证据不足)`

**示例**：
```markdown
- 根构建配置：`BUILD.gn:14-23`
- DAYU210 构建目标：`dayu210/BUILD.gn:18-42`
- 条件编译：`if (is_support_graphic)` → `dayu210/BUILD.gn:26`
```

---

## 双路线阅读指南

根据您的身份选择推荐路线：

### 🟢 新人学习路线

目标：30分钟内理解项目定位、架构和基本使用方法

1. [00_Overview.md](00_Overview.md) → 项目定位、开发板介绍
2. [02_Directory_Structure.md](02_Directory_Structure.md) → 代码组织结构
3. [05_Board_Configurations.md](05_Board_Configurations.md) → 选择开发板
4. [04_GN_Build.md](04_GN_Build.md) → 编译构建流程

### 🔴 安全研究路线

目标：快速识别攻击面、信任边界和安全风险

1. [00_Overview.md](00_Overview.md) → 理解项目边界
2. [07_Security_Review.md](07_Security_Review.md) → 安全风险清单
3. [05_Board_Configurations.md](05_Board_Configurations.md) → 定位敏感配置
4. [02_Directory_Structure.md](02_Directory_Structure.md) → 追踪代码路径

---

## 文档结构

```
wiki/
├── README.md              # 本文档
├── SUMMARY.md             # 全站导航 + 双路线阅读指南
├── 00_Overview.md        # 项目概览
├── 01_Project_Scope.md   # 项目边界
├── 02_Directory_Structure.md  # 目录结构
├── 03_Architecture.md    # 架构说明
├── 04_GN_Build.md        # GN 构建配置
├── 05_Board_Configurations.md # 开发板配置
├── 06_Hardware_Drivers.md # 硬件驱动
├── 07_Security_Review.md # 安全风险评审
├── 08_Troubleshooting.md # 常见问题
└── appendix/
    ├── Config_Flags.md   # 配置参数
    └── Callchains.md     # 调用链
```

---

## 更新方式

### 手动更新

本文档基于代码手动维护。更新时请：

1. **修改对应 `.md` 文件**
2. **保持证据追溯**: 所有关键结论需标注代码证据（文件路径+行号）
3. **更新 SUMMARY.md**: 新增页面需添加导航链接
4. **检查链接有效性**: 确保内部链接可访问

### 版本对应

| 文档版本 | 仓库版本 | 更新日期 | 说明 |
|----------|----------|----------|------|
| v1.1 | master | 2026-02-07 | 新增双路线导航、证据追溯原则 |
| v1.0 | master | 2026-02-06 | 初始版本 |

---

## 相关链接

- **上游仓库**: [device_soc_rockchip](https://gitee.com/openharmony/device_soc_rockchip), [device_soc_winnermicro](https://gitee.com/openharmony/device_soc_winnermicro)
- **产品仓库**: [vendor/hihope](https://gitee.com/openharmony/vendor_hihope)
- **官方文档**: [OpenHarmony 文档](https://docs.openharmony.cn)
