# 编译产物与运行时

## 目的

本文档描述 `communication_cangjie_wrapper` 的编译产物、安装路径和运行时加载关系。

## 适用范围

- 构建工程师
- 系统集成人员
- 部署运维人员

## 编译产物清单

### 主要产物

| 产物名称 | 类型 | 来源目标 | 说明 |
|----------|------|----------|------|
| `libohos.rpc.so` | 共享库 | ohos/rpc:ohos.rpc | 核心 IPC 功能库 |
| `libkit.IPCKit.so` | 共享库 | kit/IPCKit:kit.IPCKit | Kit 层接口库 |
| SDK 头文件/元数据 | 文件集 | copy_sdk_communication_cangjie_libs | 开发 SDK |

### 产物详情

#### libohos.rpc.so

**来源**: `//foundation/communication/communication_cangjie_wrapper/ohos/rpc:ohos.rpc`

**包含内容**:
- MessageSequence 实现
- Ashmem 实现
- Parcelable 接口
- RemoteObject/Proxy 框架
- FFI 桥接层

**大小估计**: ~200KB（基于 ROM 300KB 减去 Kit 层）

**依赖**:
- `libipc_cj_ffi.so`（外部组件）
- `libohos.ffi.so`（来自 cangjie_ark_interop）
- `libohos.hilog.so`（来自 hiviewdfx_cangjie_wrapper）

#### libkit.IPCKit.so

**来源**: `//foundation/communication/communication_cangjie_wrapper/kit/IPCKit:kit.IPCKit`

**包含内容**:
- 统一导出 `ohos.rpc.*`

**大小估计**: ~100KB

**依赖**:
- `libohos.rpc.so`

---

## 安装路径

### 设备端路径

```
/system/
├── lib/
│   ├── libohos.rpc.so              # 核心库
│   └── libkit.IPCKit.so            # Kit 层库（可选）
│
├── lib64/                          # 64位系统
│   ├── libohos.rpc.so
│   └── libkit.IPCKit.so
│
└── etc/                            # 配置文件
    └── ...

/vendor/
└── lib/                            # 厂商定制库（如需要）
    └── libohos.rpc.so
```

### SDK 路径

```
sdk/
├── cangjie/
│   └── libs/
│       ├── libohos.rpc.so
│       └── libkit.IPCKit.so
│
└── sources/
    └── communication_cangjie_wrapper/
        └── ...                     # 源码/头文件
```

---

## 运行时加载关系

### 库依赖图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Cangjie 应用                                 │
│                    import kit.IPCKit.*                              │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ 编译时链接
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      libkit.IPCKit.so                               │
│                         (Kit 层)                                     │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ 运行时依赖
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       libohos.rpc.so                                │
│                      (核心实现层)                                    │
│  ┌──────────────┬──────────────┬──────────────┐                    │
│  │ MessageSequence │  Ashmem   │ Parcelable   │                    │
│  └──────────────┴──────────────┴──────────────┘                    │
└──────────┬─────────────────┬──────────────────┬─────────────────────┘
           │                 │                  │
           ▼                 ▼                  ▼
┌──────────────────┐ ┌──────────────┐ ┌─────────────────────┐
│ libohos.ffi.so   │ │ libipc_...   │ │ libohos.hilog.so   │
│ (FFI 基础)        │ │ (IPC 底层)   │ │ (日志)              │
└──────────────────┘ └──────────────┘ └─────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                      Kernel Space                            │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │  Binder Driver  │  │ SoftBus Driver  │                   │
│  └─────────────────┘  └─────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

### 加载时序

```
应用启动
    │
    ├──> 加载 libkit.IPCKit.so (如使用 Kit 层)
    │       │
    │       └──> 加载 libohos.rpc.so
    │               │
    │               ├──> 加载 libohos.ffi.so
    │               ├──> 加载 libohos.hilog.so
    │               └──> 延迟加载 libipc_cj_ffi.so (首次 IPC 调用)
    │
    └──> 应用运行
            │
            ├──> 首次 IPC/RPC 调用
            │       │
            │       └──> 加载 libipc_cj_ffi.so
            │               │
            │               └──> 初始化 Binder/SoftBus 连接
            │
            └──> 后续调用直接使用已建立连接
```

---

## 运行时资源占用

### 内存占用（基于 bundle.json）

| 指标 | 数值 | 说明 |
|------|------|------|
| ROM | 300KB | 库文件总大小 |
| RAM | 228KB | 运行时内存占用 |

### 内存分布估计

```
RAM 228KB 分布:
├── 库代码段        ~100KB    (共享，多进程复用)
├── 库数据段        ~20KB
├── 堆内存          ~80KB     (Ashmem/Buffer 等)
├── 栈内存          ~20KB
└── 其他            ~8KB
```

### 峰值内存场景

1. **大 Ashmem 传输**:
   - 创建 128MB Ashmem
   - 峰值内存 += 128MB（映射内存）

2. **大量并发 IPC**:
   - 每个连接占用独立缓冲区
   - 默认缓冲区 200KB/连接

---

## 运行时配置

### 环境变量

| 变量 | 用途 | 默认值 |
|------|------|--------|
| `OHOS_IPC_LOG_LEVEL` | IPC 日志级别 | INFO |
| `OHOS_CANGJIE_DEBUG` | Cangjie 调试模式 | 0 |

### 系统属性

```
# 查看 IPC 相关属性
getprop | grep ipc
getprop | grep rpc

# 设置日志级别
setprop persist.sys.hilog.debug 1
```

---

## 部署检查清单

### 预部署检查

- [ ] 目标系统版本支持 API Level 22+
- [ ] 目标设备类型为 standard
- [ ] 底层 `ipc` 组件已部署
- [ ] 依赖库 `cangjie_ark_interop` 已部署
- [ ] 依赖库 `hiviewdfx_cangjie_wrapper` 已部署

### 部署后验证

```bash
# 1. 检查库文件存在
ls -la /system/lib/libohos.rpc.so
ls -la /system/lib/libkit.IPCKit.so

# 2. 检查依赖满足
ldd /system/lib/libohos.rpc.so

# 3. 检查符号导出
nm -D /system/lib/libohos.rpc.so | grep MessageSequence

# 4. 运行简单测试（如有）
# ...
```

---

## 升级策略

### 向后兼容

| 组件 | 兼容性 | 说明 |
|------|--------|------|
| libohos.rpc.so | 需同步升级 | 与底层 ipc 组件绑定 |
| libkit.IPCKit.so | 向后兼容 | 仅导出接口 |
| API 接口 | 向后兼容 | 遵循 OpenHarmony API 规范 |

### 升级步骤

1. 停止相关服务
2. 备份旧版本库
3. 部署新版本库
4. 重启服务
5. 验证功能

---

## 参考文档

- [GN Targets](05_GN_Targets.md) - 构建目标说明
- [常见问题](08_Troubleshooting.md) - 运行时问题诊断
