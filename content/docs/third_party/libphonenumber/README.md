# libphonenumber - OpenHarmony 第三方库文档

## 概览

**libphonenumber** 是 Google 开发的通用电话号码解析、格式化和验证库，支持 Java、C++ 和 JavaScript。OpenHarmony 集成了其 C++ 版本，用于 SMS/MMS 模块中的电话号码验证和格式化功能。

---

## 版本信息

| 项目 | 版本 |
|------|------|
| **上游版本** | 8.13.31 |
| **OpenHarmony 版本** | 3.1 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/google/libphonenumber.git |

---

## 功能概述

**原始库功能**：
- 解析、格式化和验证全球 200+ 个国家/地区的电话号码
- 区分号码类型：固定电话、移动电话、免费号码、紧急号码等
- 提供电话号码匹配和比较功能
- 支持实时格式化（AsYouType）
- 提供地理编码功能（根据电话号码获取地理位置信息）

---

## 在 OpenHarmony 中的定位

libphonenumber 在 OpenHarmony 中作为**基础电信组件**，主要为以下模块提供电话号码处理能力：

1. **SMS/MMS 模块** (`base/telephony/sms_mms`)
   - 验证目标电话号码格式
   - 格式化电话号码显示
   - 地理信息查询（通过 geocoding 库）

2. **系统服务层**
   - 通过 NDK/ArkTS 接口暴露电话号码验证功能
   - 支持应用层的电话号码输入验证

---

## 文档导航

| 文档 | 描述 | 优先级 |
|-----|------|--------|
| [01_Overview.md](01_Overview.md) | 库概览和 OH 集成概述 | ⭐⭐⭐ |
| [02_Patches.md](02_Patches.md) | Patch 详细分析和 OH 适配 | ⭐⭐⭐⭐⭐⭐ |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建系统适配 | ⭐⭐⭐ |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系和使用场景 | ⭐⭐⭐⭐ |
| [05_API_Differences.md](05_API_Differences.md) | API 差异说明 | ⭐⭐ |
| [06_Security.md](06_Security.md) | 安全风险分析 | ⭐⭐ |

**快速开始**：
1. 阅读 [02_Patches.md](02_Patches.md) 了解 OH 的所有定制化内容
2. 阅读 [03_Build_Integration.md](03_Build_Integration.md) 理解构建配置
3. 阅读 [04_Usage_in_OH.md](04_Usage_in_OH.md) 了解如何集成

---

## 关键特性

### OpenHarmony 特性

1. **运行时元数据更新** (`LIBPHONENUMBER_UPGRADE`)
   - 无需重新编译库即可更新电话号码规则
   - 从 `/system/etc/LIBPHONENUMBER/mount_dir/` 加载更新

2. **动态地理编码**
   - 支持运行时更新地理编码数据
   - 提供电话号码到地理位置的映射

3. **安全加固**
   - 集成 `libsec_shared` 进行边界检查
   - 使用 branch protector 和 CFI（Control Flow Integrity）

---

## 快速链接

- [工作目录](../wiki/_work/) - 分析过程记录
- [评估报告](../wiki/_work/ASSESSMENT.md) - Phase 0 详细评估
- [任务计划](../wiki/_work/PLAN.md) - 任务进度跟踪

---

**最后更新**: 2026-02-08
