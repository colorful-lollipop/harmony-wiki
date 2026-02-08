# T2Stack 编译产物与运行时加载

## 编译产物清单

### 标准系统产物

| 模块 | Target | 产物类型 | 预计文件名 | 路径 |
|------|--------|----------|------------|------|
| **Fillp** | `FillpSo.open` | 动态库 | `libFillpSo.so` | `out/.../libs/` |
| **DFile** | `nstackx_dfile.open` | 动态库 | `libnstackx_dfile.so` | `out/.../libs/` |
| **NStackX Ctrl** | `nstackx_ctrl` | 动态库 | `libnstackx_ctrl.so` | `out/.../libs/` |
| **NStackX Util** | `nstackx_util.open` | 动态库 | `libnstackx_util.so` | `out/.../libs/` |
| **Congestion** | `nstackx_congestion.open` | 静态库 | `libnstackx_congestion.a` | `out/.../obj/` |

### LiteOS 产物

| 模块 | Target | 产物类型 | 预计文件名 |
|------|--------|----------|------------|
| **Fillp** | `FillpSo.open` | 动态库 | `libFillp.so` |
| **DFile** | `nstackx_dfile.open` | 动态库 | `libdfile.so` |
| **NStackX Ctrl** | `nstackx_ctrl` | 静态库/动态库 | `libnstackx_ctrl.a/.so` |
| **NStackX Util** | `nstackx_util.open` | 静态库/动态库 | `libnstackx_util.a/.so` |

---

## 安装路径

### 标准系统

```
/system/lib/ldMusl_xxx.so      # 动态链接器
/system/lib/
    ├── libhilog.so             # 日志库（依赖）
    ├── libcrypto.so            # OpenSSL（依赖）
    ├── libnstackx_util.so      # 公共模块
    ├── libnstackx_ctrl.so      # 设备发现
    ├── libnstackx_dfile.so     # 文件传输
    └── libFillp.so             # 流传输
```

### LiteOS

```
/lib/
    ├── libnstackx_util.so
    ├── libnstackx_ctrl.so
    ├── libdfile.so
    └── libFillp.so
```

---

## 运行时加载关系

### 动态库依赖图

```
应用程序
    │
    ├─→ libnstackx_ctrl.so ───┬──→ libnstackx_util.so ───→ libhilog.so
    │                        ├──→ libcjson.so
    │                        └──→ libcoap.so
    │
    ├─→ libnstackx_dfile.so ──┼──→ libnstackx_util.so
    │                        ├──→ libnstackx_congestion.a (静态链接)
    │                        └──→ libcrypto.so
    │
    └─→ libFillp.so ──────────┼──→ libnstackx_util.so
                             └──→ libhilog.so
```

### 加载顺序

1. `libhilog.so` - 最先加载（日志基础）
2. `libnstackx_util.so` - 公共模块
3. `libcrypto.so` / `libcoap.so` - 加密/协议依赖
4. `libnstackx_dfile.so` / `libnstackx_ctrl.so` / `libFillp.so` - 业务模块

### dlopen 依赖

```c
// 手动加载示例
void *handle = dlopen("libnstackx_dfile.so", RTLD_LAZY);
if (!handle) {
    // 加载失败
}
```

---

## 符号导出

### 导出符号管理

T2Stack 使用以下方式管理导出符号：

| 模块 | 导出宏 | 主要导出符号前缀 |
|------|--------|------------------|
| **Fillp** | `DLL_API` | `Ft` (FtSocket, FtBind, FtSend...) |
| **DFile** | `NSTACKX_EXPORT` | `NSTACKX_DFile*` |
| **NStackX** | `DFINDER_EXPORT` | `NSTACKX_*` |

### 符号可见性

```gn
# BUILD.gn 中的配置
ldflags = [
  "-Wl,-z,relro,-z,now",  # Full RELRO
  "-fvisibility=hidden",  # 隐藏默认符号
]

# 使用导出宏暴露公共 API
#define DLL_API __attribute__((visibility("default")))
```

---

## 资源占用

### ROM 占用

| 模块 | ROM 估计 | 说明 |
|------|----------|------|
| **Fillp** | ~1000KB | 完整的流传输协议栈 |
| **DFile** | ~800KB | 文件传输和加密模块 |
| **NStackX Ctrl** | ~600KB | 设备发现和 CoAP |
| **NStackX Util** | ~400KB | 公共基础模块 |
| **Congestion** | ~200KB | 拥塞控制算法 |
| **总计** | **~3000KB** | bundle.json:27 |

### RAM 占用

| 场景 | RAM 估计 | 说明 |
|------|----------|------|
| **Idle** | ~5MB | 仅基础模块加载 |
| **单会话传输** | ~10MB | 包含一个传输会话 |
| **多会话传输** | ~20-40MB | 多个并行传输 |
| **最大配置** | **~40MB** | bundle.json:28 |

---

## 相关文档

- [构建系统](./05_Build_System.md) - GN Targets 配置
- [API 参考](./03_CAPI_Reference.md) - 接口使用
- [安全评审](./07_Security_Review.md) - 安全风险分析

---

*文档版本：1.0.0*
*最后更新：2026-02-06*
*数据来源：bundle.json 配置和 BUILD.gn 分析*
