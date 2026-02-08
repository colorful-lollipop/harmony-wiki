# 06 - 安全风险分析

## 概览

| 评估项目 | 状态 |
|----------|------|
| 已知 CVE (25.01) | 待确认 |
| Patch 引入的新攻击面 | 无 |
| 编译安全选项 | ✅ PAC-RET (ARM64) |
| 代码质量 | ✅ 高 |

**总体评估**: 低风险，成熟稳定的库，Public Domain 许可证无合规风险。

---

## 1. CVE 安全漏洞

### 1.1 LZMA SDK 已知漏洞

**TODO(需确认)**: 以下信息需要进一步验证和更新。

根据公开 CVE 数据库查询：

| CVE ID | 影响版本 | 严重程度 | 描述 | OH 状态 |
|--------|----------|----------|------|---------|
| CVE-2024-XXXX | 待查询 | - | - | 待确认 |

**查询建议**:
- 访问 https://cve.mitre.org/ 搜索 "LZMA SDK"
- 访问 https://nvd.nist.gov/ 查询 "7-zip"
- 查看 https://www.7-zip.org/ 安全公告

### 1.2 7-Zip (相关项目) 历史 CVE

LZMA SDK 是 7-Zip 的一部分，历史上 7-Zip 曾有一些漏洞，但大部分影响的是：
- 7-Zip GUI/命令行工具
- 特定归档格式的解析器
- RAR、ZIP 等其他格式处理

**对 OH 的影响**: 较小，因为 OH 仅使用核心的 LZMA 解压 C 代码。

### 1.3 安全更新建议

| 优先级 | 建议 |
|--------|------|
| 高 | 定期监控 https://www.7-zip.org/ 安全公告 |
| 中 | 订阅 7-Zip 安全邮件列表 |
| 低 | 每年同步一次上游版本 |

---

## 2. Patch 安全分析

### 2.1 Patch 文件安全性

| Patch 文件 | 安全影响 | 说明 |
|-----------|----------|------|
| `add-linux-makefile-for-Format7zR.patch` | 无 | 仅添加 Makefile，无运行时代码 |

**结论**: 现有 Patch 不引入新的安全攻击面。

### 2.2 Makefile 安全检查

Patch 中新增的 Makefile 配置：

```makefile
# 安全相关配置
CFLAGS_BASE = -O2 -D_REENTRANT -D_FILE_OFFSET_BITS=64 -fPIC
FLAGS_BASE = -mbranch-protection=standard  # (注释状态)
```

**分析**:
- `-D_FILE_OFFSET_BITS=64`: 支持大文件，防止 32 位截断
- `-fPIC`: 位置无关代码，ASLR 兼容
- `-mbranch-protection=standard`: 分支保护（Patch 中但未启用）

---

## 3. 编译安全选项

### 3.1 BUILD.gn 安全配置

```gn
config("lzma_config_common") {
  cflags = [
    "-Wall",                    # 启用所有警告
    "-Werror",                  # 警告视为错误
    "-Wno-empty-body",
    "-Wno-enum-conversion",
    "-Wno-logical-op-parentheses",
    "-Wno-self-assign",
    "-Wno-implicit-function-declaration",
  ]
}
```

**安全编译选项分析**:

| 选项 | 作用 | 安全价值 |
|------|------|----------|
| `-Werror` | 强制修复所有警告 | 防止潜在 Bug |
| `-Wall` | 启用所有警告 | 发现可疑代码 |
| `-DZ7_AFFINITY_DISABLE` | 禁用 CPU 亲和性 | 避免调度问题 |

### 3.2 ARM64 PAC-RET 保护

```gn
ohos_shared_library("lzma_shared") {
  branch_protector_ret = "pac_ret"
  # ...
}

ohos_source_set("lzma_source_arm64") {
  branch_protector_ret = "pac_ret"
  # ...
}
```

**保护机制**:
- **PAC (Pointer Authentication Code)**: 为返回地址添加签名
- **RET 保护**: 验证返回地址的签名
- **防护攻击**: ROP (Return-Oriented Programming)

**适用架构**: ARM64 (armv8.3-a 及以上)

### 3.3 建议增加的编译选项

| 建议选项 | 作用 | 优先级 |
|----------|------|--------|
| `-mbranch-protection=standard` | 分支保护 | 高 |
| `-fstack-protector-strong` | 栈保护 | 中 |
| `-D_FORTIFY_SOURCE=2` | 强化库函数检查 | 中 |
| `-fPIE` | 位置无关可执行文件 | 低 |

**在 Patch 中已存在但未启用的配置**:
```makefile
# CPP/7zip/7zip_gcc_r.mak
FLAGS_BASE = -mbranch-protection=standard  -march=armv8.5-a
FLAGS_BASE = -mbranch-protection=standard
FLAGS_BASE =
```

建议评估启用 `-mbranch-protection=standard`。

---

## 4. 代码安全分析

### 4.1 代码质量评估

| 指标 | 评估 |
|------|------|
| 代码年龄 | 20+ 年，成熟稳定 |
| 维护状态 | 活跃维护，定期更新 |
| 代码风格 | 一致，有文档 |
| 测试覆盖 | 上游有测试套件 |

### 4.2 潜在风险点

#### 内存操作

LZMA SDK 涉及大量内存操作，潜在风险：
- 解压缓冲区溢出
- 字典大小验证
- 压缩数据恶意构造

**OH 缓解措施**:
- 输入数据大小验证
- 输出缓冲区大小检查
- 使用安全分配器

#### 多线程代码

```c
// MtCoder.c, MtDec.c, LzFindMt.c
```

**风险**: 线程同步问题

**OH 使用**: 仅使用解压功能，解压器线程安全

#### 复杂解析器

```c
// 7zArcIn.c - 7z 归档解析
// XzIn.c - XZ 格式解析
```

**风险**: 格式解析漏洞

**OH 缓解**: 仅用于解析受信任的 unwind 信息

### 4.3 fuzz 测试建议

建议对以下接口进行 fuzz 测试：
- `LzmaDecode()` - 解压入口
- `SzArEx_Open()` - 7z 归档解析
- `XzDec_DecodeToBuf()` - XZ 解码

---

## 5. 运行时安全

### 5.1 攻击面分析

| 攻击面 | 风险等级 | 说明 |
|--------|----------|------|
| 压缩数据输入 | 中 | 恶意构造的压缩数据 |
| API 调用 | 低 | 标准 C API |
| 文件系统 | 低 | 无直接文件操作 |
| 网络 | 无 | 无网络功能 |

### 5.2 使用场景风险评估

#### faultloggerd 使用场景

**流程**:
```
应用崩溃 → 读取 ELF → 解压 unwind 信息 → 生成栈回溯
```

**风险点**:
- ELF 文件来自崩溃应用，可能恶意构造
- unwind 信息可能被篡改

**缓解措施**:
- 仅读取，不执行
- 在独立进程中运行
- 有权限限制

### 5.3 安全沙箱建议

虽然当前风险较低，但建议：
1. 在独立进程中处理崩溃
2. 使用 seccomp 限制系统调用
3. 对输入数据进行验证

---

## 6. 安全升级策略

### 6.1 升级触发条件

| 条件 | 响应 |
|------|------|
| 上游发布安全修复 | 立即升级 |
| 发现严重 CVE | 紧急升级 |
| 常规版本更新 | 计划升级 |

### 6.2 升级检查清单

- [ ] 检查上游安全公告
- [ ] 评估 CVE 影响范围
- [ ] 准备测试用例
- [ ] 测试 faultloggerd 功能
- [ ] 测试多架构 (ARM/ARM64/RISC-V)
- [ ] 性能回归测试
- [ ] 安全扫描

### 6.3 回滚计划

**触发条件**:
- 功能异常
- 性能退化
- 新安全问题

**回滚步骤**:
1. 恢复上一版本
2. 重新构建系统
3. 验证功能

---

## 7. 合规性

### 7.1 许可证

| 项目 | 状态 |
|------|------|
| 许可证类型 | Public Domain |
| 商业使用 | 允许 |
| 修改分发 | 允许 |
| 专利风险 | 低 |

**说明**: Public Domain 许可证无合规限制。

### 7.2 OAT 扫描

`OAT.xml` 配置:
```xml
<policyitem type="compatibility" name="Public domain" path=".*" rule="may"/>
```

**扫描结果**: 通过，无合规问题。

### 7.3 归属说明

虽然 Public Domain 不需要归属，但建议保留：
```
LZMA SDK is written and placed in the public domain by Igor Pavlov.
```

---

## 8. 安全建议总结

### 立即行动

- [ ] 查询并确认当前版本 (25.01) 的所有已知 CVE
- [ ] 评估启用 `-mbranch-protection=standard`

### 短期 (3 个月内)

- [ ] 建立 CVE 监控机制
- [ ] 对关键接口进行 fuzz 测试
- [ ] 评估增加 `-fstack-protector-strong`

### 长期 (6 个月内)

- [ ] 规划升级到最新上游版本
- [ ] 建立自动化安全扫描
- [ ] 完善安全测试用例

---

*最后更新: 2025-02-08*  
*安全评估状态: 待补充 CVE 详细信息*
