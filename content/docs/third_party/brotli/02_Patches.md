# Patch 详细分析

## Patch 概述

**本节关键结论**：**Brotli 库在 OpenHarmony 中没有存放任何代码 Patch**

这是一个重要的设计决策，反映了 Brotli 库的良好跨平台特性和 OH 的补丁管理策略。

---

## Patch 策略说明

### OH 的 Patch 管理模式

OpenHarmony 采用**集中式 Patch 管理**策略，对于 Brotli 这样的库：

| 策略 | 说明 |
|-----|------|
| **Patch 存放位置** | 不在本仓库存放 `.patch` 文件 |
| **配置管理** | 通过 `productdefine_common` 项目集中管理 |
| **PR 关联** | https://gitee.com/openharmony/productdefine_common/pulls/863 |

### patches.json 配置文件

```json
{
    "patches":[
        {
            "project":"productdefine_common",
            "path":"productdefine/common",
            "pr_url":"https://gitee.com/openharmony/productdefine_common/pulls/863"
        }
    ]
}
```

这个配置文件的作用是：
1. **声明依赖关系**：告知构建系统该库的 Patch 在其他仓库
2. **版本追踪**：关联到具体的产品定义 PR
3. **配置同步**：确保 OH 特定配置能被正确应用

---

## 为什么不需要 Patch？

### 1. 代码跨平台特性

Brotli 是用纯 C 语言编写的压缩库，具有以下特性：

- ✅ **无 OS 依赖**：代码不直接调用任何操作系统 API
- ✅ **自包含**：所有平台抽象都在 `c/common/platform.c` 中实现
- ✅ **标准 C**：遵循 C99 标准，兼容性好

### 2. 现有文件结构

```
brotli/
├── c/
│   ├── common/         # 公共组件
│   │   ├── platform.c  # 平台抽象层（检测编译环境）
│   │   └── platform.h
│   ├── dec/           # 解码器（完全平台无关）
│   ├── enc/           # 编码器（完全平台无关）
│   └── include/       # 公共 API 头文件
├── BUILD.gn           # OH 构建配置
└── patches.json       # Patch 策略声明
```

### 3. 构建配置通用性

BUILD.gn 中的编译选项都是跨平台的：

```gn
config("brotli_config") {
  include_dirs = [ "c/include" ]
  cflags = [
    "-Wno-deprecated-declarations",  # 通用警告抑制
    "-D_GNU_SOURCE",                # GNU 扩展（兼容 POSIX）
    "-D_HAS_EXCEPTIONS=0",           # C 异常控制（通用）
    "-DHAVE_CONFIG_H",              # 构建配置检测（通用）
    "-Wno-macro-redefined",         # 警告抑制（通用）
  ]
}
```

---

## OH 特定配置（非 Patch）

虽然不需要代码 Patch，但 OH 还是做了一些构建层面的适配：

### PAC 指针认证

```gn
ohos_shared_library("brotli_shared") {
  branch_protector_ret = "pac_ret"  # ARM PAC 安全特性
  # ...
}
```

**说明**：
- 这是 GN 构建系统的配置选项
- 不是代码修改，而是链接时安全加固
- 用于 ARM 平台的 Return-Oriented Programming (ROP) 攻击防护

### 安装目标配置

```gn
install_images = [ "updater", "system" ]
```

- 指定库被安装到 `updater` 和 `system` 分区
- 这是 OH 特有的文件系统布局配置

---

## Patch 维护建议

### 1. Patch 升级策略

如果未来需要添加 OH 特有功能，建议：

| 策略 | 适用场景 |
|-----|---------|
| **推向上游** | 通用的 Bugfix 和优化 |
| **本地 Patch** | OH 特有的功能需求 |
| **配置层修改** | 编译选项、构建配置 |

### 2. 版本升级检查清单

当升级 Brotli 上游版本时：

- [ ] 检查上游是否有重大 API 变更
- [ ] 验证 `patches.json` 中的配置 PR 是否需要更新
- [ ] 测试 curl 对 Brotli 的集成是否正常
- [ ] 验证 PAC 配置仍然有效
- [ ] 检查是否有新的 CVE 需要关注

### 3. 监控上游变更

建议关注的变更类型：

| 变更类型 | 影响 | 处理方式 |
|---------|-----|---------|
| 安全修复 | 高 | 优先同步 |
| API 变更 | 中 | 评估兼容性 |
| 性能优化 | 低 | 按需同步 |
| 新功能 | 低 | 按需同步 |

---

## 总结

| 项目 | 状态 | 说明 |
|-----|------|------|
| **代码 Patch** | ✅ 无 | 代码跨平台，无需修改 |
| **配置 Patch** | ⚠️ 外部 | 通过 productdefine_common 管理 |
| **构建适配** | ✅ 已完成 | GN 构建配置完整 |
| **维护复杂度** | 低 | 升级时只需关注配置同步 |

---

## 相关资源

- **productdefine_common PR 863**：https://gitee.com/openharmony/productdefine_common/pulls/863
- **patches.json 路径**：`//third_party/brotli/patches/patches.json`
- **上游仓库**：http://github.com/google/brotli
