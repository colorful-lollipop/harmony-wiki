# 常见问题与排查

> **阅读时间**: 15 分钟 | **目标**: 快速定位和解决开发中的常见问题

---

## 概述

本章节汇总 TA 开发过程中常见的问题、错误信息和解决方案。

**问题分类**:
- 🔧 编译问题
- 🔐 签名问题
- ⚙️ 配置问题
- 🐛 运行时问题

**使用建议**:
1. 根据错误信息查找对应问题
2. 按照解决方案步骤操作
3. 如果问题未解决，参考相关文档

---

## 问题快速索引

| 错误类型 | 问题描述 | 章节 |
|---------|---------|------|
| 🔧 编译 | 找不到头文件 | [问题 1](#问题-1-找不到头文件) |
| 🔧 编译 | 工具链路径未配置 | [问题 2](#问题-2-工具链路径未配置) |
| 🔧 编译 | Python 依赖缺失 | [问题 3](#问题-3-python-依赖缺失) |
| 🔧 编译 | 64位/32位架构配置错误 | [问题 4](#问题-4-6432位架构配置错误) |
| 🔐 签名 | 密钥不匹配 | [问题 5](#问题-5-签名失败---密钥不匹配) |
| 🔐 签名 | ELF 格式错误 | [问题 6](#问题-6-签名失败---elf-格式错误) |
| 🔐 签名 | 内存基线检查失败 | [问题 7](#问题-7-签名失败---内存基线检查失败) |
| 🔐 签名 | UUID 格式错误 | [问题 8](#问题-8-签名失败---uuid-格式错误) |
| ⚙️ 配置 | configs.xml 解析错误 | [问题 11](#问题-11-configsxml-解析错误) |
| ⚙️ 配置 | 服务名称无效 | [问题 12](#问题-12-服务名称无效) |
| 🐛 运行时 | CA 连接失败 | [问题 9](#问题-9-ca-连接-ta-失败) |
| 🐛 运行时 | TA 返回错误码 | [问题 10](#问题-10-ta-返回错误码) |

---

### 问题 1: 找不到头文件

**错误信息**:
```
fatal error: 'tee_ext_api.h' file not found
```

**原因**: 第三方头文件未正确导入

**解决方案**:
```bash
# 1. 下载第三方源码
git clone git@gitee.com:openharmony/third_party_musl.git
git clone git@gitee.com:openharmony/third_party_bounds_checking_function.git

# 2. 执行导入脚本
./tee_dev_kit/sdk/thirdparty/open_source/import_open_source_header.sh
```

**证据**: `README.md:64-79`

---

### 问题 2: 工具链路径未配置

**错误信息**:
```
clang: error: unable to execute command: No such file or directory
clang: error: clang frontend failed to emit LLVM IR
```

**原因**: LLVM 工具链未添加到 PATH

**解决方案**:
```bash
# 添加工具链路径 (根据实际路径修改)
export PATH=openharmony/prebuilts/clang/ohos/linux-x86_64/15.0.4/llvm/bin:$PATH
```

**证据**: `README.md:56-62`

---

### 问题 3: Python 依赖缺失

**错误信息**:
```
ModuleNotFoundError: No module named 'Crypto'
```

**原因**: pycryptodome 未安装

**解决方案**:
```bash
pip install pycryptodome defusedxml
```

**证据**: `README.md:95-99`

---

### 问题 4: 64位/32位架构配置错误

**错误信息**:
```
unrecognized emulation mode: aarch64
```

**原因**: Makefile 中 TARGET_S_SARM64 配置与实际工具链不匹配

**解决方案**:
```makefile
# 在 Makefile 开头检查
ifeq ($(TARGET_S_SARM64),y)
    $(info "Building 64-bit TA")
    # 64位配置
else
    $(info "Building 32-bit TA")
    # 32位配置
endif
```

**证据**: `README.md:138-139`

---

## 签名问题

### 问题 5: 签名失败 - 密钥不匹配

**错误信息**:
```
ERROR: Signature verification failed
```

**原因**: 签名使用的私钥与 TEE OS 中的公钥不匹配

**解决方案**:
1. 检查 `config_ta_public.ini` 中的密钥路径
2. 确保使用的是正确的私钥
3. 商用环境必须替换默认密钥

**证据**: `README.md:81-86`

---

### 问题 6: 签名失败 - ELF 格式错误

**错误信息**:
```
ERROR: invalid elf header info
```

**原因**: ELF 文件格式不符合要求

**排查步骤**:
```bash
# 检查 ELF 文件
file libcombine.so

# 验证 ELF 头
hexdump -C libcombine.so | head -n 5
```

**证据**: `sdk/build/script/signtool_sec.py:103-124`

---

### 问题 7: 签名失败 - 内存基线检查失败

**错误信息**:
```
WARNING: memory baseline checking failed, but sign will continue temporarily.
```

**原因**: configs.xml 中的内存配置超出基线限制

**解决方案**:
1. 检查 `stack_size` 和 `heap_size` 是否合理
2. 或在配置中启用内存检查跳过（仅限调试）

**证据**: `sdk/build/script/signtool_sec.py:746-747`

---

### 问题 8: 签名失败 - UUID 格式错误

**错误信息**:
```
ERROR: Invalid UUID format in configs.xml
```

**原因**: configs.xml 中的 UUID 格式不正确

**正确格式**:
```xml
<uuid>12345678-1234-1234-1234-123456789abc</uuid>
```

**证据**: `sdk/build/TA_demo/configs.xml`

---

## 运行时问题

### 问题 9: CA 连接 TA 失败

**错误信息**:
```
TEEC_ERROR_COMMUNICATION
```

**原因**: TEE Framework 通信失败

**排查步骤**:
1. 确认 TEE OS 已正确加载
2. 确认 TA 已正确安装
3. 检查 TA UUID 是否匹配

---

### 问题 10: TA 返回错误码

**常见错误码**:

| 错误码 | 说明 | 排查方向 |
|--------|------|----------|
| `TEE_ERROR_BAD_PARAMETERS` | 参数错误 | 检查输入参数有效性 |
| `TEE_ERROR_SHORT_BUFFER` | 缓冲区不足 | 增加输出缓冲区大小 |
| `TEE_ERROR_OUT_OF_MEMORY` | 内存不足 | 检查 heap_size 配置 |
| `TEE_ERROR_NOT_IMPLEMENTED` | 未实现 | 检查命令 ID 是否正确 |
| `TEE_ERROR_SECURITY` | 安全错误 | 检查权限和安全操作 |

**证据**: `sdk/src/TA/helloworld_demo/ta_demo.c`

---

## 配置问题

### 问题 11: configs.xml 解析错误

**错误信息**:
```
XML parse error in configs.xml
```

**原因**: XML 格式不正确

**排查**:
```bash
# 使用 xmllint 验证
xmllint --noout configs.xml
```

---

### 问题 12: 服务名称无效

**错误信息**:
```
Invalid service_name: must be alphanumeric with '_' or '-'
```

**原因**: service_name 包含非法字符

**限制**:
- 长度: 1-64 字符
- 字符: 数字、字母、`_`、`-`

---

## 调试技巧

### 启用详细日志

```c
// 在 TA 代码中添加
#define TEE_LOG_LEVEL_DEBUG
#include <tee_log.h>

tlogd("Debug message: %d", value);
tlogi("Info message");
tloge("Error message: 0x%x", result);
```

**证据**: `sdk/src/TA/helloworld_demo/ta_demo.c`

### CA 白名单配置

```c
// 在 TA_CreateEntryPoint 中添加
const uint8_t hash[] = {
    0xca, 0x9f, 0x5e, 0xd7, ... // CA 哈希
};
AddCaller_CA(hash, sizeof(hash));
```

**证据**: `sdk/src/TA/helloworld_demo/ta_demo.c:58-70`

---

## 继续阅读

### 推荐阅读路径

| 文档 | 阅读时间 | 目标 |
|------|---------|------|
| [02_TA_Development_Guide.md](02_TA_Development_Guide.md) | 30 分钟 | 开发指南 |
| [03_Build_System.md](03_Build_System.md) | 20 分钟 | 构建系统 |
| [04_Security_Review.md](04_Security_Review.md) | 30 分钟 | 安全评审 |
| [05_Attack_Surface.md](05_Attack_Surface.md) | 15 分钟 | 攻击面分析 |
| [05_Examples.md](05_Examples.md) | 20 分钟 | 示例代码 |

---

## 变更历史

| 版本 | 日期 | 变更说明 |
|------|------|---------|
| 1.0.0 | 2026-02-07 | 初始版本 |

---

*最后更新: 2026-02-07*
