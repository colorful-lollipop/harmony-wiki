# 03 - OpenHarmony 构建适配

本文档详细说明 decimal.js 在 OpenHarmony 中的构建适配，包括 BUILD.gn 配置和与上游构建系统的差异。

---

## 3.1 BUILD.gn 完整分析

### 文件位置

`third_party/decimal.js/BUILD.gn`

### 完整内容

```gn
# Copyright (c) 2024 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/config/components/ets_frontend/es2abc_config.gni")
import("//build/ohos.gni")

es2abc_gen_abc("gen_decimal_abc") {
  src_js = rebase_path("decimal.mjs")
  dst_file = rebase_path(target_out_dir + "/decimal.abc")
  in_puts = [ "decimal.mjs" ]
  out_puts = [ target_out_dir + "/decimal.abc" ]
  extra_args = [ "--module" ]
}

gen_js_obj("decimal_abc") {
  input = get_label_info(":gen_decimal_abc", "target_out_dir") + "/decimal.abc"
  output = target_out_dir + "/decimal_abc.o"
  dep = ":gen_decimal_abc"
}

gen_js_obj("decimal_mjs") {
  input = rebase_path("decimal.mjs")
  output = target_out_dir + "/decimal_mjs.o"
}

ohos_shared_library("decimal") {
  sources = [ "decimal.cpp" ]

  deps = [
    ":decimal_abc",
    ":decimal_mjs",
  ]

  external_deps = [ "napi:ace_napi" ]

  license_file = "./LICENCE.md"
  relative_install_dir = "module/arkts/math"
  part_name = "decimal.js"
  subsystem_name = "thirdparty"
}
```

---

## 3.2 构建流程详解

### 整体流程图

```
┌─────────────────────────────────────────────────────────────────┐
│                        构建流程                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐    │
│  │ decimal.mjs  │────▶│  es2abc      │────▶│ decimal.abc  │    │
│  │ (源码)        │     │ (编译器)      │     │ (字节码)      │    │
│  └──────────────┘     └──────────────┘     └──────────────┘    │
│         │                                           │           │
│         │                    ┌──────────────────────┘           │
│         │                    ▼                                  │
│         │            ┌──────────────┐                          │
│         │            │ gen_js_obj   │                          │
│         │            │ (二进制嵌入)  │                          │
│         │            └──────────────┘                          │
│         │                    │                                  │
│         └────────────────────┤                                  │
│                              ▼                                  │
│                    ┌──────────────────┐                        │
│                    │ decimal_abc.o    │                        │
│                    │ decimal_mjs.o    │                        │
│                    └────────┬─────────┘                        │
│                             │                                   │
│                             ▼                                   │
│                    ┌──────────────────┐                        │
│                    │ decimal.cpp      │                        │
│                    │ (NAPI 适配层)     │                        │
│                    └────────┬─────────┘                        │
│                             │                                   │
│                             ▼                                   │
│                    ┌──────────────────┐                        │
│                    │ libdecimal.z.so  │                        │
│                    │ (最终输出)        │                        │
│                    └──────────────────┘                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 阶段 1: JavaScript → ArkTS 字节码

**目标**: 将 ES Module 转换为 ArkTS 字节码（.abc 格式）

**配置**:
```gn
es2abc_gen_abc("gen_decimal_abc") {
  src_js = rebase_path("decimal.mjs")        # 输入: ES Module 源码
  dst_file = rebase_path(target_out_dir + "/decimal.abc")  # 输出: ArkTS 字节码
  in_puts = [ "decimal.mjs" ]
  out_puts = [ target_out_dir + "/decimal.abc" ]
  extra_args = [ "--module" ]                # 按模块模式编译
}
```

**关键参数**:
- `--module`: 表示输入是 ES Module 格式
- `es2abc`: ArkCompiler 的前端编译器，将 JS/TS 转为 ABC 字节码

### 阶段 2: 字节码嵌入为二进制对象

**目标**: 将字节码文件打包为可链接的对象文件

**配置**:
```gn
gen_js_obj("decimal_abc") {
  input = get_label_info(":gen_decimal_abc", "target_out_dir") + "/decimal.abc"
  output = target_out_dir + "/decimal_abc.o"
  dep = ":gen_decimal_abc"
}

gen_js_obj("decimal_mjs") {
  input = rebase_path("decimal.mjs")
  output = target_out_dir + "/decimal_mjs.o"
}
```

**说明**:
- `gen_js_obj`: 将文件内容转换为 C++ 可链接的二进制对象
- 生成两个对象文件：字节码和原始 MJS 源码
- 这些对象文件会被链接到最终的共享库中

### 阶段 3: 构建共享库

**目标**: 编译 C++ 适配层并链接所有对象

**配置**:
```gn
ohos_shared_library("decimal") {
  sources = [ "decimal.cpp" ]                # C++ NAPI 适配层
  deps = [
    ":decimal_abc",                          # 依赖: 字节码对象
    ":decimal_mjs",                          # 依赖: 源码对象
  ]
  external_deps = [ "napi:ace_napi" ]        # 依赖: NAPI 框架
  license_file = "./LICENCE.md"
  relative_install_dir = "module/arkts/math" # 安装路径
  part_name = "decimal.js"
  subsystem_name = "thirdparty"
}
```

---

## 3.3 关键编译选项

### defines（宏定义）

无特殊 `defines` 配置，使用默认值。

### configs（编译配置）

无特殊 `configs` 配置，继承默认配置。

### cflags/cflags_cc（编译器标志）

无额外编译器标志，使用系统默认。

### 依赖配置

| 依赖类型 | 目标 | 说明 |
|----------|------|------|
| `deps` | `:decimal_abc` | 字节码对象文件 |
| `deps` | `:decimal_mjs` | 源码对象文件 |
| `external_deps` | `napi:ace_napi` | NAPI 运行时框架 |

---

## 3.4 输出产物

### 输出文件

| 文件 | 路径 | 说明 |
|------|------|------|
| `libdecimal.z.so` | `out/<product>/thirdparty/decimal.js/` | 最终共享库 |

### 安装路径

```
system/
└── lib/
    └── module/
        └── arkts/
            └── math/
                └── libdecimal.z.so
```

### 库的功能

`libdecimal.z.so` 包含：

1. **ArkTS 字节码** (`decimal.abc`)
   - 可直接被 ArkTS 运行时加载执行

2. **原始 MJS 源码** (`decimal.mjs`)
   - 用于调试或源码映射

3. **NAPI 适配层** (`decimal.cpp`)
   - 模块注册：`arkts.math.Decimal`
   - 字节码导出函数

---

## 3.5 与上游构建系统对比

### 上游构建方式

| 方面 | 上游 (npm) |
|------|-----------|
| 构建工具 | npm / Node.js |
| 输入 | `decimal.mjs` / `decimal.js` |
| 输出 | JS/MJS 文件（无需编译） |
| 使用方式 | `npm install` 后 `require()` 或 `import` |
| 安装路径 | `node_modules/decimal.js/` |

### OpenHarmony 构建方式

| 方面 | OH (GN) |
|------|---------|
| 构建工具 | GN + es2abc + Clang |
| 输入 | `decimal.mjs` |
| 处理 | es2abc 编译为字节码 → gen_js_obj 嵌入 → C++ 链接 |
| 输出 | `libdecimal.z.so` |
| 使用方式 | `import { Decimal } from '@kit.ArkTS'` |
| 安装路径 | `system/lib/module/arkts/math/` |

### 关键差异

```
┌────────────────────────────────────────────────────────────────┐
│                        差异对比                                 │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  上游 (Node.js)                    OH (ArkTS)                  │
│  ───────────────                   ─────────                   │
│                                                                 │
│  ┌─────────────┐                  ┌─────────────┐              │
│  │ decimal.mjs │                  │ decimal.mjs │              │
│  └──────┬──────┘                  └──────┬──────┘              │
│         │                                │                      │
│         │ 直接使用                        │ es2abc 编译         │
│         │                                ▼                      │
│         │                         ┌─────────────┐              │
│         │                         │ decimal.abc │              │
│         │                         └──────┬──────┘              │
│         │                                │                      │
│         │                                │ gen_js_obj 嵌入      │
│         │                                ▼                      │
│         │                         ┌─────────────┐              │
│         │                         │ decimal.cpp │ (NAPI)       │
│         │                         └──────┬──────┘              │
│         │                                │                      │
│         │                                │ 链接                 │
│         │                                ▼                      │
│         │                         ┌─────────────┐              │
│         │                         │libdecimal.so│              │
│         │                         └──────┬──────┘              │
│         │                                │                      │
│         ▼                                ▼                      │
│  ┌─────────────┐                  ┌─────────────┐              │
│  │ require()   │                  │ import from │              │
│  │ import      │                  │ @kit.ArkTS  │              │
│  └─────────────┘                  └─────────────┘              │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

---

## 3.6 升级指南

### 升级场景

需要升级到新版本时（如从 v10.5.0 升级到 v10.6.0）：

### 升级步骤

#### 步骤 1: 准备新版本源码

```bash
# 进入库目录
cd third_party/decimal.js/

# 备份当前版本
cp decimal.mjs decimal.mjs.v10.5.0
cp decimal.js decimal.js.v10.5.0

# 下载新版本（以 v10.6.0 为例）
curl -o decimal.mjs https://raw.githubusercontent.com/MikeMcl/decimal.js/v10.6.0/decimal.mjs
curl -o decimal.js https://raw.githubusercontent.com/MikeMcl/decimal.js/v10.6.0/decimal.js
curl -o decimal.d.ts https://raw.githubusercontent.com/MikeMcl/decimal.js/v10.6.0/decimal.d.ts
```

#### 步骤 2: 重新应用 OH 特有修改

```bash
# 编辑 decimal.mjs，在文件开头（第 7 行后）添加以下内容：

class BusinessError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'BusinessError';
    this.code = code;
  }
}
const RANGE_ERROR_CODE = 10200001;
const TYPE_ERROR_CODE = 401;
const PRECISION_LIMIT_EXCEEDED_ERROR_CODE = 10200060;
const CRYPTO_UNAVAILABLE_ERROR_CODE = 10200061;
```

#### 步骤 3: 更新版本号

```bash
# 更新 bundle.json 中的版本号
# 将 "version": "10.4.3" 改为 "version": "10.6.0"
```

#### 步骤 4: 编译验证

```bash
# 执行构建
./build.sh --product-name rk3568 --target third_party/decimal.js

# 检查输出
ls -la out/rk3568/thirdparty/decimal.js/libdecimal.z.so
```

#### 步骤 5: 功能测试

```bash
# 1. 验证字节码生成
file out/rk3568/thirdparty/decimal.js/decimal.abc

# 2. 验证共享库
nm -D out/rk3568/thirdparty/decimal.js/libdecimal.z.so | grep NAPI

# 3. 运行时测试（在设备上）
# 创建测试 ArkTS 应用，验证 Decimal 功能正常
```

### 升级检查清单

| 检查项 | 检查方法 | 期望结果 |
|--------|----------|----------|
| 源码替换 | 对比文件头版本号 | 显示新版本号 |
| BusinessError 添加 | 检查 decimal.mjs 前 20 行 | 存在错误类定义 |
| es2abc 编译 | 查看构建日志 | 无编译错误 |
| 对象文件生成 | ls *.o | 存在 decimal_abc.o 和 decimal_mjs.o |
| 共享库链接 | ls *.so | 存在 libdecimal.z.so |
| 符号导出 | nm -D libdecimal.z.so | 存在 NAPI_arkts_math_Decimal_* 符号 |
| 运行时加载 | 设备测试 | Decimal 模块可正常加载 |
| API 功能 | 运行测试用例 | 计算结果正确 |

---

## 3.7 常见问题

### Q1: 为什么需要 es2abc 编译？

**A**: ArkTS 应用运行在 ArkCompiler 运行时上，需要 ABC（Ark Bytecode）格式，而非原生 JavaScript。

### Q2: 为什么同时嵌入字节码和源码？

**A**: 
- 字节码（.abc）: 运行时执行用
- 源码（.mjs）: 调试和源码映射用

### Q3: 如何调试构建问题？

**A**:
1. 检查 es2abc 编译日志：`--verbose` 选项
2. 检查对象文件：`objdump -t decimal_abc.o`
3. 检查共享库符号：`nm -D libdecimal.z.so`

### Q4: 可以跳过字节码编译直接使用源码吗？

**A**: 不可以。ArkTS 运行时只能加载 ABC 字节码，不能执行原始 JavaScript。

---

## 3.8 相关文件

| 文件 | 说明 |
|------|------|
| `BUILD.gn` | 主构建配置 |
| `decimal.cpp` | NAPI 适配层源码 |
| `decimal.mjs` | 主库源码（ES Module） |
| `bundle.json` | OH 组件配置 |

---

*下一章: [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 依赖关系与使用*
