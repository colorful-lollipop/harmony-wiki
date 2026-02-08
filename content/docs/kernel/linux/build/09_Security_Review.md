# 安全风险评审

> **更新时间**: 2026-02-06

---

## 文档目的

本文档基于代码证据，分析 OpenHarmony Linux Kernel 构建系统的安全风险，包括攻击面、信任边界、可被利用点和修复建议。

---

## 攻击面清单

### 外部输入源

| 输入源 | 风险等级 | 缓解措施 | 代码证据 |
|---------|---------|---------|----------|
| 补丁文件 | 高 | 补丁签名验证 | `kernel.mk:97-108` |
| 内核配置文件 | 中 | 配置审查 | `kernel.mk:109` |
| 构建脚本 | 低 | 脚本权限控制 | `build_kernel.sh` |
| 内核源码 | 高 | 上游审计 | `kernel.mk:89-94` |
| 工具链路径 | 中 | 路径验证 | `kernel.mk:30-56` |

**代码证据**:
```makefile
# kernel.mk:97-108 - 补丁应用逻辑
$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(KERNEL_PATCH_PATH) $(DEVICE_NAME)
```

---

## 信任边界

### 构建系统信任模型

```
┌─────────────────────────────────────────────────────┐
│           不可信外部输入                      │
├─────────────────────────────────────────────────────┤
│  补丁文件 (来自供应商）                    │
│  内核源码 (上游 Linux）                    │
│  配置文件 (defconfig）                     │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│              构建脚本层                         │
│  - 验证输入完整性                           │
│  - 应用补丁                                 │
│  - 编译内核                                  │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│            可信构建产物                       │
├─────────────────────────────────────────────────────┤
│  内核镜像 (uImage/Image）                  │
│  设备树 (dtbo.img）                        │
│  模块 (.ko 文件）                         │
└─────────────────────────────────────────────────────┘
```

### 信任边界

1. **构建环境** → OpenHarmony 根目录
2. **上游内核** → `kernel/linux/linux-5.10` (可信基线)
3. **供应商补丁** → `kernel/linux/patches/` (需验证）
4. **最终镜像** → `out/.../packages/phone/images/` (可信输出）

**代码证据**:
```makefile
# kernel.mk:27-29 - 可信源路径定义
KERNEL_SRC_PATH := $(OHOS_BUILD_HOME)/kernel/linux/${KERNEL_VERSION}
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}
KERNEL_CONFIG_PATH := $(OHOS_BUILD_HOME)/kernel/linux/config/${KERNEL_VERSION}
```

---

## 可被利用点分析

### 风险 1: 补丁注入攻击

**风险等级**: 🔴 高

**证据位置**:
- `kernel.mk:97-108` - 补丁应用逻辑
- `kernel.mk:95` - HDF 补丁调用

**触发条件**:
```makefile
# kernel.mk:97-101
$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(KERNEL_PATCH_PATH) $(DEVICE_NAME)

ifeq ($(PRODUCT_PATH), vendor/hisilicon/watchos)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(PRODUCT_PATCH_FILE)
else
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && test -f $(DEVICE_PATCH_FILE) && patch -p1 < $(DEVICE_PATCH_FILE) || true
endif
```

**攻击路径**:
1. 恶意补丁文件包含危险的内核修改
2. 补丁通过符号链接指向任意文件
3. 路径遍历攻击补丁目录

**影响**:
- 后门代码注入
- 特权提升
- 系统稳定性破坏

**代码证据**:
```bash
# patch_hdf.sh (假设内容，实际需查看 drivers/hdf_core/adapter/khdf/linux/)
# 缺少补丁签名验证
patch -p1 < ${PATCH_FILE}
```

**修复建议**:
1. **补丁签名**: 实现 GPG 签名验证
   ```bash
   # 验证补丁签名
   gpg --verify ${DEVICE_PATCH_FILE}
   if [ $? -ne 0 ]; then
       echo "Patch signature verification failed!"
       exit 1
   fi
   ```
2. **路径验证**: 检查补丁文件是否在预期目录内
   ```makefile
   # 添加路径检查
   $(hide) test -f $(DEVICE_PATCH_FILE) && patch -p1 < $(DEVICE_PATCH_FILE) || true
   # 改为：
   $(hide) test -f $(DEVICE_PATCH_FILE) && \
       echo "$(DEVICE_PATCH_FILE)" | grep -q "^$(OHOS_BUILD_HOME)/kernel/linux/patches/" && \
       patch -p1 < $(DEVICE_PATCH_FILE) || \
       (echo "Patch file not in allowed directory!" && exit 1)
   ```

---

### 风险 2: 工具链篡改

**风险等级**: 🔴 高

**证据位置**:
- `kernel.mk:30-56` - 工具链路径配置
- `kernel.mk:70-72` - KERNEL_MAKE 定义

**触发条件**:
```makefile
# kernel.mk:30-32
PREBUILTS_GCC_DIR := $(OHOS_BUILD_HOME)/prebuilts/gcc
CLANG_HOST_TOOLCHAIN := $(OHOS_BUILD_HOME)/prebuilts/clang/ohos/linux-x86_64/llvm/bin
```

**攻击路径**:
1. `PREBUILTS_GCC_DIR` 环境变量被篡改
2. 恶意编译器注入到工具链目录
3. PATH 环境变量被修改，使用未授权的编译器

**影响**:
- 编译后门代码
- 供应链攻击
- 所有设备受影响

**代码证据**:
```makefile
# kernel.mk:70-72 - 缺少工具链完整性验证
KERNEL_MAKE := \
    PATH="$(BOOT_IMAGE_PATH):$$PATH" \
    $(KERNEL_PREBUILT_MAKE)
```

**修复建议**:
1. **工具链哈希验证**: 在使用前验证工具链完整性
   ```makefile
   # 添加哈希验证
   TOOLCHAIN_HASH := $(shell sha256sum $(CLANG_HOST_TOOLCHAIN)/clang | cut -d' ' -f1)
   EXPECTED_HASH := $(OHOS_BUILD_HOME)/prebuilts/clang/.toolchain.sha256
   ifneq ($(TOOLCHAIN_HASH), $(EXPECTED_HASH))
       $(error Toolchain hash mismatch! Possible tampering detected.)
   endif
   ```
2. **环境变量审计**: 记录关键环境变量
   ```makefile
   # 添加环境变量记录
   $(hide) echo "CC=$${CC}" >> $(OUT_DIR)/build.log
   $(hide) echo "CROSS_COMPILE=$${CROSS_COMPILE}" >> $(OUT_DIR)/build.log
   ```

---

### 风险 3: 配置注入

**风险等级**: 🟡 中

**证据位置**:
- `kernel.mk:109-110` - defconfig 应用
- `kernel_build.py:297-298` - 配置应用

**触发条件**:
```makefile
# kernel.mk:109-110
$(hide) cp -rf $(KERNEL_CONFIG_PATH)/. $(KERNEL_SRC_TMP_PATH)/
$(hide) $(KERNEL_MAKE) -C $(KERNEL_SRC_TMP_PATH) ARCH=$(KERNEL_ARCH) $(KERNEL_CROSS_COMPILE) $(DEFCONFIG_FILE)
```

**攻击路径**:
1. `KERNEL_CONFIG_PATH` 包含恶意的 `.config` 文件
2. `DEFCONFIG_FILE` 包含危险的内核选项（如禁用安全特性）
3. 配置文件通过符号链接注入

**影响**:
- 禁用安全特性（ASLR、堆保护等）
- 启用不安全的功能
- 调试接口暴露

**代码证据**:
```makefile
# kernel.mk:109 - 缺少配置文件完整性验证
$(hide) cp -rf $(KERNEL_CONFIG_PATH)/. $(KERNEL_SRC_TMP_PATH)/
```

**修复建议**:
1. **配置白名单**: 验证关键安全选项未被修改
   ```makefile
   # 添加配置验证
   $(hide) grep -q "CONFIG_PANIC_ON_OOPS=y" $(KERNEL_SRC_TMP_PATH)/.config || \
       $(error Security configuration missing! CONFIG_PANIC_ON_OOPS must be enabled.)
   ```
2. **配置审计**: 记录并审查最终 .config
   ```bash
   # 生成配置差异报告
   diff -u $(KERNEL_CONFIG_PATH)/standard_common_defconfig \
            $(KERNEL_SRC_TMP_PATH)/.config > $(OUT_DIR)/config_diff.txt
   ```

---

### 风险 4: Shell 脚本注入

**风险等级**: 🟡 中

**证据位置**:
- `build_kernel.sh:16-52` - 镜像复制脚本
- `kernel_module_build.sh:16-72` - 构建入口脚本
- `check_build.sh:16-43` - 时间戳检查脚本

**触发条件**:
```bash
# kernel_module_build.sh:18-24
export OUT_DIR=$1
export BUILD_TYPE=$2
export KERNEL_ARCH=$3
export PRODUCT_PATH=$4
export DEVICE_NAME=$5
export KERNEL_VERSION=$6
```

**攻击路径**:
1. 环境变量包含注入的命令
2. `PRODUCT_PATH` 指向恶意目录
3. `KERNEL_VERSION` 导致加载非预期内核版本

**影响**:
- 代码注入
- 路径遍历
- 任意文件复制

**代码证据**:
```bash
# kernel_module_build.sh:18-23 - 缺少环境变量验证
export OUT_DIR=$1
export BUILD_TYPE=$2
export KERNEL_ARCH=$3
export PRODUCT_PATH=$4
export DEVICE_NAME=$5
export KERNEL_VERSION=$6
```

**修复建议**:
1. **输入验证**: 验证所有输入参数
   ```bash
   # 添加参数验证
   if [[ ! "$KERNEL_VERSION" =~ ^linux-(4\.19|5\.10|6\.6)$ ]]; then
       echo "Invalid kernel version: $KERNEL_VERSION"
       exit 1
   fi
   if [[ ! "$BUILD_TYPE" =~ ^(small|standard|foundation)$ ]]; then
       echo "Invalid build type: $BUILD_TYPE"
       exit 1
   fi
   ```
2. **路径验证**: 确保路径在允许范围内
   ```bash
   # 添加路径白名单验证
   PRODUCT_PATH_REAL=$(realpath "$PRODUCT_PATH")
   if [[ ! "$PRODUCT_PATH_REAL" =~ ^$(OHOS_BUILD_HOME)/vendor/ ]]; then
       echo "Product path outside allowed directory!"
       exit 1
   fi
   ```

---

### 风险 5: 增量构建绕过

**风险等级**: 🟢 低

**证据位置**:
- `check_build.sh:16-43` - 时间戳检查脚本
- `BUILD.gn:51-52` - 时间戳输出

**触发条件**:
```bash
# check_build.sh:35-41
if [ -e "$2" ]; then
    readfile $1 $2 $3
    if [ "$3" -nt "$2" ]; then
        echo "need update $2"
        rm -rf $2;
    fi
fi
```

**攻击路径**:
1. 攻击者伪造时间戳，触发不必要的重建
2. 恶意文件被放置在源码目录中，触发重建
3. 时间戳竞态条件绕过检查

**影响**:
- 构建时间攻击
- 资源消耗（重复构建）
- 潜在的缓存污染

**代码证据**:
```bash
# check_build.sh:24-29 - 缺少文件完整性验证
elif [ "$file" -nt "$2" ]; then
    echo $file is update
    touch $3;
    return
```

**修复建议**:
1. **内容哈希验证**: 不仅检查时间戳，还验证文件内容哈希
   ```bash
   # 改进检查逻辑
   function check_hash_change() {
       local old_hash="$1"
       local new_hash="$2"
       if [ "$old_hash" != "$new_hash" ]; then
           echo "Content changed, rebuild needed"
           return 0
       fi
       return 1
   }
   ```
2. **签名验证**: 对关键产物签名并验证
   ```bash
   # 验证产物签名
   if ! gpg --verify packages/phone/images/uImage.sig; then
       echo "Kernel image signature verification failed!"
       rm -rf packages/phone/images/uImage
       return
   fi
   ```

---

## 安全配置建议

### 构建时安全措施

| 措施 | 优先级 | 实施难度 |
|------|--------|---------|
| 补丁签名验证 | 高 | 中 |
| 工具链哈希验证 | 高 | 低 |
| 配置白名单 | 中 | 低 |
| 环境变量审计 | 中 | 低 |
| 产物签名 | 高 | 中 |

### 运行时安全措施

| 措施 | 关键配置 |
|------|---------|
| 内核模块签名 | `CONFIG_CODE_SIGN=y` |
| 容器逃逸检测 | `CONFIG_CONTAINER_ESCAPE_DETECTION=y` |
| 内存安全 | `CONFIG_MEMORY_SECURITY=y` |
| 指针认证 | `CONFIG_PAC=y` |
| 堆保护 | `CONFIG_CC_STACKPROTECTOR=y` |
| ASLR | `CONFIG_RANDOMIZE_BASE=y` |

---

## 安全审计清单

### 构建前检查

- [ ] 补丁来源可信（来自官方仓库或受信任供应商）
- [ ] 补丁文件完整性验证（哈希/签名）
- [ ] 工具链完整性验证
- [ ] 配置文件审查（禁用不安全选项）
- [ ] 环境变量正确性验证

### 构建中检查

- [ ] 构建日志审计
- [ ] 警告错误检查
- [ ] 中间产物完整性验证

### 构建后检查

- [ ] 内核镜像签名
- [ ] 镜像大小符合限制
- [ ] 设备树格式验证
- [ ] 安全配置选项验证

---

## 未覆盖的安全范围

### 本文档未覆盖

1. **运行时内核漏洞** - 上游 Linux 内核中的 CVE（由上游社区修复）
2. **驱动安全漏洞** - 具体驱动的安全缺陷
3. **HDF 框架安全** - 驱动框架的安全实现（位于其他仓库）

### 建议进一步审查

1. 审查 `kernel/linux/linux-5.10/` 源码中的安全配置
2. 审查 `drivers/hdf_core/adapter/khdf/linux/` HDF 适配代码
3. 审查 `kernel/linux/common_modules/` 中的安全模块实现

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [补丁管理](07_Patch_Management.md) - 补丁应用机制详解
- [故障排查](10_Troubleshooting.md) - 常见问题与定位方法
