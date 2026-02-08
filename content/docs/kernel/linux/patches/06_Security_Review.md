# 安全评审

本文档对 `kernel_linux_patches` 仓库进行安全风险评审，识别潜在安全风险并提供修复建议。

## 评审范围

### 已审查内容

| 类别 | 范围 | 状态 |
|------|------|------|
| 补丁代码 | HDF 补丁、SOC 补丁 | 已审查 |
| 配置文件 | defconfig 配置文件 | 已审查 |
| 构建脚本 | kernel.mk 构建流程 | 已审查 |
| 预编译文件 | 头文件和脚本 | 已审查 |

### 未审查内容

| 类别 | 原因 | 建议 |
|------|------|------|
| Linux 内核源码 | 超出本仓库范围 | 参考内核安全公告 |
| 交叉编译工具链 | 外部依赖 | 使用官方版本 |
| 运行时环境 | 部署时考虑 | 安全加固指南 |

## 威胁模型

### 攻击面分析

```
┌─────────────────────────────────────────────────────────────┐
│                    攻击面                                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  外部输入              敏感操作              输出/结果       │
│     │                     │                     │          │
│     ▼                     ▼                     ▼          │
│  ┌─────────┐         ┌─────────┐         ┌─────────┐      │
│  │补丁文件 │ ─────▶ │ 代码注入 │ ─────▶ │ 内核镜像 │      │
│  └─────────┘         └─────────┘         └─────────┘      │
│     │                     │                     │          │
│     ▼                     ▼                     ▼          │
│  配置参数            权限提升            设备状态          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 信任边界

| 边界 | 组件 | 风险级别 |
|------|------|----------|
| 外部输入 | 补丁文件 | 中 |
| 构建环境 | 编译工具链 | 中 |
| 部署环境 | 目标设备 | 高 |

## 识别的风险

### 风险 1: 补丁文件验证不足

**证据**: `patch_hdf.sh` 脚本路径：`kernel/linux/build/kernel.mk`

```makefile
$(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) \
    $(KERNEL_SRC_TMP_PATH) \
    $(KERNEL_PATCH_PATH) \
    $(DEVICE_NAME)
```

**触发条件**:
1. 攻击者替换补丁文件
2. 使用未验证的补丁源

**影响**:
- 内核代码注入
- 恶意代码执行
- 系统权限提升

**修复建议**:
```bash
# 1. 添加补丁文件签名验证
gpg --verify hdf.patch.sig hdf.patch

# 2. 使用 HTTPS 下载补丁
curl -fsSL https://secure.source/patch | patch -p1

# 3. 检查补丁来源
verify_patch_source() {
    local patch_path="$1"
    local expected_hash=$(cat "${patch_path}.sha256")
    local actual_hash=$(sha256sum "${patch_path}" | cut -d' ' -f1)
    [ "$expected_hash" = "$actual_hash" ]
}
```

### 风险 2: 路径遍历漏洞

**证据**: kernel.mk 中的变量使用

```makefile
DEVICE_PATCH_DIR := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}/$(DEVICE_NAME)_patch
DEVICE_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME).patch
```

**触发条件**:
1. `DEVICE_NAME` 包含恶意路径
2. 构建环境变量被篡改

**影响**:
- 任意文件写入
- 构建产物篡改

**修复建议**:
```makefile
# 1. 白名单验证设备名称
VALID_DEVICES := hispark_taurus rk3568 imx8mm
ifeq ($(findstring $(DEVICE_NAME),$(VALID_DEVICES)),)
$(error Invalid device name: $(DEVICE_NAME))
endif

# 2. 规范化路径
realpath $(DEVICE_PATCH_DIR)
```

### 风险 3: 内核配置安全选项缺失

**证据**: 配置文件位置：`config/{version}/arch/arm/configs/`

**触发条件**:
1. 未启用安全相关内核选项
2. 使用过于宽松的配置

**影响**:
- 内核漏洞利用
- 信息泄露

**修复建议**:
在 defconfig 中启用以下选项：

```kconfig
# 内存保护
CONFIG_STRICT_DEVMEM=y          # 限制 /dev/mem 访问
CONFIG_IO_STRICT_DEVMEM=y       # 严格 I/O 内存访问
CONFIG_HARDENED_USERCOPY=y      # 用户空间拷贝检查
CONFIG_RANDOMIZE_BASE=y         # KASLR 基础随机化

# 权限控制
CONFIG_STRICT_MODULE_RWX=y      # 模块读写执行保护
CONFIG_MODULE_SIG=y             # 模块签名验证
CONFIG_MODULE_SIG_FORCE=y       # 强制模块签名

# 安全特性
CONFIG_STACKPROTECTOR=y         # 栈保护
CONFIG_STACKPROTECTOR_STRONG=y  # 强栈保护
CONFIG_REFCOUNT_FULL=y          # 引用计数检查
```

### 风险 4: 构建产物完整性

**证据**: 产物路径：`out/kernel/`

**触发条件**:
1. 构建产物被篡改
2. 中间文件被替换

**影响**:
- 恶意内核镜像部署
- 系统被植入后门

**修复建议**:
```bash
# 1. 构建产物签名
sign_kernel_image() {
    local image="$1"
    local sig="${image}.sig"
    openssl dgst -sha256 -sign private.key -out "$sig" "$image"
}

# 2. 构建时验证
verify_build() {
    local build_id="$1"
    local expected_hash=$(get_expected_hash "$build_id")
    local actual_hash=$(sha256sum "$KERNEL_IMAGE" | cut -d' ' -f1)
    [ "$expected_hash" = "$actual_hash" ]
}
```

### 风险 5: 预编译头文件安全

**证据**: 头文件位置：`linux-{version}/prebuilts/usr/include/`

**触发条件**:
1. 预编译头文件被篡改
2. 恶意头文件包含

**影响**:
- 编译时代码注入
- 供应链攻击

**修复建议**:
```bash
# 1. 验证头文件完整性
verify_headers() {
    local header_dir="$1"
    find "$header_dir" -name "*.h" -exec sha256sum {} \; > headers.sha256
    sha256sum -c headers.sha256
}

# 2. 使用可信来源
HEADERS_SOURCE="https://gitee.com/openharmony/kernel_linux_patches/raw/master/linux-{version}/prebuilts/"
```

## 安全最佳实践

### 开发阶段

| 实践 | 说明 |
|------|------|
| 补丁审查 | 所有补丁必须经过代码审查 |
| 安全测试 | 使用静态分析工具检查补丁 |
| 版本锁定 | 使用特定版本的内核和工具链 |

### 构建阶段

| 实践 | 说明 |
|------|------|
| 环境隔离 | 在隔离环境中构建 |
| 签名验证 | 验证所有输入文件 |
| 产物签名 | 对最终产物签名 |

### 部署阶段

| 实践 | 说明 |
|------|------|
| 安全启动 | 启用 UEFI 安全启动 |
| 镜像验证 | 部署前验证镜像签名 |
| 完整性监控 | 运行时完整性监控 |

## 安全检查清单

### 补丁安全检查

- [ ] 补丁来源可信
- [ ] 补丁文件签名验证
- [ ] 补丁不包含敏感信息
- [ ] 补丁经过代码审查

### 配置安全检查

- [ ] 启用内核安全选项
- [ ] 禁用不必要的功能
- [ ] 配置权限控制
- [ ] 启用审计日志

### 构建安全检查

- [ ] 构建环境安全
- [ ] 工具链可信
- [ ] 产物完整性验证
- [ ] 构建日志完整

## 相关文档

- [补丁分析](./04_Patches_Analysis.md)
- [构建系统](./05_Build_System.md)
- [配置详情](./appendix/Config_Details.md)
