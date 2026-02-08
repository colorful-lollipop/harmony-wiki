# 构建与产物 (Build)

## 1. GN 构建配置

### 1.1 顶层 BUILD.gn

**文件**: `services/BUILD.gn`

**主要目标**:
```gn
# 服务端库
shared_library("midi_service") {
    sources = [ ... ]
    deps = [
        "//foundation/multimedia/midi_framework/services/idl:midi_service_idl",
        "//foundation/multimedia/midi_framework/services/common:midi_common",
    ]
}

# 服务端包
ohos_source_set("midi_service_packages") {
    deps = [
        ":midi_service",
        "//foundation/multimedia/midi_framework/sa_profile:midi_service_sa_profile",
    ]
}
```

### 1.2 框架层 BUILD.gn

**文件**: `frameworks/native/ohmidi/BUILD.gn`

**目标**: `ohmidi` - Native API 库
```gn
ohos_shared_library("ohmidi") {
    sources = [ "OHMidi.cpp" ]
    deps = [
        "//foundation/multimedia/midi_framework/frameworks/native/midi:midi_client",
    ]
}
```

**文件**: `frameworks/native/midi/BUILD.gn`

**目标**: `midi_client` - 客户端核心

**文件**: `frameworks/native/midiutils/BUILD.gn`

**目标**: `midi_utils` - 工具库

---

## 2. 编译命令

### 2.1 编译 32 位 ARM

```bash
./build.sh \
    --product-name {product_name} \
    --ccache \
    --build-target midi_framework
```

### 2.2 编译 64 位 ARM

```bash
./build.sh \
    --product-name {product_name} \
    --ccache \
    --target-cpu arm64 \
    --build-target midi_framework
```

### 2.3 支持的 product_name

- `rk3568` - 瑞芯微 RK3568 开发板
- 其他支持标准系统的平台

---

## 3. 编译产物

### 3.1 共享库

| 产物 | 路径 | 说明 |
|------|------|------|
| `libohmidi.so` | `/system/lib/` 或 `/system/lib64/` | Native API 库 |
| `libmidi_client.so` | `/system/lib/` 或 `/system/lib64/` | 客户端核心 |
| `libmidi_service.z.so` | `/system/lib/` 或 `/system/lib64/` | 服务端库 |

### 3.2 配置文件

| 产物 | 路径 | 说明 |
|------|------|------|
| `midi_server.json` | `/system/profile/` | SA 配置文件 |
| `midi_server.cfg` | `/system/etc/` | 进程启动配置 |

### 3.3 头文件

| 头文件 | 安装路径 | 说明 |
|--------|----------|------|
| `native_midi.h` | SDK 开发包 | 对外 C API |
| `native_midi_base.h` | SDK 开发包 | 基础类型定义 |

---

## 4. bundle.json 构建配置

**文件**: `bundle.json`

```json
{
    "component": {
        "name": "midi_framework",
        "subsystem": "multimedia",
        "adapted_system_type": ["standard", "small", "mini"],
        "rom": "4096KB",
        "ram": "4096KB",
        "build": {
            "group_type": {
                "fwk_group": [
                    "//foundation/multimedia/midi_framework/frameworks/native/ohmidi:ohmidi"
                ],
                "service_group": [
                    "//foundation/multimedia/midi_framework/sa_profile:midi_service_sa_profile",
                    "//foundation/multimedia/midi_framework/services:midi_service_packages"
                ]
            },
            "test": [
                "//foundation/multimedia/midi_framework/test:midi_unit_test",
                "//foundation/multimedia/midi_framework/test:midi_demo_test"
            ]
        }
    }
}
```

---

## 5. Feature 开关

当前版本未发现条件编译 Feature 开关。所有功能通过运行时检测启用。

### 5.1 运行时配置

**卸载延迟配置**:
```cpp
// services/server/src/midi_service_controller.cpp:35
constexpr uint32_t UNLOAD_DELAY_DEFAULT_TIME_IN_MS = 60 * 1000;  // 60秒
```

**测试模式**:
```cpp
// 通过宏 UNIT_TEST_SUPPORT 启用测试辅助方法
#ifdef UNIT_TEST_SUPPORT
void ClearStateForTest();
#endif
```

---

## 6. 依赖检查

编译前确保以下组件已加入编译:

```json
// vendor/hihope/rk3568/config.json
{
    "components": [
        "midi_framework",
        "drivers_peripheral_midi",
        "drivers_interface_midi"
    ]
}
```

---

## 7. 调试构建

### 7.1 启用详细日志

修改日志级别:
```cpp
// interfaces/midi_log.h
#define MIDI_DEBUG_LOG(fmt, ...) // 取消注释启用 debug 日志
```

### 7.2 单元测试编译

```bash
./build.sh \
    --product-name rk3568 \
    --build-target midi_unit_test
```

---

*文档结束*
