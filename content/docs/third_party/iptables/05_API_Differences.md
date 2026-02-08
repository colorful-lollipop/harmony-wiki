# API/接口差异

## 概述

iptables 在 OpenHarmony 中的适配属于**纯构建适配**，不涉及 API 或接口的功能性修改。

| 维度 | 状态 | 说明 |
|-----|------|------|
| OH 新增 API | 无 | iptables 未扩展新 API |
| 行为变更 | 无 | 保持上游行为一致 |
| 废弃功能 | 无 | 未禁用任何上游功能 |
| 接口兼容性 | 完全兼容 | 与上游 1.8.x 版本一致 |

---

## 与上游版本对比

### 命令行接口 (CLI)

OpenHarmony 中的 iptables 命令行与上游完全兼容：

```bash
# 所有标准命令均可用
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables-save > /data/iptables.rules
iptables-restore < /data/iptables.rules
```

#### 命令行参数对比

| 参数 | 上游 | OH 版本 | 差异 |
|-----|------|--------|------|
| 基础操作 (-A, -D, -I) | ✅ | ✅ | 无 |
| 表选择 (-t) | ✅ | ✅ | 无 |
| 匹配模块 (-m) | ✅ | ✅ | 无 |
| 目标动作 (-j) | ✅ | ✅ | 无 |
| 链管理 (-N, -X) | ✅ | ✅ | 无 |
| 规则保存/恢复 | ✅ | ✅ | 无 |

### 库接口 (libiptc)

#### C API 对比

```c
// 上游 libiptc 接口 - 完全保留
#include <libiptc/libiptc.h>

// 初始化句柄
struct xtc_handle *h = iptc_init("filter");

// 遍历规则
const struct ipt_entry *e = iptc_first_rule("INPUT", h);
while (e) {
    // 处理规则
    e = iptc_next_rule(e, h);
}

// 提交更改
iptc_commit(h);
iptc_free(h);
```

**状态**: OH 版本完全保留上游 API，无修改。

### xtables 扩展接口

扩展模块的编写接口保持不变：

```c
// 标准 xtables 扩展模板 - 完全兼容
#include <xtables.h>

static struct xtables_match my_match = {
    .family        = NFPROTO_UNSPEC,
    .name          = "myMatch",
    .version       = XTABLES_VERSION,
    .size          = XT_ALIGN(sizeof(struct my_match_info)),
    .userspacesize = XT_ALIGN(sizeof(struct my_match_info)),
    .help          = my_match_help,
    .parse         = my_match_parse,
    .final_check   = my_match_check,
    .print         = my_match_print,
    .save          = my_match_save,
};

void _init(void) {
    xtables_register_match(&my_match);
}
```

---

## OH 特有的间接接口

虽然 iptables 本身未新增 API，但 OpenHarmony 的上层模块提供了**包装接口**：

### 1. NetManager Wrapper 接口

**位置**: `foundation/communication/netmanager_base/services/netmanagernative/include/iptables_wrapper.h`

```cpp
// OH 封装的 C++ 接口
namespace OHOS {
namespace NetManagerStandard {

class IptablesWrapper {
public:
    // 单例访问
    static IptablesWrapper& GetInstance();
    
    // 执行 iptables 命令
    int32_t RunIptablesCommand(const std::string& command);
    int32_t RunIp6tablesCommand(const std::string& command);
    
    // 批量操作
    int32_t RunIptablesCommands(const std::vector<std::string>& commands);
    
    // 规则持久化
    int32_t SaveRules(const std::string& filepath);
    int32_t RestoreRules(const std::string& filepath);
};

} // namespace NetManagerStandard
} // namespace OHOS
```

**说明**: 这是 OH **上层封装**，不是 iptables 本身的新增 API。

### 2. EDM 管理接口

**位置**: `base/customization/enterprise_device_management/common/native/include/iptables_utils.h`

```cpp
// EDM 封装的 iptables 工具函数
namespace OHOS {
namespace EDM {

class IptablesUtils {
public:
    // 执行命令并获取结果
    static int32_t ExecuteCommand(const std::string& cmd, std::string& result);
    
    // 检查规则是否存在
    static bool IsRuleExists(const std::string& chain, const std::string& rule);
    
    // 清空特定链
    static int32_t FlushChain(const std::string& table, const std::string& chain);
    
    // 创建/删除自定义链
    static int32_t CreateChain(const std::string& table, const std::string& chain);
    static int32_t DeleteChain(const std::string& table, const std::string& chain);
};

} // namespace EDM
} // namespace OHOS
```

**说明**: EDM 提供的**高级管理接口**，底层仍调用标准 iptables。

---

## 功能可用性

### 标准功能

所有 iptables 标准功能在 OH 中均可用：

| 功能类别 | 功能 | OH 可用性 |
|---------|------|----------|
| **表** | filter | ✅ 完全可用 |
| | nat | ✅ 完全可用 |
| | mangle | ✅ 完全可用 |
| | raw | ✅ 完全可用 |
| | security | ✅ 完全可用 |
| **扩展匹配** | conntrack | ✅ 可用 |
| | multiport | ✅ 可用 |
| | iprange | ✅ 可用 |
| | time | ✅ 可用 |
| | string | ✅ 可用 |
| | limit | ✅ 可用 |
| | (其他 60+) | ✅ 均可用 |
| **扩展目标** | ACCEPT/DROP | ✅ 可用 |
| | REJECT | ✅ 可用 |
| | LOG | ✅ 可用 |
| | SNAT/DNAT | ✅ 可用 |
| | MASQUERADE | ✅ 可用 |
| | REDIRECT | ✅ 可用 |
| | MARK | ✅ 可用 |
| | (其他 20+) | ✅ 均可用 |

### 可能受限的功能

以下功能可能受 OH 内核配置影响：

| 功能 | 依赖 | OH 状态 |
|-----|------|--------|
| connlabel | CONFIG_NETFILTER_XT_MATCH_CONNLABEL | 需确认 |
| ipvs | CONFIG_IP_VS | 需确认 |
| nfacct | CONFIG_NETFILTER_NETLINK_ACCT | 需确认 |
| osf | CONFIG_NETFILTER_XT_MATCH_OSF | 需确认 |

**说明**: 这些扩展模块已编译进 iptables，但功能依赖内核配置。实际使用前需验证目标设备的内核配置。

---

## 移植注意事项

### 从 Linux 到 OH

如果你的应用/服务从标准 Linux 移植到 OpenHarmony：

#### ✅ 无需修改

```bash
# 所有标准 iptables 命令可直接使用
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

#### ⚠️ 需要注意

```bash
# 路径差异 - OH 中 iptables 位于 /system/bin/
# Linux 通常在 /sbin/iptables 或 /usr/sbin/iptables

# 规则持久化路径
# OH 建议使用 /data/ 目录
iptables-save > /data/iptables.rules

# 某些扩展可能依赖特定内核配置
# 使用前最好测试
```

#### ❌ 不支持的

```bash
# OH 使用静态链接，不支持动态扩展加载
# 以下操作无效：
iptables -m my_custom_module  # 无法加载外部 .so 文件

# 但所有标准扩展均已静态链接，可直接使用
iptables -m string --string "block" --algo bm -j DROP  # ✅ 可用
```

---

## 版本兼容性

### iptables 1.8.x 系列

OpenHarmony 使用的 iptables 版本（1.8.7/1.8.11）属于 1.8.x 系列：

| 版本 | 特性 | OH 状态 |
|-----|------|--------|
| 1.8.0 | nftables 后端支持 | ✅ 可用 |
| 1.8.1+ | 增量更新 | ✅ 可用 |
| 1.8.7 | 当前 bundle.json 版本 | ✅ 当前版本 |
| 1.8.11 | README.OpenSource 版本 | ✅ 可能已升级 |

**nftables 后端**: iptables 1.8.x 支持使用 nftables 作为后端（通过 `iptables-nft`），但 OH 版本使用传统 `iptables-legacy` 模式。

### 升级兼容性

从 1.8.x 升级到 1.8.y：
- ✅ 向后兼容
- ✅ 规则格式不变
- ✅ API 不变
- ⚠️ 可能新增扩展模块（需要更新 BUILD.gn）

---

## 接口稳定性评级

| 接口层级 | 稳定性 | 说明 |
|---------|-------|------|
| 命令行 | ⭐⭐⭐⭐⭐ | 极其稳定，10+ 年不变 |
| libiptc C API | ⭐⭐⭐⭐ | 稳定，但标记为 internal |
| xtables 扩展 API | ⭐⭐⭐⭐ | 稳定，版本兼容 |
| OH Wrapper (C++) | ⭐⭐⭐ | 可能随 OH 版本演进 |

---

## 总结

### 无 API 差异

iptables 在 OpenHarmony 中**没有功能性 API 差异**：
- 命令行完全兼容
- C API 完全兼容
- 扩展接口完全兼容

### 仅有构建差异

差异仅限于构建层面：
- 静态链接 vs 动态链接
- GN 构建 vs autotools
- musl libc vs glibc (已通过 Patch 解决)

### 建议

1. **直接使用标准 iptables 命令**: 无需适配
2. **参考 OH Wrapper**: 如使用 C++，建议使用 NetManager 的 IptablesWrapper
3. **验证内核支持**: 使用非标准扩展前验证内核配置
4. **规则持久化**: 使用 `/data/` 目录保存规则
