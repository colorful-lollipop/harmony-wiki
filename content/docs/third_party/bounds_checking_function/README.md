# bounds_checking_function Wiki

## 库概览

**bounds_checking_function** 是 OpenHarmony 的核心基础安全库，提供遵循 C11 Annex K 标准的边界检查安全函数。

### 关键信息

| 属性 | 值 |
|------|-----|
| **原始库** | openEuler:libboundscheck |
| **版本** | v1.1.16 |
| **许可证** | MulanPSL-2.0 |
| **OH 组件** | @ohos/bounds_checking_function |
| **OH 版本** | 3.1 |
| **子系统** | thirdparty |

### 核心特性

- **40 个安全函数** - 覆盖内存、字符串、格式化 I/O 操作
- **零 Patch 策略** - 与上游保持完全一致
- **全系统依赖** - 1941+ 个 BUILD.gn 文件依赖
- **多系统支持** - mini / small / standard 全适配
- **安全增强** - 支持 PAC 等安全编译选项

---

## 文档导航

### 快速入门

| 文档 | 内容 | 适合读者 |
|------|------|----------|
| [01_Overview.md](./01_Overview.md) | 库简介、OH 中的作用和定位 | 所有开发者 |
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议 | 新接触者 |

### 深度分析

| 文档 | 内容 | 适合读者 |
|------|------|----------|
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） | 维护者、升级评估 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 配置详解 | 系统开发者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系与使用场景 | 架构师、开发者 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异说明 | 应用开发者 |
| [06_Security.md](./06_Security.md) | 安全风险分析 | 安全工程师 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估报告 |

---

## 关键结论

### 1. 无 Patch 策略

**bounds_checking_function 是 OpenHarmony 中罕见的"零 Patch"基础库**：

- 与上游 libboundscheck 源码完全一致
- OH 特有的适配通过 BUILD.gn 配置实现
- 升级维护成本低，无 Patch 冲突风险

### 2. 基础设施地位

**这是 OpenHarmony 最基础的系统库之一**：

```
依赖统计:
├── 1941+ BUILD.gn 文件直接依赖
├── 5000+ 源文件间接依赖
├── 覆盖几乎所有子系统
└── 系统启动全生命周期必需
```

### 3. OH 特有的适配

虽然无源码 Patch，但 BUILD.gn 中包含重要的 OH 特有配置：

- **PAC 保护** (`branch_protector_ret = "pac_ret"`) - ARM64 指针认证
- **多系统支持** - ohos_lite / standard 条件编译
- **SDK 分层** - chipsetsdk_sp / platformsdk / sasdk
- **多镜像安装** - system / updater / ramdisk

### 4. 安全价值

**消除 C 语言中最危险的内存安全漏洞**：

- 缓冲区溢出 (CWE-120, CWE-121)
- 格式化字符串漏洞
- 未初始化内存访问

---

## 快速参考

### 常用安全函数替换

| 不安全函数 | 安全替换 | 风险 |
|-----------|----------|------|
| `strcpy` | `strcpy_s(dest, destsz, src)` | 缓冲区溢出 |
| `strcat` | `strcat_s(dest, destsz, src)` | 缓冲区溢出 |
| `sprintf` | `sprintf_s(buf, bufsz, fmt, ...)` | 缓冲区溢出 |
| `memcpy` | `memcpy_s(dest, destsz, src, n)` | 溢出/重叠 |
| `gets` | `gets_s(buf, bufsz)` | 无限输入 |

### 在 BUILD.gn 中引用

```gn
# 推荐方式
ohos_shared_library("my_module") {
  external_deps = [
    "bounds_checking_function:libsec_shared",
  ]
}

# 头文件
#include "securec.h"
```

### 错误处理模式

```cpp
#include "securec.h"

if (strcpy_s(dest, sizeof(dest), src) != EOK) {
    // 错误处理
    HILOG_ERROR("String copy failed");
    return false;
}
```

---

## 维护者信息

| 项目 | 信息 |
|------|------|
| **上游地址** | https://gitee.com/openeuler/libboundscheck |
| **维护者** | jianghan2@huawei.com |
| **OH 子系统** | thirdparty |
| **OH 部件** | bounds_checking_function |

## 更新历史

| 日期 | 版本 | 变更 |
|------|------|------|
| 2025-02 | Wiki 1.0 | 初始文档创建 |

---

## 贡献与反馈

如发现文档错误或需要补充内容，请联系：
- OpenHarmony 第三方库维护团队
- 提交 Issue 至 OpenHarmony 主仓库

---

**文档版本**: 1.0  
**最后更新**: 2025-02-08  
**维护状态**: 活跃
