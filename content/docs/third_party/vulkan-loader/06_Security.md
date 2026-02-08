# 06 - 安全风险分析

## 6.1 已知 CVE 评估

### 当前版本状态

| 项目 | 信息 |
|------|------|
| **当前版本** | v1.4.309 |
| **Apache-2.0 许可** | 是 |
| **CVE 查询** | 建议通过 NVD (National Vulnerability Database) 查询 |

### 历史 CVE 参考

Vulkan-Loader 历史上 CVE 较少，主要安全风险类别：

| 类别 | 风险级别 | 说明 |
|------|----------|------|
| 环境变量注入 | 中 | 通过环境变量加载恶意驱动/Layer |
| JSON 解析 | 低 | 配置文件的解析漏洞 |
| 路径遍历 | 中 | 驱动/Layer 路径注入 |

**TODO**: 需查询 NVD 数据库获取具体 CVE 列表：
- https://nvd.nist.gov/vuln/search
- 搜索关键词："Vulkan-Loader", "vulkan-loader"

---

## 6.2 OH Patch 引入的攻击面

### 1. Bundle 管理器集成

**攻击面**: 应用身份验证绕过  
**风险级别**: 中  
**相关代码**: `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.cpp`

**潜在风险**:
- Bundle 信息被篡改导致非法应用通过验证
- IPC 通信被劫持

**缓解措施**:
- BundleManager 服务运行在系统进程，受 SELinux 保护
- 仅允许调试版本应用加载调试 Layer
- 应用沙箱隔离

**审计建议**:
```cpp
// 检查点 1: Bundle 验证逻辑
bool InitBundleInfo(char* debugHapName) {
    // 验证包名是否匹配当前应用
    if (vkBundleMgrHelper->g_bundleInfo.name == debugHap) {
        return true;
    }
}

// 检查点 2: 调试版本检查
bool CheckAppProvisionTypeIsDebug() {
    // 确保只有 release 类型被阻止
    if (vkBundleMgrHelper->g_bundleInfo.applicationInfo.appProvisionType == "release") {
        return false;
    }
}
```

---

### 2. 动态库加载 (namespace dlopen)

**攻击面**: 恶意库加载  
**风险级别**: 中  
**相关代码**: `loader/vk_loader_platform.h`

**潜在风险**:
- 从 `passthrough` namespace 加载恶意驱动
- 库路径注入攻击

**缓解措施**:
```cpp
// 优先从 passthrough 加载（受系统保护）
if (!dlns_get("passthrough", &ns_ps)) {
    handle = dlopen_ns(&ns_ps, libPath, LOADER_DLOPEN_MODE);
}
// 回退到默认（受沙箱限制）
if (!handle) {
    handle = dlopen(libPath, LOADER_DLOPEN_MODE);
}
```

**审计建议**:
- 确保 `passthrough` namespace 仅包含受信任的库
- 验证驱动路径在允许的目录内

---

### 3. 环境变量/系统参数处理

**攻击面**: 参数注入  
**风险级别**: 低-中  
**相关代码**: `loader/loader_environment.c`, `loader/loader.c`

**OH 特有参数**:
- `debug.graphic.debug_layer`
- `debug.graphic.debug_hap`
- `debug.graphic.system_layer_flag`
- `debug.graphic.vklayer_json_path`

**潜在风险**:
- 通过参数注入加载恶意 Layer
- 路径遍历攻击

**缓解措施**:
- 调试 Layer 功能仅对调试应用开放（Bundle 验证）
- 系统参数需要特殊权限修改
- 路径验证（TODO: 确认是否有路径遍历检查）

**审计建议**:
```cpp
// 检查点: 路径验证（待确认）
if (strstr(debug_layer_json_path, "..") != NULL) {
    // 应拒绝包含 .. 的路径
}
```

---

### 4. IPC 通信

**攻击面**: BundleManager IPC 劫持  
**风险级别**: 中  
**相关代码**: `openharmony/bundle_mgr_helper/vk_bundle_mgr_helper.cpp`

**潜在风险**:
- IPC 通信被中间人攻击
- BundleManager 服务被伪造

**缓解措施**:
- 使用 SAMGR (System Ability Manager) 获取可信服务
- IPC 受内核 SELinux 策略保护

```cpp
sptr<ISystemAbilityManager> systemAbilityManager =
    SystemAbilityManagerClient::GetInstance().GetSystemAbilityManager();
sptr<IRemoteObject> remoteObject_ = systemAbilityManager->GetSystemAbility(
    BUNDLE_MGR_SERVICE_SYS_ABILITY_ID);
```

---

## 6.3 标准 Vulkan-Loader 风险

### 1. 驱动加载安全

**风险**: 加载恶意 ICD (Installable Client Driver)

**缓解措施（OHOS）**:
- 驱动配置文件需放在受保护目录
- `/vendor/etc/vulkan/icd.d/` 需要 root/system 权限写入
- 应用无法自行安装驱动

### 2. Layer 加载安全

**风险**: 加载恶意 Layer 拦截/篡改 API 调用

**缓解措施（OHOS）**:
- 系统 Layer 路径受保护
- 用户 Layer 仅在调试模式下可用
- Bundle 验证确保应用身份

### 3. JSON 配置解析

**风险**: 配置文件解析漏洞

**缓解措施**:
- 使用标准 cJSON 库
- 配置路径限制在指定目录

---

## 6.4 权限模型

### 调试 Layer 权限

```
┌─────────────────────────────────────────────────────┐
│                   权限检查流程                        │
├─────────────────────────────────────────────────────┤
│ 1. 检查环境参数是否设置                               │
│    └── debug.graphic.debug_layer 设置？              │
│         ├── 否 → 跳过调试 Layer                      │
│         └── 是 → 继续                                │
│                                                    │
│ 2. 检查应用身份（Bundle 验证）                        │
│    └── InitBundleInfo() 成功？                       │
│         ├── 否 → 拒绝加载                            │
│         └── 是 → 继续                                │
│                                                    │
│ 3. 检查应用版本类型                                  │
│    └── CheckAppProvisionTypeIsDebug()                │
│         ├── release → 拒绝加载                       │
│         └── debug   → 允许加载                       │
└─────────────────────────────────────────────────────┘
```

---

## 6.5 安全建议

### 对于系统开发者

1. **定期更新**: 跟踪上游 Vulkan-Loader 安全更新
2. **CVE 监控**: 订阅 Khronos 安全通告
3. **审计**: 定期审计 OH 特有代码的安全边界
4. **测试**: 添加安全相关的 fuzz 测试

### 对于驱动开发者

1. **最小权限**: 驱动仅请求必要的权限
2. **输入验证**: 验证所有来自 Loader 的输入
3. **内存安全**: 使用 safe 函数（如 `memcpy_s`）

### 对于应用开发者

1. **Layer 来源**: 仅使用可信来源的 Vulkan Layer
2. **调试关闭**: 发布版本确保调试功能关闭
3. **日志保护**: 避免在日志中输出敏感信息

---

## 6.6 安全升级策略

### 紧急响应流程

1. **CVE 发现** → 评估影响范围
2. **临时缓解** → 发布安全补丁或配置规避
3. **版本升级** → 同步上游修复版本
4. **回归测试** → 验证 OH 特有功能
5. **发布更新** → 推送安全更新

### 版本维护建议

| 维护项 | 频率 | 负责方 |
|--------|------|--------|
| CVE 扫描 | 每周 | 安全团队 |
| 上游版本同步 | 每季度 | 图形团队 |
| 安全审计 | 每半年 | 安全团队 |
| Fuzz 测试 | 持续 | QA 团队 |

---

## 6.7 TODO 安全待办

- [ ] **CVE 数据库查询**: 查询 NVD 获取 Vulkan-Loader 历史 CVE 完整列表
- [ ] **路径遍历检查**: 确认 `debug_layer_json_path` 是否有路径遍历防护
- [ ] **模糊测试**: 补充 JSON 配置文件的 fuzz 测试
- [ ] **IPC 审计**: 审计 BundleManager IPC 通信安全性
- [ ] **沙箱测试**: 验证应用沙箱对 Layer 加载的隔离效果

---

*文档版本：v1.0*
*最后更新：2026-02-07*
*状态：部分完成，待 CVE 详细数据补充*
