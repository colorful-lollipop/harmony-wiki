# 构建系统

## GN 目标

### 主构建目标

| 目标 | 路径 | 类型 | 输出 |
|------|------|------|------|
| `wifi` | `//foundation/communication/wifi_lite:wwi` | group | 头文件导出 |

**证据**: [BUILD.gn:18-20](BUILD.gn#L18-L20)

### 配置目标

| 目标 | 路径 | 导出内容 |
|------|------|----------|
| `include` | `//foundation/communication/wifi_lite:include` | `interfaces/wifiservice` 目录 |

**证据**: [BUILD.gn:14-16](BUILD.gn#L14-L16)

## BUILD.gn 完整内容

```gn
# Copyright (c) 2020 Huawei Device Co., Ltd.
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

config("include") {
  include_dirs = [ "interfaces/wifiservice" ]
}

group("wifi") {
  public_configs = [ ":include" ]
}
```

**证据**: [BUILD.gn 完整文件](BUILD.gn)

## 组件描述（bundle.json）

### 组件信息

| 字段 | 值 |
|------|------|
| 名称 | `@openharmony/wifi_lite` |
| 版本 | 3.1.0 |
| 描述 | wifi_lite |
| 发布类型 | code-segment |

**证据**: [bundle.json:2-5](bundle.json#L2-L5)

### 子系统与组件

```json
{
  "subsystem": "communication",
  "component": "wifi_lite",
  "adapted_system_type": ["mini"]
}
```

**证据**: [bundle.json:17-20](bundle.json#L17-L20)

### 构建配置

```json
{
  "build": {
    "sub_component": [
      "//foundation/communication/wifi_lite:wifi"
    ]
  }
}
```

**证据**: [bundle.json:32-35](bundle.json#L32-L35)

## 编译产物

### 产物清单

| 类型 | 说明 | 路径 |
|------|------|------|
| 头文件 | C 接口定义 | `interfaces/wifiservice/*.h` |

### 特性说明

- **无源码编译**: 本模块仅包含头文件，无 `.c/.cpp` 源文件
- **纯接口库**: BUILD.gn 中无 `sources` 列表，仅导出 `include_dirs`
- **静态链接**: 使用方通过 `public_configs` 依赖本模块的头文件

### 使用方式

在目标模块的 `BUILD.gn` 中添加依赖：

```gn
ohos_shared_library("my_wifi_app") {
  deps += [
    "//foundation/communication/wifi_lite:wifi"
  ]
}
```

或在 `include_dirs` 中手动添加：

```gn
config("my_config") {
  include_dirs += [ "//foundation/communication/wifi_lite/interfaces/wifiservice" ]
}
```

## 依赖关系

### 入向依赖（使用本模块的模块）

| 模块 | 依赖方式 |
|------|----------|
| Wi-Fi 服务实现 | `public_configs` |
| 上层应用 | `deps` 或 `include_dirs` |

### 出向依赖（本模块依赖的模块）

| 模块 | 依赖内容 |
|------|----------|
| 无 | 本模块为纯接口层，无外部依赖 |

**证据**: bundle.json 中的 `deps` 字段为空

## 编译命令

### 标准编译

```bash
hb set
hb build wifi_lite
```

### 仅验证配置

```bash
gn gen out/{device_name}
ninja -C out/{device_name} //foundation/communication/wifi_lite:wifi
```

## 安装路径

编译产物（头文件）安装到：

```
${OHOS_SDK}/native/sysroot/include/openharmony/
```

## 注意事项

1. **接口稳定性**: 本模块提供的 C 接口为稳定接口
2. **版本兼容性**: 3.1.0 版本接口定义完整
3. **构建产物**: 无 `.so`/`.a` 产物，仅头文件索引

---

[返回 SUMMARY.md](SUMMARY.md) | [概览](01_Overview.md) | [API 参考](02_API_Reference.md) | [安全评审](04_Security_Review.md)
