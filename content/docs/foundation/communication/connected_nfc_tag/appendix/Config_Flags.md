# 附录：配置开关说明

## 1. 编译时配置

### 1.1 GN 变量 (connected_nfc_tag.gni)

| 变量名 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `NFC_TAG_DIR` | string | `//foundation/communication/connected_nfc_tag` | 组件根目录 |
| `connected_nfc_tag_only_system_app_access_api` | bool | `false` | 系统应用访问控制 |

### 1.2 编译器标志

#### common_cflags

| 标志 | 值 | 说明 |
|------|------|------|
| `-D_FORTIFY_SOURCE` | `2` | 运行时缓冲区溢出检测 |
| `-fdata-sections` | - | 数据段优化 |
| `-ffunction-sections` | - | 函数段优化 |
| `-Os` | - | 优化代码大小 |
| `-O2` | - | 优化执行速度 |

#### global_sanitize

| 标志 | 值 | 说明 |
|------|------|------|
| `boundary_sanitize` | `true` | 边界检查 |
| `cfi` | `true` | 控制流完整性 |
| `cfi_cross_dso` | `true` | DSO 间 CFI |
| `integer_overflow` | `true` | 整数溢出检测 |
| `ubsan` | `true` | 未定义行为检测 |

---

## 2. 运行时配置

### 2.1 服务配置 (nfc_tag_service.cfg)

| 配置项 | 值 | 说明 |
|--------|------|------|
| `uid` | `nfc_tag` | 服务运行用户 |
| `gid` | `["nfc_tag", "shell"]` | 服务运行组 |
| `secon` | `u:r:nfc_tag_service:s0` | SELinux 上下文 |
| `caps` | `["CAP_NET_BIND_SERVICE", "CAP_NET_RAW"]` | Linux 能力 |

### 2.2 SA 配置 (1148.json)

| 配置项 | 值 | 说明 |
|--------|------|------|
| `name` | `1148` | SA ID |
| `libpath` | `libnfc_tag_service.z.so` | 服务库路径 |
| `run-on-create` | `true` | 随系统启动 |
| `distributed` | `false` | 非分布式服务 |
| `dump_level` | `1` | dump 级别 |

---

## 3. 特性开关

### 3.1 connected_nfc_tag_only_system_app_access_api

**作用**: 控制 NFC Tag API 是否仅限系统应用访问。

**启用方式**:

```bash
# 方式 1: gn 参数
gn gen out/release --args="global_parts_info={ connected_nfc_tag_only_system_app_access_api = true }"

# 方式 2: 在产品配置中
parts {
  connected_nfc_tag {
    features += [
      "connected_nfc_tag_only_system_app_access_api"
    ]
  }
}
```

**效果**:
- 启用: `nfc_tag_sys_perm.cpp` 加入编译，强制系统应用检查
- 禁用: 所有应用均可调用

---

## 4. 日志配置

### 4.1 日志域和标签

| 配置 | 值 |
|------|------|
| Log Domain | `0xD000308` |
| Log Tag | `NFCTAG` |

### 4.2 日志宏使用

| 宏 | 级别 | 用途 |
|------|------|------|
| `HILOGD()` | DEBUG | 调试信息 |
| `HILOGI()` | INFO | 一般信息 |
| `HILOGW()` | WARN | 警告 |
| `HILOGE()` | ERROR | 错误 |
| `HILOGF()` | FATAL | 致命错误 |

---

## 5. 限制参数

### 5.1 运行时限制

| 参数 | 值 | 说明 |
|------|------|------|
| `MAX_LISTENER_NUM` | `30` | 最大回调监听数 |
| `NOTIFY_TYPE_LEN` | `64` | 事件类型字符串长度 |
| `NFC_TAG_MAX_LEN` | `512` | NDEF 数据最大长度 |

---

## 6. 配置文件位置

| 配置文件 | 路径 |
|----------|------|
| GN 配置 | `connected_nfc_tag.gni` |
| 服务配置 | `services/etc/init/nfc_tag_service.cfg` |
| SA 描述 | `sa_profile/1148.json` |
| 组件描述 | `bundle.json` |
| 版本脚本 | `services/libnfc_tag_service_version_script.txt` |

---

## 7. 相关文档

| 文档 | 链接 |
|------|------|
| 构建说明 | [04_Build.md](./04_Build.md) |
| 安全评估 | [05_Security.md](./05_Security.md) |
