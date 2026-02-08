# 安全风险评审

## 评审范围

本评审覆盖 `applications/standard/hap` 仓库的以下方面：

| 类别 | 覆盖范围 | 说明 |
|------|----------|------|
| HAP 包 | ✅ | 预构建应用安装包 |
| 构建配置 | ✅ | BUILD.gn, build.sh |
| 系统事件 | ✅ | hisysevent 配置 |
| 资源文件 | ✅ | 音频、模板资源 |
| 签名配置 | ✅ | 签名工具与流程 |
| 源码 | ⚠️ 不适用 | 本仓库为预构建归档 |

## 攻击面分析

### 输入源

| 输入类型 | 来源 | 风险等级 |
|----------|------|----------|
| 流水线构建产物 | OpenHarmony CI/CD | 中 |
| Git 仓库代码 | 外部源码仓库 | 中 |
| 签名密钥 | 开发工具 | 高 |
| SDK | 官方分发 | 低 |

### 信任边界

```
┌─────────────────────────────────────────────────────┐
│                    外部不可信区                       │
│  ┌─────────────────────────────────────────────┐   │
│  │  Git 仓库源码                               │   │
│  │  OpenHarmony 流水线                         │   │
│  │  第三方 SDK                                │   │
│  └─────────────────────────────────────────────┘   │
│                      ↑                              │
│              签名验证边界                            │
│                      ↓                              │
│  ┌─────────────────────────────────────────────┐   │
│  │  预构建 HAP (本仓库)                         │   │
│  │  - 需验证签名                               │   │
│  │  - 需校验完整性                             │   │
│  └─────────────────────────────────────────────┘   │
│                      ↑                              │
│              系统执行边界                            │
│                      ↓                              │
│  ┌─────────────────────────────────────────────┐   │
│  │  OpenHarmony 系统 (可信区)                  │   │
│  │  - Bundle Manager                           │   │
│  │  - 权限系统                                  │   │
│  │  - 应用沙箱                                  │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

## 数据流

### 构建数据流

```
源码仓库 ──→ 流水线构建 ──→ HAP 签名 ──→ 归档存储
    │            │            │            │
    ↓            ↓            ↓            ↓
  代码审查    依赖解析     密钥签名     版本管理
```

### 安装数据流

```
HAP 文件 ──→ Bundle Manager ──→ 权限校验 ──→ 应用沙箱
    │             │                │            │
    ↓             ↓                ↓            ↓
  签名验证    配置解析     ACL 检查    资源隔离
```

## 可被利用点

### 1. 签名密钥泄露风险

**证据**: `build.sh:540-541`

```bash
java -jar hap-sign-tool.jar sign-app \
  -keyAlias "openharmony application release" \
  -signAlg "SHA256withECDSA" \
  -keystoreFile "OpenHarmony.p12" \
  -keyPwd "123456" \
  -keystorePwd "123456"
```

**问题描述**:
- 签名密钥硬编码在脚本中
- 密码使用默认占位符
- 密钥文件路径暴露

**触发条件**:
- 构建脚本泄露
- 密钥文件未妥善保管
- CI/CD 环境配置不当

**影响范围**:
- 可签名恶意 HAP
- 伪装成系统应用
- 绕过系统签名校验

**修复建议**:
1. 使用密钥管理服务
2. 环境变量注入密钥
3. CI/CD 密钥保险库
4. 移除硬编码密码

---

### 2. SDK 下载中间人攻击

**证据**: `build.sh:146-148`

```bash
npm config set registry https://repo.huaweicloud.com/repository/npm/
npm config set @ohos:registry https://repo.harmonyos.com/npm/
```

**问题描述**:
- SDK 通过 HTTP/HTTPS 下载
- 未验证下载文件完整性
- 依赖包来源未校验

**触发条件**:
- 网络劫持
- DNS 污染
- 恶意镜像站

**影响范围**:
- 注入恶意代码
- 供应链攻击
- 构建产物被篡改

**修复建议**:
1. 使用 HTTPS 强制
2. 添加校验和验证
3. 使用官方镜像
4. npm 签名验证

---

### 3. HAP 文件完整性无校验

**证据**: `BUILD.gn:16-21`

```gn
ohos_prebuilt_etc("launcher_hap") {
  source = "Launcher.hap"
  module_install_dir = "app/com.ohos.launcher"
  part_name = "prebuilt_hap"
  subsystem_name = "applications"
}
```

**问题描述**:
- GN 预构建目标未包含 hash/checksum
- 无法检测文件篡改
- 依赖外部存储完整性

**触发条件**:
- 存储库被攻陷
- 文件被替换
- 构建缓存污染

**影响范围**:
- 部署恶意 HAP
- 系统应用被篡改
- 用户设备被攻击

**修复建议**:
1. 添加 SHA256 校验
2. 使用 manifest 验证
3. 引入代码签名
4. 区块链存证

---

### 4. hisysevent 配置注入

**证据**: `hisysevent/com.ohos.systemui/hisysevent.yaml:14-21`

```yaml
domain: SYSTEMUI_APP

SYSTEMUI_FAULT:
  __BASE: {type: FAULT, level: CRITICAL, desc: fault log}
  CORE_SYSTEM: {type: STRING, desc: core system}
  TARGET_API: {type: STRING, desc: target api}
```

**问题描述**:
- 系统事件定义可被修改
- YAML 解析可能存在注入点
- 事件字段类型宽松

**触发条件**:
- 配置文件被篡改
- 恶意事件注入
- 日志解析漏洞

**影响范围**:
- 日志注入攻击
- 事件风暴 (DoS)
- 安全监控失效

**修复建议**:
1. YAML schema 验证
2. 字段白名单校验
3. 事件频率限制
4. 完整性签名

---

### 5. 资源文件路径遍历

**证据**: `BUILD.gn:142-152`

```gn
ohos_prebuilt_etc("demo.wav") {
  source = "resources/demo.wav"
  part_name = "prebuilt_hap"
  subsystem_name = "applications"
}
```

**问题描述**:
- 资源路径硬编码
- 无路径验证机制
- 下载模板含执行能力

**触发条件**:
- 资源文件被替换
- 路径注入攻击
- 模板引擎漏洞

**影响范围**:
- 恶意资源加载
- 本地文件读取
- 远程代码执行

**修复建议**:
1. 路径白名单校验
2. 文件类型检查
3. 资源签名验证
4. 沙箱资源隔离

---

### 6. 产品变体条件编译绕过

**证据**: `BUILD.gn:309-346`

```gn
if (defined(product_name) && product_name == "watchos") {
  deps -= [ ":calendarData_hap", ":printspooler_hap", ... ]
} else if (defined(product_name) && product_name == "rk3568") {
  deps += [ "//applications/standard/admin_provisioning:adminprovisioning_hap" ]
}
```

**问题描述**:
- 产品名称可被外部定义
- 条件依赖可被绕过
- admin_provisioning 权限敏感

**触发条件**:
- 恶意 product_name 注入
- 构建参数劫持
- 依赖替换攻击

**影响范围**:
- 高权限功能暴露
- 构建产物不一致
- 安全边界失效

**修复建议**:
1. 产品名称白名单
2. 构建参数签名
3. 依赖完整性校验
4. 安全配置审计

---

### 7. 签名配置模板不安全

**证据**: `build.sh:530-541`

```bash
sed -i "s/\"com.OpenHarmony.app.test\"/\"${bundle_name}\"/g" ${arg_profile}
sed -i "s/\"normal\"/\"${arg_apl}\"/g" ${arg_profile}
java -jar hap-sign-tool.jar sign-profile -keyAlias "..." ...
```

**问题描述**:
- sed 替换存在注入风险
- APL 级别可被修改
- 配置文件生成不安全

**触发条件**:
- bundle_name 注入
- APL 级别越权
- 配置文件覆盖

**影响范围**:
- 权限提升攻击
- 签名绕过
- 应用伪装

**修复建议**:
1. 参数白名单校验
2. 使用模板引擎
3. 配置签名验证
4. 最小权限原则

---

### 8. 构建脚本命令注入

**证据**: `build.sh:474-486`

```bash
# Historical reasons need to be compatible with NODE_HOME path issue
if grep -q "\${NODE_HOME}/bin/node" hvigorw ; then
  if [ ! -x "${NODE_HOME}/bin/node" ];then
    export NODE_HOME=$(dirname ${NODE_HOME})
  fi
else
  ...
fi
./hvigorw clean --no-daemon
```

**问题描述**:
- 动态命令执行
- 路径处理不安全
- 环境变量注入

**触发条件**:
- 恶意 NODE_HOME 设置
- hvigorw 被篡改
- 环境变量注入

**影响范围**:
- 任意命令执行
- 构建环境劫持
- 恶意代码注入

**修复建议**:
1. 路径白名单校验
2. 环境变量过滤
3. 使用绝对路径
4. 命令参数化

---

### 9. 临时文件清理不彻底

**证据**: `build.sh:554`

```bash
rm -rf ${arg_project}/sign_helper
```

**问题描述**:
- 敏感临时文件可能残留
- 密钥文件可能被泄露
- 构建产物清理不完整

**触发条件**:
- 构建中断
- 权限不足
- 磁盘空间不足

**影响范围**:
- 密钥泄露
- 敏感信息残留
- 逆向工程风险

**修复建议**:
1. 使用安全删除工具
2. 构建后清理验证
3. 临时目录隔离
4. 清理失败告警

---

### 10. Git 仓库地址验证缺失

**证据**: `build.sh:301-321`

```bash
if [ "${arg_url}" != "" ]; then
  if [ "${arg_branch}" == "" ]; then
    echo "branch is not null"
    exit 1
  fi
  project_name=${arg_url##*/}
  project_name=${project_name%%.git*}
  ...
  git clone -b ${arg_branch} ${arg_url} ${project_name}
fi
```

**问题描述**:
- Git URL 未验证
- 分支名称未校验
- 克隆行为无审计

**触发条件**:
- 恶意 Git URL
- 分支名注入
- 中间人攻击

**影响范围**:
- 恶意代码拉取
- 构建污染
- 供应链攻击

**修复建议**:
1. URL 白名单校验
2. 分支名称验证
3. Git 签名验证
4. 克隆审计日志

## 安全改进建议

### 短期 (1-2 周)

1. **移除硬编码密钥**: 使用环境变量或密钥管理服务
2. **添加完整性校验**: SHA256 hash 验证
3. **启用 HTTPS 强制**: 所有下载使用 HTTPS
4. **参数白名单**: 校验所有外部输入

### 中期 (1-3 月)

1. **签名密钥轮换**: 定期更换签名密钥
2. **构建审计**: 记录所有构建操作
3. **依赖扫描**: 使用软件成分分析 (SCA)
4. **供应链安全**: 实施 SLSA 框架

### 长期 (3-6 月)

1. **零信任构建**: 实施构建签名
2. **自动化安全测试**: 集成安全扫描
3. **安全事件响应**: 建立安全事件流程
4. **合规性审计**: 定期安全评估

## 检查范围局限性

| 检查项 | 状态 | 说明 |
|--------|------|------|
| HAP 内部代码 | ❌ 未检查 | 预构建包，无法审查 |
| 运行时行为 | ❌ 未检查 | 需运行时分析 |
| 网络通信 | ❌ 未检查 | 需流量分析 |
| 权限配置 | ⚠️ 部分检查 | 需查看 module.json5 |
| 第三方依赖 | ❌ 未检查 | 需独立分析 |

## 相关链接

- [07_Security_Review.md](./07_Security_Review.md) - 本文档
- [05_Build_System.md](./05_Build_System.md) - 构建系统
- [06_Artifacts.md](./06_Artifacts.md) - 编译产物
- [OpenHarmony 安全指南](https://gitee.com/openharmony/security) - 官方安全文档
