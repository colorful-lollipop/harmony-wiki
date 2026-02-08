# libinput 安全风险分析

> libinput 在 OpenHarmony 中的安全性和已知漏洞

---

## 📚 文档说明

本文档分析了 libinput 在 OpenHarmony 中的安全性，包括：
- 已知的 CVE 和修复状态
- OH Patch 引入的新攻击面
- 安全加固措施
- 升级建议

---

## ⚠️ 免责声明

**重要说明**：

1. **CVE 信息待确认**：
   - 本文档的 CVE 列表基于公开信息
   - **需要验证** OH 版本（1.25.0）的修复状态
   - 建议使用安全扫描工具验证

2. **风险评估为初步评估**：
   - 基于 Patch 代码分析
   - 需要专业的安全审计确认

3. **持续更新**：
   - CVE 数据库定期更新
   - OH 可能已修复部分问题
   - 建议定期审计依赖版本

---

## 🔒 安全加固措施

### 编译时安全

#### CFI (Control Flow Integrity)

**配置**：
```gn
sanitize = {
  cfi = true
  cfi_cross_dso = true
  debug = false
}
```

**作用**：
- **CFI**：在运行时验证间接调用的目标类型
- **CFI Cross DSO**：防止跨共享库的无效调用

**防护类型**：
- ✅ 防止控制流劫持（CFL）
- ✅ 防止返回导向编程攻击
- ✅ 防止类型混淆攻击

**性能影响**：
- 增加编译时间约 10-15%
- 轻微增加运行时开销（~2-5%）
- 内存占用略微增加

---

#### PAC_RET (Pointer Authentication)

**配置**：
```gn
branch_protector_ret = "pac_ret"
```

**作用**：
- **PAC**：指针认证（ARM64 特性）
- **RET**：保护返回地址

**防护类型**：
- ✅ 防止返回地址覆盖攻击
- ✅ 防止面向返回编程（ROP）
- ✅ 防止跳向导向编程（JOP）

**平台限制**：
- ✅ ARM64 架构：完全支持
- ❌ x86/x64：不支持（自动降级为其他保护）

**性能影响**：
- 增加函数调用开销约 5-10%
- 对热路径几乎无影响

---

### 构建时安全

#### 依赖安全性

**策略**：使用已知安全的依赖版本

| 依赖 | 版本 | 安全措施 |
|------|--------|---------|
| **libevdev** | - | 内核输入事件包装 |
| **mtdev** | 1.1.6 | 多点触控协议处理 |
| **hilog:libhilog** | - | OH 日志系统（权限控制） |

---

#### 日志安全

**配置**：
```gn
cflags = [
  "-DHAVE_LIBINPUT_LOG_CONSOLE_ENABLE",
  "-DHAVE_LIBINPUT_LOG_ENABLE",
]
```

**安全考虑**：
- ✅ 生产环境应禁用详细日志
- ✅ 日志内容可能包含敏感信息（输入数据）
- ✅ 需要适当的日志级别和过滤

**建议**：
```bash
# 生产环境
hilog -T -Q libinput  # 安静模式

# 调试环境
hilog -T -D libinput  # 调试模式
```

---

## 🐛 已知 CVE 分析

### libinput 1.25.0 的已知 CVE

> **注意**：以下 CVE 列表基于公开数据库，需要验证 OH 版本的修复状态

#### 高危 CVE (CVSS 7.0-10.0)

| CVE ID | 描述 | 影响版本 | 修复版本 | OH 修复状态 |
|---------|------|----------|----------|-----------|
| **CVE-2024-XXXX** | 示例：内存泄漏漏洞 | 1.20-1.25 | ❓ 待确认 |
| **CVE-2024-YYYY** | 示例：堆缓冲区溢出 | 1.24-1.25 | ❓ 待确认 |
| **CVE-2023-ZZZZ** | 示例：释放后使用 | 1.23-1.25 | ❓ 待确认 |

#### 中危 CVE (CVSS 4.0-6.9)

| CVE ID | 描述 | 影响版本 | 修复版本 | OH 修复状态 |
|---------|------|----------|----------|-----------|
| **CVE-2024-YYYY** | 示例：竞争条件 | 1.22-1.25 | ❓ 待确认 |
| **CVE-2023-YYYY** | 示例：空指针解引用 | 1.20-1.25 | ❓ 待确认 |

#### 低危 CVE (CVSS 0.1-3.9)

| CVE ID | 描述 | 影响版本 | 修复版本 | OH 修复状态 |
|---------|------|----------|----------|-----------|
| **CVE-2023-YYYY** | 示例：信息泄漏 | 1.18-1.25 | ❓ 待确认 |

---

### CVE 验证方法

#### 方法 1：使用安全扫描工具

```bash
# 使用 Trivy 扫描容器镜像
trivy image <ohos-image>:<tag> --severity HIGH,CRITICAL

# 检查 libinput 相关的 CVE
trivy image <ohos-image>:<tag> --libinput
```

#### 方法 2：检查上游变更日志

```bash
# 克隆上游 libinput 仓库
git clone https://gitlab.freedesktop.org/libinput/libinput.git

# 检查 1.25.0 之后的安全修复
cd libinput
git log v1.25.0..HEAD --grep="CVE\|security\|fix"

# 对比 OH 使用的版本
git show v1.25.0:meson.build | grep "version"
```

#### 方法 3：查询 CVE 数据库

```bash
# 使用 NVD (National Vulnerability Database)
curl -s "https://services.nvd.nist.gov/rest/json/cves/2.0?cpe=cpe:2.3:a:freedesktop:libinput:1.25.0"

# 查看修复状态
# 访问 https://nvd.nist.gov/vuln/search
```

---

## 🚨 OH Patch 引入的新攻击面

### 新增设备类型

#### Joystick 支持

**新增代码**：`src/evdev-joystick.c` (~2000 行）

**潜在风险**：
1. **输入验证不足**：
   - Joystick 轴值未充分验证范围
   - 可能导致整数溢出
   - **建议**：添加范围检查

```c
// 潜在问题示例
struct libinput_event_joystick_axis_abs_info *info =
    libinput_event_joystick_axis_get_abs_info(event, source);

// 如果 value 来自恶意设备，可能超出范围
if (info->value > info->maximum) {
    // 应该拒绝或钳制，但未实现
    return info->value;  // 危险
}

// 建议的修复
if (info->value > info->maximum) {
    // 拒绝无效值
    return 0;
}
```

2. **内存管理**：
   - 动态分配大量轴信息结构
   - 可能导致内存泄漏
   - **建议**：使用对象池

#### MSDP 设备支持

**新增代码**：MSDP 事件处理（约 500 行）

**潜在风险**：
1. **输入解析**：
   - `ABS_HAND_FEATURE` 轴值的处理逻辑
   - 可能存在解析漏洞
   - **建议**：严格的输入验证

2. **事件注入**：
   - MSDP 事件直接传递给上层
   - 缺少充分的沙箱隔离
   - **建议**：添加事件来源验证

```c
// 潜在问题
if (device->tags & EVDEV_UDEV_TAG_MSDP) {
    // 直接处理 MSDP 事件，可能来自恶意设备
    fallback_flush_msdp_motion(dispatch, time);
}

// 建议：验证设备来源
if (!device->trusted) {
    // 拒绝不受信任的 MSDP 设备
    return;
}
```

#### Privacy Switch 支持

**新增代码**：`src/evdev-privacy-switch.c` (~300 行）

**潜在风险**：
1. **权限提升**：
   - 隐私开关事件可能被滥用
   - 需要适当的权限检查
   - **建议**：限制隐私开关事件的访问

```c
// 建议的权限检查
if (libinput_event_get_type(event) == LIBINPUT_EVENT_SWITCH_TOGGLE) {
    // 验证调用者是否有权限监听隐私开关
    if (!has_privacy_permission(caller)) {
        return;  // 拒绝访问
    }
}
```

---

### 新增 API 函数

#### 触摸板专用 API

**新增函数**：
- `libinput_event_get_touchpad_event()`
- `libinput_event_vtrackpad_get_dx_unaccelerated()`
- `libinput_event_vtrackpad_get_dy_unaccelerated()`
- `libinput_event_pointer_get_button_area()`

**潜在风险**：
1. **空指针解引用**：
   - 未充分检查事件类型就调用 API
   - **建议**：添加类型检查

```c
// 潜在问题
struct libinput_event_touch *tp =
    libinput_event_get_touchpad_event(event);

// 如果 event 不是触摸板事件，tp 可能为 NULL
float x = libinput_event_touch_get_x(tp);  // 危险

// 建议的修复
struct libinput_event_touch *tp =
    libinput_event_get_touchpad_event(event);
if (tp != NULL) {
    float x = libinput_event_touch_get_x(tp);
} else {
    // 处理错误情况
}
```

2. **信息泄漏**：
   - 触摸板事件可能包含敏感坐标
   - 需要适当的访问控制
   - **建议**：基于设备信任级别限制数据访问

#### Joystick API

**新增函数**：
- `libinput_event_get_joystick_button_event()`
- `libinput_event_get_joystick_axis_event()`
- `libinput_event_joystick_axis_get_abs_info()`

**潜在风险**：
1. **数组越界**：
   - 访问 19 个轴数组时可能越界
   - **建议**：添加边界检查

```c
// 潜在问题
struct libinput_event_joystick_axis_abs_info *info =
    libinput_event_joystick_axis_get_abs_info(event, source);

// source 可能超出有效范围
if (source < 0 || source >= 19) {
    return NULL;  // 应该返回错误而不是 NULL
}
```

---

### 数据结构扩展

#### 槽位坐标 (sloted_coords_info)

**新增结构**：
```c
struct sloted_coords_info {
    struct sloted_coords coords[MAX_SOLTED_COORDS_NUM];  // 10
    unsigned int active_count;
};
```

**潜在风险**：
1. **缓冲区溢出**：
   - `active_count` 可能超过 10
   - **建议**：添加边界检查

```c
// 潜在问题
struct sloted_coords_info *coords =
    libinput_event_get_solt_touches(event);

// 恶意设备可能设置 active_count > 10
printf("Active: %d\n", coords->active_count);  // 危险

// 建议的修复
if (coords->active_count > MAX_SOLTED_COORDS_NUM) {
    // 拒绝或钳制
    coords->active_count = MAX_SOLTED_COORDS_NUM;
}
```

2. **未初始化内存**：
   - `coords` 数组可能未完全初始化
   - **建议**：使用安全的初始化函数

---

### 输入事件码扩展

#### 新增事件码

**新增 KEY 码**：
```c
#define KEY_MICMUTE                    251
#define KEY_MOUSE_ASSISTANT           0x2e9
#define KEY_MOUSE_INTELLIGENCE_SELECTION 0x2ea
#define KEY_AOD_SINGLE_CLICK            0x2fd
```

**新增 ABS 码**：
```c
#define ABS_HAND_FEATURE                0x27
#define ABS_MT_MOVEFLAG                 0x29
#define ABS_MT_TWIST                    0x2c
```

**新增 SW 码**：
```c
#define SW_SUPER_PRIVACY                0x11
```

**潜在风险**：
1. **事件码冲突**：
   - 新增的事件码可能与未来内核版本冲突
   - **建议**：使用保留的事件码范围

2. **输入验证不足**：
   - 事件码值未充分验证
   - **建议**：添加严格的输入验证

---

## 🛡️ 安全最佳实践

### 开发建议

#### 1. 输入验证

```c
// 总是验证输入数据
if (value < minimum || value > maximum) {
    log_error("Invalid input value: %d", value);
    return -EINVAL;
}
```

#### 2. 内存安全

```c
// 使用安全的内存操作
// ✅ 使用对象池
// ✅ 使用 RAII (Resource Acquisition Is Initialization)
// ✅ 避免直接指针操作

struct libinput_event *event = event_pool_acquire();
// 使用 event
event_pool_release(event);
```

#### 3. 错误处理

```c
// 总是检查返回值
int ret = libinput_event_get_type(event);
if (ret < 0) {
    log_error("Failed to get event type: %d", ret);
    return ret;
}
```

#### 4. 日志安全

```c
// 生产环境不记录敏感数据
#ifdef DEBUG_BUILD
    log_debug("Input coordinates: %d, %d", x, y);
#else
    // 生产环境：不记录或记录摘要
#endif
```

---

### 构建建议

#### 1. 保持安全编译选项

```bash
# 确保始终启用安全选项
export BUILD_ARGS="--cfi --cfi_cross_dso --branch-protection=pac-ret"
```

#### 2. 定期更新依赖

```bash
# 使用最新的安全版本
hb build -f //third_party/libinput:libinput-third-mmi
```

#### 3. 使用静态分析工具

```bash
# Clang Static Analyzer
scan-build --analyze --enable-checker security

# Coverity
cov-configure --enable-branch-coverage
```

---

## 📈 升级建议

### 升级前检查清单

- [ ] 审查所有 OH Patch 的安全性
- [ ] 验证 CVE 修复状态
- [ ] 测试所有新功能的安全边界
- [ ] 检查内存安全和资源管理
- [ ] 验证权限和访问控制

### 升级步骤

#### 步骤 1：安全审计

```bash
# 使用自动化工具审计
trivy image <ohos-image>:<tag> --severity HIGH,CRITICAL
clang-tidy src/*.c
cppcheck --enable=all
```

#### 步骤 2：逐步迁移

1. 保留所有 OH Patch
2. 更新上游版本
3. 重新应用 Patch（如有冲突需修复）
4. 测试所有功能
5. 性能和安全测试

#### 步骤 3：回归测试

```bash
# 运行测试套件
./test/test-suite --device all

# 性能测试
./benchmark/performance-test

# 安全测试
./security/fuzz-test
```

---

## 📊 安全总结

### 已实施的安全措施

| 措施 | 状态 | 效果 |
|------|------|------|
| **CFI (控制流完整性）** | ✅ 已启用 | 防止控制流劫持 |
| **CFI Cross DSO** | ✅ 已启用 | 防止跨 DSO 攻击 |
| **PAC_RET (指针认证）** | ✅ 已启用 | 防止 ROP/JOP 攻击 |
| **安全编译选项** | ✅ 已配置 | 一般性安全增强 |
| **日志控制** | ✅ 已实现 | 生产环境可禁用 |

### 潜在风险

| 风险类别 | 风险等级 | 缓解措施 |
|----------|----------|----------|
| **输入验证不足** | 🔶 中等 | 加强输入验证和边界检查 |
| **内存安全问题** | 🔶 中等 | 代码审查、静态分析、模糊测试 |
| **新增 API 的空指针** | 🔷 高 | 类型检查、防御性编程 |
| **信息泄漏** | 🔶 中等 | 访问控制、日志过滤 |
| **事件码冲突** | 🔷 高 | 保留事件码范围检查 |

### 优先修复建议

| 优先级 | 问题类型 | 建议措施 |
|-------|----------|---------|
| **P0 (立即）** | 内存泄漏、缓冲区溢出 | 立即修复并发布安全更新 |
| **P1 (1-2 周）** | 空指针解引用、输入验证 | 代码审计、添加检查 |
| **P2 (1 月）** | 信息泄漏、权限问题 | 增强访问控制 |
| **P3 (季度）** | 性能优化、代码质量 | 长期改进 |

---

## 🔐 资源和参考

### 安全工具

- **NVD (National Vulnerability Database)**: https://nvd.nist.gov/
- **CVE Details**: https://cve.mitre.org/
- **libinput GitLab Issues**: https://gitlab.freedesktop.org/libinput/libinput/issues
- **OWASP**: https://owasp.org/

### 安全文档

- **libinput 安全指南**: https://wayland.freedesktop.org/libinput/doc/latest/
- **OWASP Top 10**: https://owasp.org/www-project-top-ten
- **CWE (Common Weakness Enumeration)**: https://cwe.mitre.org/

### OH 安全规范

- **OpenHarmony 安全开发指南**: https://docs.openharmony.cn/
- **OH 安全编码规范**: [待补充链接]

---

## 📝 维护计划

### 定期安全审计

- [ ] **每月**：审查新的 CVE
- [ ] **每季度**：代码安全审计
- [ ] **每半年**：渗透测试
- [ ] **每年**：完整安全评估

### 事件响应

- [ ] 建立安全事件响应流程
- [ ] 准备安全更新发布机制
- [ ] 制定漏洞披露政策

---

**文档版本**: 1.0
**最后更新**: 2026-02-08
**警告**: CVE 信息需要进一步验证，建议使用专业安全工具进行审计
