# 配置开关详解

## Feature Flags

### dataclassification_feature_enabled

**定义位置**：`interfaces/inner_api/datatransmitmgr/BUILD.gn:24`

```gn
declare_args() {
  dataclassification_feature_enabled = true
  if (defined(global_parts_info) &&
      !defined(global_parts_info.commonlibrary_c_utils)) {
    dataclassification_feature_enabled = false
  }
}
```

**类型**：Boolean

**默认值**：`true`

**作用**：总开关，控制整个 dataclassification 模块是否参与编译

**依赖条件**：
- 依赖 `c_utils` 组件
- 若 `global_parts_info` 中未定义 `commonlibrary_c_utils`，则自动禁用

**使用场景**：
- 资源受限设备可选择禁用
- 调试时可临时禁用

---

## OS Level 配置

### os_level == "standard"

**定义位置**：`BUILD.gn:16`

```gn
group("dataclassification_build_module") {
  if (os_level == "standard") {
    deps = [ "interfaces/inner_api/datatransmitmgr:data_transit_mgr" ]
  }
}
```

**作用**：仅在标准系统版本启用

**说明**：
- `mini` / `small` 系统版本不编译此模块
- 适用于资源受限设备

---

## 安全加固选项

### CFI (Control Flow Integrity)

**配置**：
```gn
cfi = true
cfi_cross_dso = true
```

**作用**：
- 防止控制流劫持攻击
- 跨 DSO 调用检查

**适用版本**：仅 `os_level == "standard"`

---

### Integer Overflow Detection

**配置**：`integer_overflow = true`

**作用**：检测整数溢出漏洞

**对应编译选项**：`-fsanitize=integer`

---

### UBSAN (Undefined Behavior Sanitizer)

**配置**：`ubsan = true`

**作用**：检测未定义行为

**检测范围**：
- 整数溢出
- 内存对齐问题
- 空指针解引用

---

### Boundary Sanitizer

**配置**：`boundary_sanitize = true`

**作用**：边界检查，防止缓冲区溢出

---

### PAC (Pointer Authentication Code)

**配置**：`branch_protector_ret = "pac_ret`

**作用**：返回地址签名认证

**适用架构**：ARMv8.3+

---

## 编译宏定义

### HILOG_ENABLE

**定义位置**：`BUILD.gn:64`

```gn
defines = [ "HILOG_ENABLE" ]
```

**作用**：启用 HiLog 日志框架

**日志标签**：`LOG_TAG = "DataSl"`

**日志域**：`LOG_DOMAIN = 0xD002F04`

**禁用效果**：降级为标准 `printf` 输出

> 证据：`interfaces/inner_api/datatransmitmgr/include/dev_slinfo_log.h:19-48`

---

### _FORTIFY_SOURCE

**配置**：`BUILD.gn:67`

```gn
cflags = [ "-D_FORTIFY_SOURCE=2" ]
```

**作用**：
- 增强内存函数安全性
- 编译期检测缓冲区溢出

---

## 运行时配置

### MAX_UDID_LENGTH

**定义位置**：`interfaces/inner_api/datatransmitmgr/include/dev_slinfo_mgr.h:25`

```c
#define MAX_UDID_LENGTH 64
```

**作用**：UDID 最大长度

---

### MAX_LIST_LENGTH

**定义位置**：`frameworks/datatransmitmgr/dev_slinfo_adpt.c:23`

```c
#define MAX_LIST_LENGTH 128
```

**作用**：异步回调链表最大长度

**溢出处理**：超过限制时触发最旧回调并移除

> 证据：`dev_slinfo_adpt.c:329-332`

---

## 配置组合矩阵

| 配置组合 | standard | small | mini |
|---------|----------|-------|------|
| `dataclassification_feature_enabled=true` | ✅ 编译 | ❌ 不编译 | ❌ 不编译 |
| `dataclassification_feature_enabled=false` | ❌ 不编译 | ❌ 不编译 | ❌ 不编译 |
| CFI/ UBSAN 加固 | ✅ 启用 | ❌ 禁用 | ❌ 禁用 |

---

## 配置验证

### 检查模块是否启用

```bash
# 查看构建配置
hb build -p --gn
```

在生成的 `args.gn` 中查找：
```
dataclassification_feature_enabled = true
```

### 检查产物是否生成

```bash
# 查找产物
find out -name "libdata_transit_mgr*" -type f
```
