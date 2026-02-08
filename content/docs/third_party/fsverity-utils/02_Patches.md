# fsverity-utils Patch 分析

## 2.1 Patch 概述

### Patch 清单

**结论**：该库**没有任何** OpenHarmony 特定 Patch 文件。

| Patch 文件 | 修改文件 | 修改函数 | 修改目的 |
|-----------|----------|----------|----------|
| 无 | - | - | - |

### 搜索结果

```bash
$ find . -name "*.patch" -o -name "patches" -type d
# 无匹配结果
```

---

## 2.2 分析说明

### 为什么没有 Patch？

fsverity-utils 在 OpenHarmony 中采用**干净导入**策略，原因如下：

#### 1. 上游代码的平台无关设计

fsverity-utils 的设计目标就是跨平台，主要特点：

```
┌─────────────────────────────────────────────────────────┐
│              fsverity-utils 架构                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  用户空间代码                                           │
│  ├── lib/compute_digest.c     ← 纯 C，标准 API          │
│  ├── lib/enable.c             ← 标准 ioctl 调用         │
│  ├── lib/hash_algs.c          ← OpenSSL 抽象层          │
│  ├── lib/sign_digest.c        ← OpenSSL PKCS#7         │
│  └── lib/utils.c              ← 标准 C 库函数           │
│                                                         │
│  平台相关部分                                           │
│  ├── common/common_defs.h     ← 编译器属性定义           │
│  ├── common/fsverity_uapi.h   ← Linux uapi 头文件       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

#### 2. 无需平台特定修改

| 功能 | 需要的修改 | fsverity-utils 情况 |
|------|-----------|---------------------|
| 文件 I/O | 平台抽象 | 使用标准 POSIX，由调用者提供 fd |
| 哈希计算 | 算法实现 | 通过 OpenSSL，提供跨平台支持 |
| ioctl 调用 | 系统调用 | 使用标准 Linux API |
| 内存管理 | 平台 API | 使用标准 C 库 |

#### 3. OH 适配仅在构建层

所有 OH 适配通过 BUILD.gn 完成，无需修改源代码：

```gn
# BUILD.gn 中的适配
ohos_shared_library("libfsverity_utils") {
  sources = [ ... ]
  external_deps = [ "openssl:libcrypto_shared" ]
  configs = [ ":common_config" ]
}
```

---

## 2.3 替代方案

### BUILD.gn 配置替代 Patch

虽然无源代码 Patch，但 OH 适配通过以下方式实现：

| 适配项 | 实现方式 | 文件 |
|--------|----------|------|
| 库目标定义 | GN 目标 | BUILD.gn |
| 编译选项 | config() | BUILD.gn |
| 依赖声明 | external_deps | BUILD.gn |
| 头文件暴露 | public_configs | BUILD.gn |

### 配置示例

```gn
# 公共头文件配置
config("libfsverity_public_config") {
  include_dirs = [
    "include",
    "common",
  ]
}

# 编译选项配置
config("common_config") {
  cflags = [
    "-Wall",
    "-Wundef",
    "-Wdeclaration-after-statement",
    "-Wmissing-field-initializers",
    "-Wmissing-prototypes",
    "-Wstrict-prototypes",
    "-Wunused-parameter",
    "-Wvla",
    "-Wno-deprecated-declarations",
  ]
}
```

---

## 2.4 升级上游版本指南

### 升级流程

由于无 OH Patch，升级流程相对简单：

```
┌─────────────────────────────────────────────────────────┐
│                   升级上游版本流程                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Step 1: 准备                                           │
│  ├── 备份当前目录内容                                    │
│  ├── 下载新版本源码                                      │
│  └── 记录当前版本信息                                    │
│                                                         │
│  Step 2: 替换源码                                       │
│  ├── 替换 lib/ 目录源文件                                │
│  ├── 替换 include/ 头文件                                │
│  ├── 替换 common/ 定义文件                               │
│  └── 更新 programs/ (如需要)                            │
│                                                         │
│  Step 3: 验证 BUILD.gn                                  │
│  ├── 检查 sources 列表                                   │
│  ├── 验证 external_deps                                  │
│  └── 确认 configs 配置                                   │
│                                                         │
│  Step 4: 更新元数据                                      │
│  ├── 更新 README.OpenSource                             │
│  ├── 更新 bundle.json                                   │
│  └── 更新本文档版本信息                                   │
│                                                         │
│  Step 5: 测试验证                                       │
│  ├── 编译验证                                           │
│  ├── 运行单元测试                                       │
│  └── 集成测试                                           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 注意事项

#### 必须保留的文件

| 文件 | 原因 |
|------|------|
| `BUILD.gn` | OH 构建配置 |
| `bundle.json` | OH 组件描述 |
| `README.OpenSource` | OH 开源声明 |

#### 需要验证的变更

```bash
# 1. 检查新版本 API 变更
git diff --stat <新版本>

# 2. 验证 BUILD.gn 兼容性
#    - 新增源文件是否需要加入 sources
#    - 依赖是否有变化
#    - 配置选项是否有新增

# 3. 检查头文件变更
#    - 公共 API 是否有删除
#    - 结构体是否有不兼容修改
```

### 常见问题

| 问题 | 解决方案 |
|------|----------|
| 新增源文件 | 更新 BUILD.gn 中的 sources 列表 |
| 新增依赖 | 添加 external_deps |
| API 废弃 | 检查 05_API_Differences.md |
| 编译选项变更 | 更新 common_config |

---

## 2.5 未来可能的 Patch 需求

### 潜在的 OH 特定功能

虽然当前无 Patch，但以下场景可能需要：

| 场景 | 描述 | 可能性 |
|------|------|--------|
| OH 专用错误码 | 定义 OH 特定的错误处理 | 低 |
| 性能优化 | 针对 OH 平台的优化 | 中 |
| 新哈希算法 | 添加国产哈希算法 | 中 |
| 硬件加速 | TEE/SE 加速支持 | 低 |

### Patch 预留策略

如果未来需要添加 Patch，建议：

1. **使用标准 Patch 格式**
   ```
   patches/
   ├── 0001-description.patch
   ├── 0002-description.patch
   └── ...
   ```

2. **Patch 命名规范**
   ```
   <序号>-<功能简述>.patch
   例如：0001-add-ohos-error-handling.patch
   ```

3. **维护变更日志**
   ```
   patches/
   ├── 0001-xxx.patch
   └── CHANGES   ← 记录每个 Patch 的变更原因
   ```

---

## 2.6 回归风险评估

### 无 Patch 的优势

| 风险项 | 评估 | 说明 |
|--------|------|------|
| 上游合并冲突 | 低 | 源码完全同步 |
| 构建失败 | 低 | BUILD.gn 独立维护 |
| API 兼容性 | 低 | 直接使用上游 API |
| 安全更新 | 低 | 可快速合并上游安全修复 |

### 升级风险矩阵

| 升级类型 | 风险 | 缓解措施 |
|----------|------|----------|
| 补丁版本 | 极低 | 通常只是 Bug 修复 |
| 次要版本 | 低 | 检查 API 兼容性 |
| 主要版本 | 中 | 可能有不兼容变更 |

---

## 2.7 总结

### Patch 状态

| 状态 | 数量 |
|------|------|
| OH 特有 Patch | 0 |
| 上游已接受的 Patch | 0 |
| 待提交上游的 Patch | 0 |

### 维护建议

1. **定期同步上游**：每月检查一次上游更新
2. **安全优先**：安全修复优先合并
3. **功能谨慎**：新功能需评估 OH 需求
4. **测试覆盖**：确保测试用例覆盖关键功能

---

## 2.8 参考

### 升级检查清单

```markdown
## 上游版本升级检查清单

### 升级前
- [ ] 备份当前目录
- [ ] 阅读上游 CHANGELOG
- [ ] 检查 API 变更公告
- [ ] 通知依赖模块开发者

### 升级中
- [ ] 替换源文件
- [ ] 验证 BUILD.gn
- [ ] 更新 README.OpenSource
- [ ] 更新 bundle.json

### 升级后
- [ ] 编译测试
- [ ] 单元测试
- [ ] 集成测试
- [ ] 更新本文档
```

### 相关文档

- [03_Build_Integration.md](03_Build_Integration.md) - 构建配置
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用场景
- [06_Security.md](06_Security.md) - 安全考虑
