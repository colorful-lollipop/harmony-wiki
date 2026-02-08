# 编译产物

## 产物清单

| 产物类型 | 路径 | 说明 |
|---------|------|------|
| 可执行文件 | `out/{product}/foundation/systemabilitymgr/safwk_lite/foundation` | 主进程 |
| Map 文件 | `out/{product}/foundation/systemabilitymgr/safwk_lite/foundation.map` | 符号映射表 |
| 动态库 | 依赖子服务组件 | abilityms, bundlems 等 |

## 可执行文件详情

### foundation

| 属性 | 值 |
|------|-----|
| 文件路径 | `foundation/systemabilitymgr/safwk_lite/foundation` |
| 文件类型 | ELF 可执行文件 |
| 架构 | ARM (32-bit) / x86 |
| 依赖内核 | LiteOS-A, Linux |

### 文件大小

| 组成部分 | 大小 (参考) |
|---------|-------------|
| 代码段 | ~50KB |
| 数据段 | ~10KB |
| 总计 (ROM) | **~100KB** |
| 运行时 (RAM) | **~2MB** |

## 安装路径

### 系统分区

| 设备类型 | 安装路径 |
|---------|---------|
| 小型设备 | `/bin/foundation` |
| 开发板 | `/system/bin/foundation` |

### 开发板映射

| 开发板 | 默认路径 |
|--------|---------|
| HiSpark_Aries | `/system/bin/foundation` |
| 其他 | `/bin/foundation` |

## 运行时加载关系

### 依赖库加载顺序

```
1. libc.so (C 标准库)
2. libstdc++.so (C++ 标准库)
3. libhilog.so (日志库)
4. libsamgr.so (Samgr 框架)
5. libipc_auth.so (IPC 认证)
6. libpms.so (权限管理)
7. 子服务 .so (abilityms, bundlems 等)
```

### dlopen 动态加载

部分子服务可能通过 `dlopen()` 动态加载：

```c
// 示例逻辑（伪代码）
void *handle = dlopen("libabilityms.so", RTLD_LAZY);
void (*Init)(void) = dlsym(handle, "ServiceInit");
Init();
```

**说明**: 具体加载机制取决于子服务实现。

## 运行时配置

### 配置文件路径

| 配置类型 | 路径 | 说明 |
|---------|------|------|
| 系统能力配置 | `/etc/sa_config/` | 各 SA 配置文件 |
| 启动脚本 | `/init.d/` | 启动参数 |

### 环境变量

| 变量 | 说明 |
|------|------|
| `DEBUG_SERVICES_SAFWK_LITE` | 启用调试日志 |

## 产物验证

### 检查产物

```bash
# 查看可执行文件信息
file out/{product}/foundation/systemabilitymgr/safwk_lite/foundation

# 查看依赖库
readelf -d out/{product}/foundation/systemabilitymgr/safwk_lite/foundation

# 查看符号表
nm out/{product}/foundation/systemabilitymgr/safwk_lite/foundation
```

### 运行时检查

```bash
# 查看进程
ps | grep foundation

# 查看内存
cat /proc/{pid}/status | grep VmRSS
```
