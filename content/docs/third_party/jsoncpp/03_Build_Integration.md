# OH 构建适配

## 概述

jsoncpp 在 OpenHarmony 中的构建适配相对简单，主要涉及 GN 构建系统的集成。由于 jsoncpp 是纯 C++ 跨平台库，OH 适配主要集中在**构建流程**和**编译选项**的调整，而非源码修改。

## BUILD.gn 结构说明

### 完整 BUILD.gn 文件

```gn
# Copyright (c) 2021 Huawei Device Co., Ltd.
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

import("//build/ohos.gni")

# 1. 源文件处理 Action
# 从 tar.gz 压缩包解压源文件
action("jsoncpp_install_action") {
  script = "//third_party/jsoncpp/install.py"
  outputs = [
    "${target_gen_dir}/jsoncpp-1.9.6/src/lib_json/json_reader.cpp",
    "${target_gen_dir}/jsoncpp-1.9.6/src/lib_json/json_value.cpp",
    "${target_gen_dir}/jsoncpp-1.9.6/src/lib_json/json_writer.cpp",
  ]

  inputs = [ "//third_party/jsoncpp/jsoncpp-1.9.6.tar.gz" ]

  args = [
    "--gen-dir",
    rebase_path("${target_gen_dir}", root_build_dir),
    "--source-file",
    rebase_path("//third_party/jsoncpp"),
  ]
}

# 2. 静态库配置
config("config_static") {
  cflags = [
    "-std=c++17",
    "-Wno-error=implicit-fallthrough",
    "-Wno-deprecated-declarations",
  ]
  visibility = [ ":*" ]
}

# 3. 共享库配置
config("jsoncpp_config") {
  cflags = [
    "-std=c++17",
    "-Wno-error=implicit-fallthrough",
    "-Wno-deprecated-declarations",
  ]
}

# 4. 异常处理配置
config("flag_config") {
  cflags_cc = [ "-fexceptions" ]
}

# 5. 公共配置（头文件路径）
config("jsoncpp_public_config") {
  include_dirs = [ get_label_info(":jsoncpp_install_action", "target_gen_dir") +
                   "/jsoncpp-1.9.6/include" ]
}

# 6. 共享库目标
ohos_shared_library("jsoncpp") {
  branch_protector_ret = "pac_ret"
  visibility = [ "*" ]
  sources = get_target_outputs(":jsoncpp_install_action")
  use_exceptions = true
  configs = [ ":jsoncpp_config" ]
  public_configs = [ ":jsoncpp_public_config" ]
  innerapi_tags = [
    "chipsetsdk_sp",
    "platformsdk",
  ]
  install_images = [
    "system",
    "updater",
  ]
  deps = [ ":jsoncpp_install_action" ]
  part_name = "jsoncpp"
  subsystem_name = "thirdparty"
}

# 7. 静态库目标
ohos_static_library("jsoncpp_static") {
  branch_protector_ret = "pac_ret"
  sources = get_target_outputs(":jsoncpp_install_action")
  use_exceptions = true
  configs = [
    ":config_static",
    ":flag_config",
  ]
  public_configs = [ ":jsoncpp_public_config" ]
  cflags_cc = [
    "-Wall",
    "-Werror",
    "-Wno-implicit-fallthrough",
  ]
  deps = [ ":jsoncpp_install_action" ]
  part_name = "jsoncpp"
  subsystem_name = "thirdparty"
}
```

## 关键编译选项

### C++ 标准

```gn
cflags = [
  "-std=c++17",    # 使用 C++17 标准
]
```

### 警告处理

```gn
cflags = [
  "-Wno-error=implicit-fallthrough",     # 警告降级为非错误
  "-Wno-deprecated-declarations",        # 忽略废弃声明警告
]
```

**说明**：
- `-Wno-error=implicit-fallthrough`：避免 switch-case fallthrough 警告导致的编译失败
- `-Wno-deprecated-declarations`：允许使用废弃的 API，避免警告干扰

### 异常支持

```gn
use_exceptions = true      # 启用 C++ 异常支持
cflags_cc = [ "-fexceptions" ]  # 生成异常支持代码
```

### 安全特性

```gn
branch_protector_ret = "pac_ret"  # PAC/BTI 指针认证保护
```

## 关键配置详解

### 1. 源文件处理（jsoncpp_install_action）

**设计原因**：使用预编译的 tar.gz 而非直接包含源码

| 方式 | 优点 | 缺点 |
|------|------|------|
| tar.gz + action | 减小仓库体积，统一管理 | 构建时需要解压 |
| 直接源码 | 构建速度快 | 仓库体积大 |

**输出文件**：
```
${target_gen_dir}/jsoncpp-1.9.6/
├── include/json/
│   ├── json.h
│   ├── value.h
│   └── ...
└── src/lib_json/
    ├── json_reader.cpp
    ├── json_value.cpp
    └── json_writer.cpp
```

### 2. 头文件导出（jsoncpp_public_config）

```gn
config("jsoncpp_public_config") {
  include_dirs = [
    get_label_info(":jsoncpp_install_action", "target_gen_dir") +
    "/jsoncpp-1.9.6/include"
  ]
}
```

**导出头文件列表**：

| 头文件 | 用途 |
|--------|------|
| `json/json.h` | 主入口，包含所有公共 API |
| `json/value.h` | Json::Value 类型定义 |
| `json/reader.h` | Json::Reader 解析器 |
| `json/writer.h` | Json::Writer 写入器 |
| `json/json_features.h` | 特性开关配置 |
| `json/config.h` | 库配置选项 |
| `json/allocator.h` | 内存分配器接口 |
| `json/assertions.h` | 断言宏定义 |
| `json/forwards.h` | 前向声明 |
| `json/version.h` | 版本信息 |

### 3. 库类型对比

#### 共享库（jsoncpp）

```gn
ohos_shared_library("jsoncpp") {
  # 特征：动态链接，多模块共享
  configs = [ ":jsoncpp_config" ]  # 使用共享库配置
  install_images = ["system", "updater"]
}
```

**优点**：
- 减少内存占用（代码段共享）
- 便于升级（替换单个 so 文件）
- 二进制体积更小

**缺点**：
- 符号冲突风险
- 启动时加载开销

#### 静态库（jsoncpp_static）

```gn
ohos_static_library("jsoncpp_static") {
  # 特征：编译时链接，代码嵌入
  configs = [
    ":config_static",
    ":flag_config",
  ]
  cflags_cc = [
    "-Wall",
    "-Werror",      # 静态库使用更严格的警告
  ]
}
```

**优点**：
- 无符号冲突
- 无运行时依赖
- 更好的优化机会

**缺点**：
- 二进制体积增大
- 代码重复（多份拷贝）

### 4. 依赖关系

```gn
deps = [ ":jsoncpp_install_action" ]
```

**依赖图**：

```
jsoncpp/jsoncpp_static
    │
    └──► jsoncpp_install_action
            │
            └──► jsoncpp-1.9.6.tar.gz
```

## 与上游构建系统的差异

### 上游构建（CMake）

```cmake
# 上游 CMakeLists.txt 摘要
cmake_minimum_required(VERSION 3.15)
project(jsoncpp)

add_library(jsoncpp STATIC
    src/lib_json/json_value.cpp
    src/lib_json/json_reader.cpp
    src/lib_json/json_writer.cpp
)

target_include_directories(jsoncpp PUBLIC
    ${JSONCPP_INCLUDE_DIR}
)
```

### OH 构建（GN）

| 方面 | 上游（CMake） | OH（GN） |
|------|---------------|----------|
| 构建系统 | CMake | GN |
| 源码获取 | 源码目录 | tar.gz + action |
| 标准版本 | 可配置 | C++17 固定 |
| 异常处理 | 可配置 | 启用 |
| 输出类型 | STATIC/SHARED | 两者都有 |

### 关键差异说明

1. **源码管理**
   - 上游：直接包含源码目录
   - OH：使用 tar.gz 压缩包

2. **构建流程**
   - 上游：直接编译
   - OH：解压 → 编译

3. **配置方式**
   - 上游：CMake 参数
   - OH：GN config

## 特殊处理

### 1. install.py 脚本

```python
def main():
    # 解析参数
    parser = argparse.ArgumentParser()
    parser.add_argument('--gen-dir', help='generate path of jsoncpp')
    parser.add_argument('--source-file', help='jsoncpp source compressed dir')
    args = parser.parse_args()
    
    # 流程：解压 → Patch → 复制 → 清理
    untar_file(tar_file_path, tmp_dir, THIS_FILE_PATH)
    do_patch(args, tmp_dir)           # Patch 框架已预留
    do_copy(tmp_dir, args.gen_dir)
    do_remove(tmp_dir)
```

**设计说明**：
- 支持 Patch 框架（当前未使用）
- 临时目录自动清理
- 错误处理完善

### 2. 头文件路径计算

```gn
include_dirs = [
  get_label_info(":jsoncpp_install_action", "target_gen_dir") +
  "/jsoncpp-1.9.6/include"
]
```

**动态路径**：使用 `get_label_info()` 获取生成目录，确保路径正确。

## 常见构建问题

### 问题 1：找不到头文件

**症状**：
```
fatal error: 'json/json.h' file not found
```

**解决方案**：
```gn
# 确保依赖 public_configs
deps = ["//third_party/jsoncpp:jsoncpp"]
public_configs = ["//third_party/jsoncpp:jsoncpp_public_config"]
```

### 问题 2：符号未定义

**症状**：
```
undefined reference to 'Json::Value::operator[](char const&)'
```

**解决方案**：
```gn
# 确保链接正确的库
deps = ["//third_party/jsoncpp:jsoncpp_static"]  # 或 jsoncpp
```

### 问题 3：异常支持缺失

**症状**：
```
exception handling disabled, but exception was thrown
```

**解决方案**：
```gn
# 确保启用异常
use_exceptions = true
cflags_cc = [ "-fexceptions" ]
```

## 性能考虑

### 1. 编译时间

jsoncpp 编译特点：

| 方面 | 情况 |
|------|------|
| 源码规模 | 小（3 个 cpp 文件） |
| 编译依赖 | 无外部依赖 |
| 增量编译 | 快速 |

### 2. 运行时性能

jsoncpp 性能特点：

| 指标 | 描述 |
|------|------|
| 解析速度 | 中等（DOM 方式） |
| 内存占用 | 中等 |
| 特性 | 支持注释保留 |

## 版本升级流程

### 升级步骤

1. **准备 tar.gz**
   ```
   # 从上游下载
   wget https://github.com/open-source-parsers/jsoncpp/archive/refs/tags/1.9.7.tar.gz
   mv 1.9.7.tar.gz jsoncpp-1.9.7.tar.gz
   ```

2. **更新 BUILD.gn**
   ```gn
   outputs = [
     "${target_gen_dir}/jsoncpp-1.9.7/src/lib_json/json_reader.cpp",
     # ... 其他文件
   ]
   
   inputs = ["//third_party/jsoncpp/jsoncpp-1.9.7.tar.gz"]
   
   include_dirs = [... + "/jsoncpp-1.9.7/include"]
   ```

3. **验证构建**
   ```bash
   hb build -p jsoncpp
   ```

4. **测试验证**
   ```bash
   # 运行依赖模块测试
   ```

### 版本兼容性

| 上游版本 | OH 版本 | 兼容性 | 备注 |
|----------|---------|--------|------|
| 1.9.6 | 3.1 | ✅ 兼容 | 当前版本 |
| 1.9.5 | 3.0 | ✅ 兼容 | 历史版本 |
| 1.9.7 | 3.2 | ✅ 应兼容 | 待验证 |

## 总结

jsoncpp 的 OH 构建适配具有以下特点：

1. **最小改动**：仅构建系统适配，无源码 Patch
2. **双库支持**：同时提供共享库和静态库
3. **标准配置**：C++17 + 异常支持
4. **安全增强**：PAC/BTI 指针保护
5. **易于维护**：升级流程简单
