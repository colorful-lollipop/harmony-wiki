# 补丁管理

> **更新时间**: 2026-02-06

---

## 文档目的

本文档详细说明 OpenHarmony Linux Kernel 的补丁管理机制，包括 HDF 补丁、设备驱动补丁、应用顺序和补丁规则。

---

## 补丁类型

OpenHarmony Linux Kernel 构建系统支持多种补丁类型，按特定顺序应用：

```
补丁应用顺序
    │
    ├─→ 1. HDF 通用补丁
    ├─→ 2. 设备特定补丁 (DEVICE_PATCH_FILE)
    ├─→ 3. 产品补丁 (PRODUCT_PATCH_FILE)
    ├─→ 4. Small 系统补丁 (SMALL_PATCH_FILE)
    └─→ 5. 统一集合补丁 (UNIFIED_COLLECTION_PATCH_FILE)
```

**代码证据**:
```makefile
# kernel.mk:95-108
$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(KERNEL_PATCH_PATH) $(DEVICE_NAME)

ifeq ($(PRODUCT_PATH), vendor/hisilicon/watchos)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(PRODUCT_PATCH_FILE)
else
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && test -f $(DEVICE_PATCH_FILE) && patch -p1 < $(DEVICE_PATCH_FILE) || true
endif

ifneq ($(findstring $(BUILD_TYPE), small),)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(SMALL_PATCH_FILE)
endif

ifeq ($(UNIFIED_COLLECTION_PATCH_FILE), $(wildcard $(UNIFIED_COLLECTION_PATCH_FILE)))
	$(hide) $(UNIFIED_COLLECTION_PATCH_FILE) $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(DEVICE_NAME) $(KERNEL_VERSION)
endif
```

---

## 1. HDF 通用补丁

### 路径和命名

```
kernel/linux/patches/
├── linux-4.19/
│   └── common_patch/
│       └── hdf.patch
└── linux-5.10/
    └── common_patch/
        └── hdf.patch
```

**代码证据**:
```makefile
# kernel.mk:28
KERNEL_PATCH_PATH := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}
```

### HDF 补丁内容

HDF (Hardware Driver Foundation) 补丁包含：

- **HDF 核心框架** - 驱动加载、设备管理、服务管理
- **驱动模型** - 网络设备、传感器、输入设备、平台设备
- **驱动能力库** - IO 通信、电源管理、总线抽象（I2C/SPI/UART）
- **驱动工具** - HDI 接口转换、驱动配置（HCS）、编译工具
- **驱动接口** - 标准化驱动接口（HDI）

### HDF 补丁应用脚本

**脚本路径**: `drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh`

**调用方式**:
```makefile
# kernel.mk:95
$(hide) $(OHOS_BUILD_HOME)/drivers/hdf_core/adapter/khdf/linux/patch_hdf.sh \
    $(OHOS_BUILD_HOME) \
    $(KERNEL_SRC_TMP_PATH) \
    $(KERNEL_PATCH_PATH) \
    $(DEVICE_NAME)
```

**参数说明**:

| 参数 | 位置 | 说明 |
|------|------|--------|
| `$1` | OHOS_BUILD_HOME | OpenHarmony 根目录 |
| `$2` | KERNEL_SRC_TMP_PATH | 内核源码临时目录 |
| `$3` | KERNEL_PATCH_PATH | 补丁目录 |
| `$4` | DEVICE_NAME | 设备名称 |

---

## 2. 设备特定补丁

### 路径和命名

```
kernel/linux/patches/
├── linux-4.19/
│   ├── hispark_taurus_patch/
│   │   └── hispark_taurus.patch
│   └── rk3568_patch/
│       └── rk3568.patch
└── linux-5.10/
    ├── hispark_taurus_patch/
    │   └── hispark_taurus.patch
    └── rk3568_patch/
        ├── kernel.patch      # RK3568 内核特定补丁
        └── hdf.patch        # RK3568 HDF 定制补丁
```

**代码证据**:
```makefile
# kernel.mk:76-77
DEVICE_PATCH_DIR := $(OHOS_BUILD_HOME)/kernel/linux/patches/${KERNEL_VERSION}/$(DEVICE_NAME)_patch
DEVICE_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME).patch
```

### 支持的设备

| 设备名称 | 内核版本 | 补丁路径 | 芯片 |
|---------|---------|---------|--------|
| hispark_taurus | linux-4.19, linux-5.10 | `hispark_taurus_patch/hispark_taurus.patch` | Hi3516D V300 |
| rk3568 | linux-5.10 | `rk3568_patch/kernel.patch`, `rk3568_patch/hdf.patch` | RK3568 |

### 设备补丁内容

**内核特定补丁** (`kernel.patch`):
- 设备树 (DTS) 配置
- 芯片特定驱动
- 设备初始化代码
- 外设驱动适配

**HDF 定制补丁** (`hdf.patch`):
- 芯片平台 HDF 适配
- 特定驱动能力实现
- 设备资源配置（HCS）

### 设备补丁应用

```makefile
# kernel.mk:97-101
ifeq ($(PRODUCT_PATH), vendor/hisilicon/watchos)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(PRODUCT_PATCH_FILE)
else
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && test -f $(DEVICE_PATCH_FILE) && patch -p1 < $(DEVICE_PATCH_FILE) || true
endif
```

**注意**: `|| true` 确保补丁不存在时不失败

---

## 3. 产品补丁

### 用途

产品补丁用于特定产品的定制化需求，通常由产品厂商提供。

**路径示例**:
```
vendor/hisilicon/watchos/patches/
└── hispark_phoenix.patch
```

**代码证据**:
```makefile
# kernel.mk:78
PRODUCT_PATCH_FILE := $(OHOS_BUILD_HOME)/vendor/hisilicon/watchos/patches/$(DEVICE_NAME).patch
```

### 产品补丁应用

```makefile
# kernel.mk:97-98
ifeq ($(PRODUCT_PATH), vendor/hisilicon/watchos)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(PRODUCT_PATCH_FILE)
endif
```

---

## 4. Small 系统补丁

### 路径和命名

```
kernel/linux/patches/${KERNEL_VERSION}/${DEVICE_NAME}_patch/
└── ${DEVICE_NAME}_${BUILD_TYPE}.patch
```

**示例**:
```
kernel/linux/patches/linux-5.10/hispark_taurus_patch/
├── hispark_taurus.patch
├── hispark_taurus_small.patch
└── hispark_taurus_standard.patch
```

**代码证据**:
```makefile
# kernel.mk:79
SMALL_PATCH_FILE := $(DEVICE_PATCH_DIR)/$(DEVICE_NAME)_$(BUILD_TYPE).patch
```

### Small 系统补丁应用

```makefile
# kernel.mk:103-105
ifneq ($(findstring $(BUILD_TYPE), small),)
	$(hide) cd $(KERNEL_SRC_TMP_PATH) && patch -p1 < $(SMALL_PATCH_FILE)
endif
```

**条件**: 仅在 `BUILD_TYPE` 为 `small` 时应用

---

## 5. 统一集合补丁

### 路径

```
kernel/linux/common_modules/ucollection/
└── apply_ucollection.sh
```

**代码证据**:
```makefile
# kernel.mk:82
UNIFIED_COLLECTION_PATCH_FILE := ${OHOS_BUILD_HOME}/kernel/linux/common_modules/ucollection/apply_ucollection.sh
```

### 统一集合补丁应用

```makefile
# kernel.mk:107-108
ifeq ($(UNIFIED_COLLECTION_PATCH_FILE), $(wildcard $(UNIFIED_COLLECTION_PATCH_FILE)))
	$(hide) $(UNIFIED_COLLECTION_PATCH_FILE) $(OHOS_BUILD_HOME) $(KERNEL_SRC_TMP_PATH) $(DEVICE_NAME) $(KERNEL_VERSION)
endif
```

**条件**: 补丁脚本存在时应用

**功能**: 将公共模块（如安全模块、容器逃逸检测等）应用到内核中

---

## 补丁规则

### patch 命令参数

```bash
patch -p1 < patch_file
```

| 参数 | 说明 |
|------|--------|
| `-p1` | 去除补丁中的第一层目录前缀 |
| `< patch_file` | 从标准输入读取补丁 |
| `|| true` | 如果补丁文件不存在，忽略错误（不中断构建） |

### 补丁格式要求

1. ** unified diff 格式**:
```diff
--- a/file.c
+++ b/file.c
@@ -1,3 +1,4 @@
 context line
 context line
-new line
 context line
```

2. **元数据**:
```diff
补丁开头应包含元数据说明：
From: <author email>
Date: <timestamp>
Subject: [PATCH] <patch description>
```

3. **路径前缀**: `a/` 表示旧版本，`b/` 表示新版本

---

## 补丁冲突处理

### 冲突场景

1. **基础内核更新** - 上游内核更改导致补丁失效
2. **补丁重叠** - 多个补丁修改相同文件
3. **版本兼容** - 不同内核版本的补丁不兼容

### 冲突检测

```bash
# 使用 patch --dry-run 检测冲突
patch --dry-run -p1 < patch_file

# 使用 git apply 检测并尝试自动合并
git apply --check patch_file
```

### 冲突解决

1. **手动解决**:
   - 编辑冲突文件（标记为 `<<<<<<<` 和 `>>>>>>>`）
   - 删除标记，保留正确的代码
   - 测试编译

2. **更新补丁**:
   - 使用 `git diff` 生成新补丁
   - 更新补丁文件的行号

---

## 创建新补丁

### 步骤

1. **修改内核源码**:
```bash
cd ${KERNEL_SRC_TMP_PATH}
# 编辑需要修改的文件
vim drivers/xxx/xxx.c
```

2. **生成补丁**:
```bash
# 使用 git 生成补丁
git add drivers/xxx/xxx.c
git commit -m "Add support for XXX"
git format-patch -1 HEAD > new_feature.patch

# 或使用 diff（如果不使用 git）
diff -Naur old/ new/ > new_feature.patch
```

3. **放置补丁**:
```bash
# 设备补丁
cp new_feature.patch kernel/linux/patches/${KERNEL_VERSION}/${DEVICE_NAME}_patch/

# 或 HDF 补丁
cp new_feature.patch kernel/linux/patches/${KERNEL_VERSION}/common_patch/hdf.patch
```

---

## 补丁验证

### 验证编译

应用补丁后验证内核可以编译：

```bash
cd ${KERNEL_SRC_TMP_PATH}
make ARCH=${KERNEL_ARCH} ${DEFCONFIG_FILE}
make ARCH=${KERNEL_ARCH} -j64
```

### 验证功能

1. 检查补丁修改的文件是否包含在构建产物中
2. 验证相关内核模块是否生成
3. 检查内核配置项是否正确启用

### 回滚补丁

如果补丁导致问题，可以回滚：

```bash
cd ${KERNEL_SRC_TMP_PATH}
patch -R -p1 < patch_file
```

---

## 常见问题

### 补丁应用失败

**错误信息**: `patch: **** patch does not apply`

**原因**:
1. 补丁与当前内核版本不匹配
2. 文件已通过其他方式修改
3. 补丁格式错误

**解决方案**:
1. 验证内核版本是否正确
2. 使用 `patch --dry-run` 检查问题
3. 检查 `patch_file` 路径是否正确

### 补丁顺序错误

**问题**: 后面的补丁覆盖前面的补丁

**解决方案**:
1. 理解补丁依赖关系
2. 调整 `kernel.mk` 中的补丁应用顺序
3. 合并依赖的补丁为一个补丁

---

## 相关文档

- [项目概览](01_Project_Overview.md) - 项目定位与核心能力
- [构建系统架构](03_Build_System_Architecture.md) - 构建流程详解
- [构建脚本](05_Build_Scripts.md) - Shell 脚本详细分析
- [内核配置](06_Kernel_Configuration.md) - defconfig 管理机制
