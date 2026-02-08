# 阅读路线建议

## 文档结构

本目录包含 libphonenumber 库在 OpenHarmony 中的完整文档，重点说明 OH 的定制化内容和适配方式。

```
wiki/
├── README.md                  # 本文件 - 库概览和导航
├── SUMMARY.md                 # 阅读路线（本文件）
├── 01_Overview.md             # 原始库简介（简要）
├── 02_Patches.md             # Patch 详细分析 ⭐（核心文档）
├── 03_Build_Integration.md     # OH 构建系统适配
├── 04_Usage_in_OH.md         # 依赖关系与使用
├── 05_API_Differences.md      # API/接口差异（如有）
└── 06_Security.md             # 安全风险分析
```

---

## 阅读建议

### 对于新集成者（推荐路径）

如果你是首次在 OpenHarmony 中使用 libphonenumber，建议按以下顺序阅读：

1. **[README.md](README.md)** (5 分钟)
   - 了解库的基本信息和在 OH 中的作用

2. **[02_Patches.md](02_Patches.md)** (15 分钟) ⭐
   - 了解 OH 的所有 Patch 和适配内容
   - 重点：LIBPHONENUMBER_UPGRADE 机制

3. **[03_Build_Integration.md](03_Build_Integration.md)** (10 分钟)
   - 理解构建配置和依赖
   - 了解如何在项目中引入 libphonenumber

4. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** (10 分钟)
   - 查看其他模块如何使用 libphonenumber
   - 理解使用场景和集成方式

**预计时间**: 40 分钟

---

### 对于维护者（深度路径）

如果你需要维护或升级 libphonenumber，建议阅读全部文档：

1. **基础层** (30 分钟)
   - [01_Overview.md](01_Overview.md)
   - [02_Patches.md](02_Patches.md)
   - [03_Build_Integration.md](03_Build_Integration.md)

2. **集成层** (20 分钟)
   - [04_Usage_in_OH.md](04_Usage_in_OH.md)
   - [05_API_Differences.md](05_API_Differences.md)

3. **安全层** (15 分钟)
   - [06_Security.md](06_Security.md)

4. **参考材料** (可选)
   - [工作目录](_work/ASSESSMENT.md) - 详细评估报告
   - [工作目录](_work/NOTES.md) - 分析过程记录

**预计时间**: 65 分钟

---

### 快速参考

| 你的需求 | 推荐阅读 |
|----------|----------|
| **了解 OH 做了什么修改** | [02_Patches.md](02_Patches.md) |
| **集成到新项目** | [03_Build_Integration.md](03_Build_Integration.md), [04_Usage_in_OH.md](04_Usage_in_OH.md) |
| **升级上游版本** | [02_Patches.md](02_Patches.md), [06_Security.md](06_Security.md) |
| **了解 API 差异** | [05_API_Differences.md](05_API_Differences.md) |
| **安全审计** | [06_Security.md](06_Security.md) |

---

## 核心概念

### LIBPHONENUMBER_UPGRADE 机制

这是 OpenHarmony 对 libphonenumber 的核心创新，允许运行时更新电话号码元数据，而无需重新编译库。

**关键文件**：
- `update_metadata.cc` - 核心更新逻辑
- `update_libphonenumber.cc` - 数据加载入口
- `update_geocoding.cc` - 地理编码更新
- `geocoding_data.proto` - 更新数据格式

**详细了解**: [02_Patches.md](02_Patches.md) 第 2 节

### Patch 分类

libphonenumber 在 OH 中有**极少的 Patch**：
- **Build Config (1 个)**: 排除 Java demo 模块
- **代码适配**: 通过条件编译实现，无需传统 Patch

**详细清单**: [02_Patches.md](02_Patches.md)

---

## 常见问题

### Q: OH 的 libphonenumber 版本和上游有什么区别？

**A**: 主要区别在于：
1. 运行时元数据更新能力（`LIBPHONENUMBER_UPGRADE`）
2. OHOS 特定的初始化代码
3. 集成安全库（`libsec_shared`）
4. 新增 `ohos/` 目录约 2455 行代码

详见：[02_Patches.md](02_Patches.md), [05_API_Differences.md](05_API_Differences.md)

---

### Q: 如何更新电话号码规则而不重新编译？

**A**: 使用 `LIBPHONENUMBER_UPGRADE` 机制：

1. 准备新的元数据文件（protobuf 格式）
2. 放置到 `/system/etc/LIBPHONENUMBER/mount_dir/MetadataInfo`
3. 重启使用 libphonenumber 的进程（或系统重启）
4. 新规则自动加载

详见：[02_Patches.md](02_Patches.md) 第 2.2 节

---

### Q: 哪些模块在使用 libphonenumber？

**A**: 目前主要使用者是：
- `base/telephony/sms_mms` - SMS/MMS 模块

完整列表：[04_Usage_in_OH.md](04_Usage_in_OH.md)

---

### Q: 如何在项目中引入 libphonenumber？

**A**: 参考现有模块的构建配置（`base/telephony/sms_mms/BUILD.gn`）：

```gn
external_deps = [
  "libphonenumber:geocoding",
  "libphonenumber:phonenumber_standard",
]
```

详见：[03_Build_Integration.md](03_Build_Integration.md)

---

**最后更新**: 2026-02-08
