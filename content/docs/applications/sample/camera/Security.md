# 安全风险分析

## 免责声明

本文档基于代码静态分析，仅反映分析时的代码状态。实际安全评估需结合动态测试、渗透测试等多种手段。本分析不构成正式的安全审计报告。

---

## 攻击面清单

### 1. 输入攻击面

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| **文件路径输入** | gallery 图库浏览 | 中 | 通过 Want.data 传递文件名 |
| **WiFi SSID/密码** | setting WiFi 配置 | 高 | 用户输入直接用于命令构造 |
| **应用包名** | setting 权限管理 | 低 | 通过 Want 传递 bundleName |
| **媒体文件** | player_sample | 中 | 播放外部媒体文件 |

### 2. 权限攻击面

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| **运行时权限授予** | setting/AppInfoAbilitySlice | 高 | 可授予/撤销任意应用权限 |
| **敏感权限声明** | cameraApp/config.json | 中 | CAMERA, MICROPHONE, WRITE_MEDIA |

### 3. 文件系统攻击面

| 攻击面 | 位置 | 风险等级 | 说明 |
|--------|------|----------|------|
| **照片/视频写入** | camera_manager.cpp | 中 | 写入 `/userdata/photo/`, `/userdata/video/` |
| **媒体文件读取** | gallery/player | 中 | 读取外部媒体文件 |
| **配置文件** | wpa_work.c | 高 | WiFi 配置文件操作 |

---

## 可被利用点详细分析

### 风险 1: WiFi 配置命令注入

**证据位置**: `setting/setting/src/main/cpp/wpa_work.c:381-410`

**代码片段**:
```c
// line 381
err = sprintf_s(cmd, sizeof(cmd), "SET_NETWORK %.*s ssid \"%s\"", 
                networkIdLen, networkId, gSsid);
// line 388
err = sprintf_s(cmd, sizeof(cmd), "SET_NETWORK %.*s psk \"%s\"", 
                networkIdLen, networkId, gPassWord);
```

**触发路径**:
1. 用户进入设置 -> WiFi 设置
2. 输入 WiFi SSID 和密码
3. `setting_wifi_input_password_ability_slice.cpp` 接收输入
4. 传递给 `wpa_work.c` 构造 wpa_cli 命令
5. 通过 `popen()` 或 socket 发送给 wpa_supplicant

**影响**:
- 若 SSID 或密码未过滤特殊字符，可能导致命令注入
- 虽然使用 `sprintf_s`，但如果 `gSsid` 或 `gPassWord` 包含 `"` 或 `;` 等字符，可能逃逸引号

**修复建议**:
1. 对 SSID 和密码进行严格的输入校验，只允许合法 WiFi 字符
2. 使用参数化接口而非字符串拼接
3. 对特殊字符进行转义

**验证状态**: 需确认调用 wpa_cli 的具体方式（待动态分析）

---

### 风险 2: 权限管理绕过

**证据位置**: `setting/setting/src/main/cpp/app_info_ability_slice.cpp:127-133`

**代码片段**:
```cpp
// line 127
int ret = QueryPermission(bundleName_, &permissions_, &permNum);
// line 86
ret = GrantPermission(bundleName_, name_);
// line 80
ret = RevokePermission(bundleName_, name_);
```

**触发路径**:
1. setting 应用获取了 `ohos.permission.GRANT_SENSITIVE_PERMISSIONS` 等系统权限
2. `AppInfoAbilitySlice` 通过 `Want.data` 接收 bundleName
3. 调用 Permission API 授予/撤销任意应用权限

**影响**:
- 恶意应用可能通过伪造 Want 参数，诱导 setting 修改其他应用权限
- 若 bundleName 未经验证，可能越权管理其他应用

**修复建议**:
1. 验证调用者身份，确保只有系统应用或授权应用可调用
2. 对 bundleName 进行白名单校验
3. 记录权限变更日志，便于审计

**当前状态**: setting 应用本身需要高权限，依赖系统框架进行权限控制

---

### 风险 3: 文件路径遍历

**证据位置**: `gallery/src/picture_ability_slice.cpp:135`

**代码片段**:
```cpp
// line 135
if (sprintf_s(imagePath, imagePathLen + 1, "%s/%s", 
              PHOTO_DIRECTORY, reinterpret_cast<char*>(want.data)) < 0) {
```

**触发路径**:
1. gallery 接收 Want.data 作为图片文件名
2. 直接拼接到 `PHOTO_DIRECTORY` 后形成完整路径
3. 打开文件并显示

**影响**:
- 若 `want.data` 包含 `../`，可能读取目录外文件
- 如: `"../../../etc/passwd"`

**修复建议**:
1. 对文件名进行规范化（realpath）
2. 验证最终路径是否在允许目录内
3. 使用白名单限制文件名字符集

**缓解措施**: `PHOTO_DIRECTORY` 定义为 `/userdata/photo/`，非系统关键目录

---

### 风险 4: 缓冲区溢出风险（已缓解）

**证据位置**: 多处使用 `sprintf_s`, `memcpy_s`, `strcpy_s`

**代码片段**:
```cpp
// camera_manager.cpp:196
if (sprintf_s(tmpFile, sizeof(tmpFile), "%s/photo%s.jpg", 
              PHOTO_PATH, timeStamp) < 0) {

// setting/app_info_ability_slice.cpp:145
ret = memcpy_s(bundleName_, sizeof(bundleName_), 
               want.data, want.dataLength);
```

**分析**:
- 代码中使用了安全版本的字符串/内存操作函数（带 `_s` 后缀）
- 这些函数来自 `libsec_shared`（bounds_checking_function）
- 相比不安全的 `sprintf`/`memcpy`，已大幅降低溢出风险

**潜在问题**:
- `want.dataLength` 是否可能大于 `sizeof(bundleName_)`？
- 虽然 `memcpy_s` 会检查，但需确认返回值处理

**修复建议**:
1. 始终检查 `_s` 函数的返回值
2. 对于用户输入，进行长度预检查

---

### 风险 5: 敏感信息泄露（日志）

**证据位置**: 多处 printf 日志

**代码片段**:
```cpp
// app_info_ability_slice.cpp:73
printf("[LOG] bundleName_-> %s +11->%s \n", bundleName_, bundleName_ + 11);

// app_info_ability_slice.cpp:129
printf("[LOG]PermissionInfoList bundleName_ -> %s ,permNum->%d\n", 
       bundleName_, permNum);

// camera_manager.cpp:606
printf("camera start init!!! \n");
```

**影响**:
- 日志中可能包含敏感信息（bundleName、权限状态等）
- 发布版本中若未禁用日志，可能泄露系统信息

**修复建议**:
1. 发布版本禁用调试日志（使用条件编译）
2. 敏感信息脱敏后再输出
3. 日志分级，生产环境只输出错误日志

---

## 信任边界

```
┌─────────────────────────────────────────────────────────────┐
│                        用户态                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  cameraApp  │  │   gallery   │  │   setting   │          │
│  │  (Untrusted)│  │  (Untrusted)│  │ (Privileged)│          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
│         │                │                │                  │
│         └────────────────┴────────────────┘                  │
│                          │                                   │
│  ┌───────────────────────┴───────────────────────┐          │
│  │              System Services                   │          │
│  │  (camera_lite, recorder_lite, permission_lite) │          │
│  │               (Trusted)                        │          │
│  └───────────────────────┬───────────────────────┘          │
│                          │                                   │
└──────────────────────────┼───────────────────────────────────┘
                           │
┌──────────────────────────┼───────────────────────────────────┐
│                          │                                   │
│  ┌───────────────────────┴───────────────────────┐          │
│  │                    Kernel                      │          │
│  │              (Highly Trusted)                  │          │
│  └───────────────────────────────────────────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**关键信任边界**:
1. 应用 <-> 系统服务：通过 IPC/SAMGR 通信，需权限检查
2. 用户输入 <-> 应用内部：需严格校验
3. setting 应用 <-> Permission 服务：高权限操作，需额外防护

---

## 数据流安全分析

### 敏感数据流

| 数据 | 流向 | 安全措施 |
|------|------|----------|
| 照片/视频 | Camera -> 文件系统 | 存储在应用私有目录，需权限访问 |
| WiFi 密码 | UI -> wpa_supplicant | 使用 `sprintf_s`，但需验证转义 |
| 权限状态 | Permission 服务 -> UI | 通过系统服务获取，可信 |
| 应用包名 | launcher/setting -> BundleManager | 系统服务验证 |

---

## 修复优先级建议

| 优先级 | 风险 | 建议措施 |
|--------|------|----------|
| **高** | WiFi 命令注入 | 输入校验 + 参数化 |
| **高** | 权限管理安全 | 调用者身份验证 |
| **中** | 路径遍历 | 路径规范化 + 校验 |
| **中** | 日志泄露 | 发布版本禁用调试日志 |
| **低** | 缓冲区溢出 | 已缓解（使用 `_s` 函数），持续监控 |

---

## 检查范围与局限性

### 已检查范围
- 所有 `.cpp` 和 `.c` 源文件（排除 test/ 目录）
- 5 个 BUILD.gn 构建文件
- 配置文件（config.json, bundle.json）
- 权限声明和 API 调用

### 未检查范围（局限性）
- 底层多媒体服务（camera_lite, recorder_lite 等）内部实现
- WiFi 协议栈（wpa_supplicant）安全性
- UI 框架（ui_lite）安全性
- 系统服务（samgr_lite, permission_lite）内部实现
- 动态运行时行为（需实际设备测试）
- 网络通信安全性
- 加密实现安全性

### 建议的后续工作
1. 动态分析：在实际设备上运行并监控行为
2. 渗透测试：模拟攻击场景
3. 依赖组件审计：检查底层服务安全性
4. 代码签名验证：确保 HAP 包完整性

---

## 相关链接

- [对外接口](./External_API.md) - 接口安全考虑
- [内部接口](./Internal_API.md) - 内部 API 安全
- [GN 构建系统](./Build_System.md) - 构建安全
