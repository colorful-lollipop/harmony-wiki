# Wi-Fi Aware 构建与编译

## 构建配置

### 根构建文件

**文件**: `BUILD.gn`

```gn
static_library("wifiaware") {
  sources = [ "frameworks/source/wifiaware.c" ]
  
  include_dirs = [
    "interfaces/kits",
    "hals",
    "//foundation/communication/wifi_lite/interfaces/wifiservice",
    "$ohos_board_adapter_dir/hals/communication/wifi_lite/wifiservice/source",
  ]
  
  deps = [ "$ohos_board_adapter_dir/hals/communication/wifi_lite/wifiaware:hal_wifiaware" ]
}
```

**证据**: `BUILD.gn:14-23`

## GN Target 分析

### Target: wifiaware

| 属性 | 值 | 说明 |
|-----|---|-----|
| **类型** | `static_library` | 静态库 |
| **源文件** | `frameworks/source/wifiaware.c` | 仅 1 个源文件 |
| **输出** | `libwifiaware.a` | 静态库产物 |

### include_dirs

| 目录 | 来源 | 说明 |
|-----|------|-----|
| `interfaces/kits` | 本模块 | 对外 API 头文件 |
| `hals` | 本模块 | HAL 接口头文件 |
| `//foundation/communication/wifi_lite/interfaces/wifiservice` | wifi_lite | WiFi 服务接口 |
| `$ohos_board_adapter_dir/hals/.../wifiservice/source` | 板级适配 | WiFi 服务 HAL 实现 |

### deps

| 依赖 | 说明 |
|-----|------|
| `$ohos_board_adapter_dir/hals/communication/wifi_lite/wifiaware:hal_wifiaware` | HAL 实现（构建时注入） |

## 依赖关系图

```
wifiaware (static_library)
    │
    ├── interfaces/kits/wifiaware.h
    │       └── (被 wifiaware.c 包含)
    │
    ├── hals/hal_wifiaware.h
    │       └── (被 wifiaware.c 包含)
    │
    ├── //foundation/communication/wifi_lite/interfaces/wifiservice/
    │       └── wifi_device_util.h, wifi_hotspot.h
    │
    └── $ohos_board_adapter_dir/hals/.../wifiservice/source/
            └── GetHotspotInterfaceName, IsHotspotActive, GetHotspotChannel
                    │
                    └── deps → hal_wifiaware
                            └── 板级 HAL 实现
```

## 模块配置

### bundle.json

**文件**: `bundle.json`

```json
{
  "name": "@ohos/wifi_aware",
  "version": "3.1.0",
  "component": {
    "name": "wifi_aware",
    "subsystem": "communication",
    "adapted_system_type": ["small", "standard"],
    "rom": "967KB",
    "ram": "28MB"
  }
}
```

**证据**: `bundle.json:1-42`

### 子组件

| 子组件 | 路径 |
|-------|------|
| `//foundation/communication/wifi_aware:wifiaware` | BUILD.gn 中定义 |

## 编译产物

### 静态库

| 产物 | 路径（预计） | 说明 |
|-----|-------------|-----|
| `libwifiaware.a` | `out/{product}/libs/` | Wi-Fi Aware 静态库 |

### 产物大小

| 类型 | 大小 | 来源 |
|-----|------|-----|
| ROM | 967 KB | bundle.json |
| RAM | 28 MB | bundle.json |

> **注意**: ROM/RAM 数据来自 bundle.json 配置，实际大小可能因优化选项而异。

## 构建时注入机制

### ohos_board_adapter_dir

构建系统通过 `ohos_board_adapter_dir` 变量注入板级 HAL 实现：

```gn
deps = [ "$ohos_board_adapter_dir/hals/communication/wifi_lite/wifiaware:hal_wifiaware" ]
```

此机制允许：
- 不同芯片提供不同 HAL 实现
- 无需修改框架代码
- 构建时动态绑定

## 支持的芯片

| 芯片 | 状态 | 实现位置 |
|-----|------|---------|
| Hi3861 | ✅ 支持 | `$ohos_board_adapter_dir/hals/communication/wifi_lite/wifiaware/` |
| 其他 | ❓ 待定 | 需在 device 目录实现 |

> **限制**: 当前仅支持 Hi3861 开发板。

## 构建命令示例

```bash
# 完整系统构建
./build.sh --product {product_name}

# 仅构建 wifi_aware
hb build -f //foundation/communication/wifi_aware

# 查看产物
ls out/{product}/libs/
```

## 后续阅读

- [架构文档](architecture.md) - 模块层次
- [API 文档](api.md) - C API
- [HAL 接口](hal.md) - HAL 定义
