# 常见问题

## 目的

本文档收集 `system_resources` 模块的常见构建、运行和调试问题，帮助开发者快速定位和解决问题。

## 适用范围

- **模块**: `/base/global/system_resources`
- **问题分类**: 构建问题、配置问题、资源问题

## 构建问题

### Q1: 构建时找不到字体文件

**问题描述**:
```
ERROR: source file "fonts/HarmonyOS_Sans.ttf" not found
```

**可能原因**:
1. 字体文件未正确放置在 `fonts/` 目录
2. `systemres.gni` 中的 `font_path` 配置错误

**排查步骤**:

1. 检查字体文件是否存在:
```bash
ls -la fonts/HarmonyOS_Sans.ttf
```

2. 检查 systemres.gni 配置:
```bash
grep -A5 "HarmonyOS_Sans" systemres.gni
```

3. 检查 font_path 是否正确:
```gn
# systemres.gni 中的配置
{
  font_name = "HarmonyOS_Sans"
  font_path = "fonts/HarmonyOS_Sans.ttf"  # 路径必须正确
  support_devices = ["default", "watch"]
}
```

**解决方案**:
- 确保字体文件存在于 `fonts/` 目录
- 确保 `font_path` 与实际文件路径一致

**证据**: `systemres.gni:29-36`

---

### Q2: 构建产物未安装到预期目录

**问题描述**:
构建成功但字体文件/Hap 包未安装到系统分区

**可能原因**:
1. `module_install_dir` 配置错误
2. 构建目标产品不匹配

**排查步骤**:

1. 检查 module_install_dir 配置:
```bash
# BUILD.gn 中检查
ohos_prebuilt_etc("HarmonyOS_Sans") {
  module_install_dir = "fonts"  # 必须为 "fonts"
}
```

2. 检查产品配置:
```bash
# 检查 system_resources_font_feature_product 值
gn args out/xxx --list | grep system_resources_font_feature_product
```

**解决方案**:
- 确保 `module_install_dir = "fonts"`
- 确保产品配置包含目标设备类型

**证据**: `BUILD.gn:33`

---

### Q3: Hap 签名失败

**问题描述**:
```
ERROR: Failed to sign hap: certificate not found
```

**可能原因**:
1. 签名证书文件不存在
2. 证书路径配置错误
3. 密钥别名错误

**排查步骤**:

1. 检查证书文件是否存在:
```bash
ls -la vendor/tools/hap_sign_conf/global/system_resources/SystemResources.p7b
```

2. 检查证书路径配置:
```bash
# systemres.gni 中检查
certificate_profile_path =
    "//vendor/tools/hap_sign_conf/global/system_resources/SystemResources.p7b"
```

3. 检查密钥配置:
```bash
# BUILD.gn 中检查
key_alias = "OHSystemResources"
private_key_path = "OHSystemResources"
```

**解决方案**:
- 确保证书文件存在于指定路径
- 检查 vendor 目录配置完整性

**证据**: `BUILD.gn:33-41`, `systemres.gni:19-20`

---

## 配置问题

### Q4: 如何新增系统字体

**问题描述**: 需要添加新的字体文件到系统资源

**解决方案**:

1. 将字体文件放入 `fonts/` 目录:
```bash
cp NewFont.ttf fonts/
```

2. 在 `systemres.gni` 的 `sys_fonts_list` 中添加配置:
```gn
{
  font_name = "NewFont"
  font_path = "fonts/NewFont.ttf"
  support_devices = ["default"]
  alias_name = ""
},
```

3. 提交并构建:
```bash
# 重新生成构建文件
gn gen out/xxx
ninja -C out/xxx systemres_hap
```

**证据**: `systemres.gni:28-36`

---

### Q5: 如何添加新的权限定义

**问题描述**: 需要在系统资源中添加新的权限元数据

**解决方案**:

在 `systemres/main/module.json` 的 `definePermissions` 数组中添加:

```json
{
  "name": "ohos.permission.NEW_PERMISSION",
  "grantMode": "system_grant",
  "availableLevel": "system_basic",
  "since": X,
  "deprecated": "",
  "provisionEnable": true,
  "distributedSceneEnable": false
}
```

**注意事项**:
- `name`: 权限完整名称，必须唯一
- `grantMode`: "system_grant" 或 "user_grant"
- `availableLevel`: "normal" / "system_basic" / "system_core"
- `since`: API 起始版本号

**证据**: `module.json:17-42`

---

### Q6: 如何适配新的设备类型

**问题描述**: 需要让字体/资源支持新的设备类型

**解决方案**:

1. 修改 `systemres.gni` 中字体的 `support_devices`:
```gn
{
  font_name = "HarmonyOS_Sans"
  font_path = "fonts/HarmonyOS_Sans.ttf"
  support_devices = ["default", "watch", "new_device"]
}
```

2. 添加设备特定资源配置:
```bash
mkdir -p systemres/main/resources/new_device/element/
```

3. 在 `module.json` 中添加设备类型:
```json
"deviceTypes": ["default", "new_device"]
```

**证据**: `module.json:7-14`, `systemres.gni:32-35`

---

## 资源问题

### Q7: 字体不显示

**问题描述**: 系统字体未正确渲染

**排查步骤**:

1. 检查字体是否正确安装:
```bash
# 在设备上检查
ls -la /system/fonts/HarmonyOS_Sans.ttf
```

2. 检查资源管理服务日志:
```bash
hilog | grep -i "font"
```

3. 检查字体文件完整性:
```bash
file fonts/HarmonyOS_Sans.ttf
```

**解决方案**:
- 重新刷写系统镜像
- 检查字体文件是否损坏

**相关模块**: global_resmgr_standard

---

### Q8: 权限不生效

**问题描述**: 在 module.json 中定义的权限在系统中不可用

**排查步骤**:

1. 检查 SystemResources.hap 是否正确安装:
```bash
ls -la /system/app/ohos.global.systemres/
```

2. 检查权限定义语法:
```bash
# 验证 JSON 语法
python3 -m json.tool module.json > /dev/null
```

3. 检查权限级别是否匹配:
- normal 权限所有应用可用
- system_basic 需 system_basic 级别应用
- system_core 需 system_core 级别应用

**解决方案**:
- 重新构建并部署 SystemResources.hap
- 检查应用是否有权限声明

**证据**: `module.json:17-1301+`

---

### Q9: 多语言资源未加载

**问题描述**: 应用未显示正确的本地化字符串

**排查步骤**:

1. 检查语言资源是否存在:
```bash
ls systemres/main/resources/zh_CN/element/
```

2. 检查资源 ID 是否正确:
```bash
# 在代码中使用正确的资源 ID
```

3. 检查系统语言设置:
```bash
# 在设备上检查
getprop ro.product.locale
```

**解决方案**:
- 确保对应语言的资源目录存在
- 确保资源 ID 正确

**证据**: `systemres/main/resources/` 目录结构

---

## 调试技巧

### 日志查看

| 模块 | 日志命令 |
|------|----------|
| 资源管理 | `hilog | grep -i "res"` |
| 字体加载 | `hilog | grep -i "font"` |
| 权限服务 | `hilog | grep -i "permission"` |

### 构建调试

| 场景 | 命令 |
|------|------|
| 查看 GN 变量 | `gn args out/xxx --list` |
| 查看构建依赖 | `ninja -C out/xxx -t deps systemres_hap` |
| 详细构建日志 | `ninja -C out/xxx -v` |

### 常见错误码

| 错误码 | 含义 | 排查方向 |
|--------|------|----------|
| - | 字体文件不存在 | 检查 fonts/ 目录 |
| - | JSON 解析失败 | 检查配置文件语法 |
| - | 证书无效 | 检查签名配置 |

## 相关文档

| 文档 | 链接 |
|------|------|
| 全局导航 | [SUMMARY.md](SUMMARY.md) |
| 项目概览 | [00_Overview.md](00_Overview.md) |
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 构建系统 | [03_Build_System.md](03_Build_System.md) |
| 系统资源 | [04_Resources.md](04_Resources.md) |
| 安全评审 | [05_Security.md](05_Security.md) |

## 贡献问题

如果您遇到未在本文档中列出问题，请:

1. 在 [Gitee Issues](https://gitee.com/openharmony/resources/issues) 提出
2. 包含:
   - 问题描述
   - 复现步骤
   - 日志输出
   - 环境信息

## 更新日志

| 日期 | 变更 | 负责人 |
|------|------|--------|
| 2026-02-06 | 初始版本 | Wiki Generator |
