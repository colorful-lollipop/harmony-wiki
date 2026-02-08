# 编译产物

## 目的

本文档说明 startup_cangjie_wrapper 的编译产物，包括产物清单、安装路径和运行时加载关系。

## 适用范围

- 需要了解组件产物的开发者
- 准备部署或集成组件的工程师

---

## 产物清单

### 编译目标

| 目标 | 类型 | 输出文件名 | 说明 |
|------|------|-----------|------|
| copy_sdk_startup_cangjie_libs | copy_ohos_cangjie_sdk_api_lib | - | 复制 SDK 库 |
| ohos.device_info | ohos_cangjie_shared_library | TODO | 仓颉共享库 |

**总计**: 2 个编译目标

**证据**: `BUILD.gn:14-22`, `ohos/device_info/BUILD.gn:20-36`

---

## 主要产物：ohos.device_info

### 目标类型

**类型**: `ohos_cangjie_shared_library`

**说明**: 仓颉共享库，可被仓颉应用动态加载

**证据**: `ohos/device_info/BUILD.gn:20`

### 输出文件

**文件名**: TODO（待确认）

**可能的形式**:
- `libohos.device_info.so` - Linux/Android 风格
- `ohos.device_info` - 仓颉风格

**文件类型**: 共享库（.so 或类似）

**大小**: TODO（需要实际编译验证）

### 源文件映射

| 平台 | 源文件 | 说明 |
|------|--------|------|
| Linux/OpenHarmony | `device_info.cj` | 真实实现，包含 FFI 调用 |
| Windows/Mac | `mock/ohos.device_info.cj` | 模拟实现，返回默认值 |

**证据**: `ohos/device_info/BUILD.gn:22-26`

---

## 安装路径

### 系统路径

**TODO**: 需要确认

**可能的位置**:
- `/usr/lib/` - 系统库目录
- `/system/lib/` - OpenHarmony 系统库目录
- `/system/lib64/` - 64 位系统库目录
- SDK 目录 - 作为 SDK 的一部分

### SDK 路径

**组件**: `@ohos/startup_cangjie_wrapper`

**包名**: `ohos.device_info`

**导入方式**:
```cangjie
import ohos.device_info
```

**证据**: `bundle.json:2,14`, `device_info.cj:18`

---

## 运行时加载关系

### 应用加载流程

```
1. 仓颉应用启动
   ↓
2. import ohos.device_info
   ↓
3. 仓颉运行时加载 libohos.device_info.so
   ↓
4. 初始化 FFI 依赖
   ├─ init:cj_device_info_ffi
   └─ cangjie_ark_interop:ohos.labels
   ↓
5. 应用调用 DeviceInfo.xxx
   ↓
6. FFI 调用到 init SA 服务
```

### 依赖库加载顺序

```
libohos.device_info.so
  ├─ 依赖 libcj_device_info_ffi.so (init 组件)
  └─ 依赖 libcangjie_ark_interop.so
```

**证据**:
- `ohos/device_info/BUILD.gn:28-32`
- `bundle.json:24-26`

---

## 内存占用

### 组件资源占用

| 资源类型 | 大小 | 说明 |
|---------|------|------|
| ROM | 150KB | 存储占用 |
| RAM | 116KB | 运行时占用 |

**证据**: `bundle.json:20-21`

### 运行时内存

**TODO**: 需要实际测量

**估算**:
- 代码段: ~50KB
- 数据段: ~10KB
- 堆栈: <10KB（无状态设计）

---

## 产物验证

### 编译输出检查

```bash
# 查看编译产物
ls -lh out/产品名/.../libohos.device_info.so

# 查看符号
readelf -s libohos.device_info.so | grep FfiOHOSDeviceInfo

# 查看依赖
readelf -d libohos.device_info.so | grep NEEDED
```

### 运行时检查

```bash
# 检查共享库是否加载
cat /proc/<pid>/maps | grep device_info

# 检查符号解析
ldd libohos.device_info.so
```

---

## 集成到应用

### 依赖声明

在 `build-profile.json5` 中声明依赖：

```json
{
  "externalModules": [
    {
      "name": "@ohos/startup_cangjie_wrapper",
      "version": "6.1",
      "sources": []
    }
  ]
}
```

**证据**: `bundle.json:2-5`

### 权限声明

在 `module.json5` 中声明权限（如需访问 UDID）：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.sec.ACCESS_UDID",
        "reason": "需要获取设备唯一标识",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

**证据**: `README_zh.md:49`

---

## 产物差异

### 真实实现 vs 模拟实现

| 特性 | 真实实现 (Linux/OpenHarmony) | 模拟实现 (Windows/Mac) |
|------|---------------------------|---------------------|
| 源文件 | device_info.cj | mock/ohos.device_info.cj |
| FFI 调用 | ✓ | ✗ |
| 返回值 | 真实设备信息 | 默认值（String() 或 0） |
| 权限检查 | ✓ | ✗ |
| 适用场景 | 真实设备 | 开发/测试环境 |

**证据**: `ohos/device_info/BUILD.gn:22-26`, `mock/ohos.device_info.cj`

---

## 关键结论

1. **产物简单**: 主要产物为 ohos.device_info 共享库
2. **平台适配**: 通过条件编译切换真实实现和模拟实现
3. **依赖明确**: 依赖 init 组件和 cangjie_ark_interop
3. **资源占用小**: ROM 150KB，RAM 116KB
4. **无状态设计**: 运行时内存占用小
5. **动态加载**: 由仓颉运行时动态加载
6. **安装路径**: TODO（需要确认）

---

## TODO（待确认）

- [ ] 确认 ohos.device_info 的实际输出文件名和扩展名
- [ ] 确认产物的实际安装路径（系统路径和 SDK 路径）
- [ ] 确认 copy_sdk_startup_cangjie_libs 的实际复制目标路径
- [ ] 确认仓颉共享库的运行时加载机制和加载路径
- [ ] 实际测量运行时内存占用

---

## 相关跳转链接

- [00_Overview.md](00_Overview.md) - 项目概览
- [06_GN_Targets.md](06_GN_Targets.md) - GN 目标详解
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构

---

## 参考资料

- [OpenHarmony 应用包结构](https://docs.openharmony.cn/application-dev/quick-start/start-overview)
- [仓颉语言运行时](https://developer.openharmony.cn/cn/doc/cangjie-runtime)
- [OpenHarmony 共享库规范](https://docs.openharmony.cn/application-dev/napi/ffi-napi)

---

*最后更新: 2026-02-06*
