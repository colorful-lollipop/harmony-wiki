# 附录 B: 关键配置项

## 编译配置

### 覆盖率配置

**文件**: `config/BUILD.gn`

```gn
declare_args() {
  neural_network_runtime_coverage = false
}

config("coverage_flags") {
  if (neural_network_runtime_coverage) {
    cflags = [ "--coverage" ]
    cflags_cc = [ "--coverage" ]
    ldflags = [ "--coverage" ]
  }
}
```

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `neural_network_runtime_coverage` | bool | false | 启用代码覆盖率检测 |

## 安全编译选项

### Core 库配置

**文件**: `frameworks/native/neural_network_core/BUILD.gn`

```gn
config("nnrt_config") {
  cflags = [
    "-fstack-protector-all",  # 栈保护
    "-fexceptions",           # C++ 异常支持
  ]
}

ohos_shared_library("libneural_network_core") {
  branch_protector_ret = "pac_ret"  # 分支保护
}
```

### Runtime 库配置

**文件**: `frameworks/native/neural_network_runtime/BUILD.gn`

```gn
config("nnrt_config") {
  include_dirs = [ "//foundation/ai/neural_network_runtime/interfaces/kits/c" ]
  cflags_cc = [ "-fexceptions" ]
}

ohos_shared_library("libneural_network_runtime") {
  branch_protector_ret = "pac_ret"
}
```

## 安装配置

### 安装镜像

| 产物 | 安装镜像 | 路径 |
|------|----------|------|
| libneural_network_core.so | system, updater | /system/lib/ |
| libneural_network_runtime.so | system, updater | /system/lib/ |
| libnnrt_device_service_2.0.so | chipset_base_dir | /vendor/lib/ |
| libnnrt_driver.so | chipset_base_dir | /vendor/lib/ |

## 元数据配置

### 子系统和部件

**文件**: `bundle.json`

```json
{
  "component": {
    "name": "neural_network_runtime",
    "subsystem": "ai",
    "syscap": ["SystemCapability.AI.NeuralNetworkRuntime"],
    "adapted_system_type": ["standard"],
    "rom": "1024KB",
    "ram": "2048KB"
  }
}
```

| 配置项 | 值 | 说明 |
|--------|-----|------|
| subsystem | ai | 所属子系统 |
| part_name | neural_network_runtime | 部件名称 |
| syscap | SystemCapability.AI.NeuralNetworkRuntime | 系统能力 |
| rom | 1024KB | ROM 占用 |
| ram | 2048KB | RAM 占用 |

### API 标签

```gn
metadata = {
  subsystem_name = "ai"
  innerapi_tags = [ "ndk" ]  # NDK 公开 API
  part_name = "neural_network_runtime"
}
```

## 运行时配置

### 自动卸载时间

**文件**: `frameworks/native/neural_network_runtime/nnexecutor.cpp`

```cpp
// 自动卸载延迟时间 (毫秒)
const uint32_t AUTOUNLOAD_TIME = 10 * 60 * 1000;  // 10 分钟
```

### 最大设备数量

**文件**: `frameworks/native/neural_network_core/backend_manager.h`

```cpp
// 后端管理器配置
class BackendManager {
    std::vector<size_t> m_backendIDs;
    std::unordered_map<size_t, std::shared_ptr<Backend>> m_backends;
    // ...
};
```

### 内存管理配置

**文件**: `frameworks/native/neural_network_runtime/memory_manager.h`

```cpp
class MemoryManager {
    // key: buffer pointer, value: Memory struct
    std::unordered_map<const void*, Memory> m_memorys;
    std::mutex m_mtx;
};
```

## 驱动服务配置

### 进程配置

**文件**: `example/drivers/nnrt/v2_0/hdi_cpu_service/device_info.hcs`

```hcs
nnrt :: host {
    hostName = "nnrt_host";
    priority = 50;
    uid = "";
    gid = "";
    caps = ["DAC_OVERRIDE", "DAC_READ_SEARCH"];
}
```

### SELinux 配置

**文件**: `example/drivers/README_zh.md`

```
# 服务上下文
nnrt_device_service  u:object_r:hdf_nnrt_device_service:s0

# 进程类型
type nnrt_host, hdfdomain, domain;

# 权限规则
allow nnrt_host hdf_nnrt_device_service:hdf_devmgr_class { add get };
allow nnrt_host hdf_nnrt_device_service:binder { call };
```

## 相关跳转

- [GN 构建目标](../06_GN_Targets.md)
- [编译产物](../07_Build_Artifacts.md)
- [安全风险评审](../08_Security_Review.md)
