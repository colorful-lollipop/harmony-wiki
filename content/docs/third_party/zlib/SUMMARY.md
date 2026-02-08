# SUMMARY - 阅读路线建议

本文档提供 zlib Wiki 的阅读建议，帮助读者快速找到所需信息。

---

## 快速导航

### 如果你是...

#### 1. 第一次接触 zlib + OH

**阅读顺序**:
1. [README.md](./README.md) - 快速概览
2. [01_Overview.md](./01_Overview.md) - 了解 OH 中的定位
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看依赖关系

**时间**: 15 分钟

#### 2. 需要集成 zlib 的开发者

**阅读顺序**:
1. [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 详解
2. [05_API_Differences.md](./05_API_Differences.md) - API 使用指南
3. [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 查看使用示例

**时间**: 20 分钟

#### 3. 负责升级/维护的人员

**阅读顺序**:
1. [02_Patches.md](./02_Patches.md) - Patch 详细分析
2. [03_Build_Integration.md](./03_Build_Integration.md) - 构建系统
3. [06_Security.md](./06_Security.md) - 安全风险

**时间**: 30 分钟

#### 4. 安全审计人员

**阅读顺序**:
1. [06_Security.md](./06_Security.md) - 完整安全分析
2. [02_Patches.md](./02_Patches.md) - Patch 安全评估
3. [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估

**时间**: 20 分钟

---

## 文档内容摘要

| 文档 | 字数 | 核心内容 | 目标读者 |
|-----|-----|---------|---------|
| [README.md](./README.md) | ~500 | 快速概览和导航 | 所有人 |
| [01_Overview.md](./01_Overview.md) | ~1500 | OH 定位和作用 | 架构师、新手 |
| [02_Patches.md](./02_Patches.md) | ~2000 | Patch 详细分析 | 维护者、升级人员 |
| [03_Build_Integration.md](./03_Build_Integration.md) | ~3000 | BUILD.gn 详解 | 开发者、集成人员 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | ~2500 | 依赖关系和使用场景 | 开发者、架构师 |
| [05_API_Differences.md](./05_API_Differences.md) | ~2000 | API 说明和示例 | 开发者 |
| [06_Security.md](./06_Security.md) | ~2000 | 安全风险分析 | 安全人员、维护者 |

---

## 主题导向阅读

### 主题: Patch 与适配

相关文档:
- [02_Patches.md](./02_Patches.md) - Patch 详解
- [03_Build_Integration.md](./03_Build_Integration.md) - BUILD.gn 配置
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 评估报告

### 主题: 使用与集成

相关文档:
- [03_Build_Integration.md](./03_Build_Integration.md) - 如何依赖
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景
- [05_API_Differences.md](./05_API_Differences.md) - API 使用

### 主题: 安全与升级

相关文档:
- [06_Security.md](./06_Security.md) - 安全分析
- [02_Patches.md](./02_Patches.md) - Patch 维护
- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 升级建议

---

## 核心信息速查

### 基本信息

```yaml
版本: 1.3.1
许可证: zlib/libpng
OH 组件: @ohos/zlib v3.1
子系统: thirdparty
```

### 关键文件

```
BUILD.gn              - OH 构建配置
huawei_zlib_CMakeList.patch - 唯一 Patch
zlib.ndk.json         - NDK 导出
```

### 构建目标

```
libz          - 静态库
shared_libz   - 共享库
libz_crc      - CRC 优化库
```

### Patch 概要

```
数量: 1 个
类型: 构建适配
修改: 禁用 MINGW 和测试代码
```

---

## 外部参考

- [上游 zlib 文档](https://github.com/madler/zlib/tree/develop/doc)
- [zlib 手册](https://zlib.net/manual.html)
- [RFC 1950-1952](https://tools.ietf.org/html/rfc1950)
