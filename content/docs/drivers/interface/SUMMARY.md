# Wiki 导航

## 快速开始路线

### 🎓 新人学习路线 (推荐)
适合刚接触 OpenHarmony 驱动开发的工程师：

1. **[项目概览](./01_Overview.md)** - 5分钟了解仓库定位
2. **[目录结构与模块地图](./02_Directory_Structure.md)** - 15分钟找到目标模块
3. **[HDI 接口定义规范](./03_HDI_IDL_Specification.md)** - 30分钟掌握 IDL 语法
4. **[构建系统与编译产物](./04_Build_System.md)** - 了解 GN 构建
5. **[内部架构与实现原理](./06_Inner_Architecture.md)** - 深入理解机制

### 🔒 安全研究路线 (推荐)
适合进行安全审计和漏洞挖掘的研究员：

1. **[攻击面分析](./05_AttackSurface.md)** - 快速识别所有 IPC 入口
2. **[安全机制与风险评估](./07_Security.md)** - 深度安全风险分析
3. **[构建系统安全配置](./04_Build_System.md)** - 编译选项安全分析

---

## 文档列表

### 核心文档 (8篇)

| 文档 | 说明 | 目标读者 | 预计阅读时间 |
|-----|------|---------|-------------|
| [README](./README.md) | 文档覆盖范围与更新方式 | 所有人 | 5分钟 |
| [01_Overview](./01_Overview.md) | 项目定位、核心能力、运行环境 | 新人 | 15分钟 |
| [02_Directory_Structure](./02_Directory_Structure.md) | 模块分类与职责说明 | 开发者 | 20分钟 |
| [03_HDI_IDL_Specification](./03_HDI_IDL_Specification.md) | IDL 语法、版本管理、接口定义 | 开发者 | 30分钟 |
| [04_Build_System](./04_Build_System.md) | GN 构建配置、模板参数、产物清单 | 开发者 | 25分钟 |
| [05_AttackSurface](./05_AttackSurface.md) | 完整攻击面清单与数据流分析 | 安全研究员 | 40分钟 |
| [06_Inner_Architecture](./06_Inner_Architecture.md) | 模块依赖、线程模型、生命周期 | 架构师 | 30分钟 |
| [07_Security](./07_Security.md) | 安全架构、风险评估、修复建议 | 安全工程师 | 45分钟 |

### 附录文档

| 文档 | 说明 |
|-----|------|
| [appendix/Callgraphs](./appendix/Callgraphs.md) | 关键调用链（入口→核心逻辑） |
| [appendix/Config_Flags](./appendix/Config_Flags.md) | 关键宏与 feature flags |

---

## 模块快速索引

### 基础 I/O
- [Audio](./02_Directory_Structure.md#audio) - 音频接口 (v1_0 - v6_0)
- [Display](./02_Directory_Structure.md#display) - 显示接口 (buffer/composer)
- [Input](./02_Directory_Structure.md#input) - 输入设备接口 (passthrough 模式)
- [USB](./02_Directory_Structure.md#usb) - USB 接口 (host/gadget/ddk)
- [WLAN](./02_Directory_Structure.md#wlan) - 无线局域网接口

### 安全认证 (⚠️ 安全敏感)
- [HUKS](./05_AttackSurface.md#huks-密钥管理) - 硬件密钥管理 (ESL3, passthrough 模式) ⚠️
- [User Auth](./05_AttackSurface.md#user-auth-用户认证框架) - 用户认证框架 ⚠️
- [Fingerprint Auth](./05_AttackSurface.md#fingerprint-auth-指纹认证) - 指纹认证 ⚠️
- [Face Auth](./05_AttackSurface.md#face-auth-人脸认证) - 人脸认证 ⚠️
- [Pin Auth](./05_AttackSurface.md#pin-auth-pin-认证) - PIN 认证 ⚠️
- [Secure Element](./05_AttackSurface.md#secure-element-安全元件) - 安全元件访问 ⚠️

### 感知类
- [Camera](./02_Directory_Structure.md#camera) - 相机接口 (v1_0 - v1_5)
- [Sensor](./02_Directory_Structure.md#sensor) - 传感器接口 (v1_0 - v3_1)
- [Location](./02_Directory_Structure.md#location) - 定位接口 (GNSS/AGNSS/Geofence)

### 连接类
- [Bluetooth](./02_Directory_Structure.md#bluetooth) - 蓝牙接口
- [NFC](./02_Directory_Structure.md#nfc) - 近场通信接口
- [RIL](./02_Directory_Structure.md#ril) - 无线接口层

### 多媒体
- [Codec](./02_Directory_Structure.md#codec) - 编解码接口
- [DRM](./02_Directory_Structure.md#drm) - 数字版权管理
- [Vibrator](./02_Directory_Structure.md#vibrator) - 振动器接口

---

## 版本兼容性

| 系统类型 | 语言支持 | 生成模式 | 示例设备 | 安全等级 |
|---------|---------|---------|---------|---------|
| **Standard** | C/C++ | IPC / Passthrough | 手机、平板、车机 | 高 |
| **Small** | C/C++ | Passthrough | 手表、电视、音箱 | 中 |
| **Mini** | C | Low (直通) | IoT 设备 | 低 |

### 模式安全对比

| 模式 | 隔离性 | 性能 | 适用场景 | 风险等级 |
|-----|-------|------|---------|---------|
| **IPC** | 进程隔离 | 较低 | Standard 系统 | 低 |
| **Passthrough** | 无隔离 | 高 | Small 系统 / 安全模块 | **高** |
| **Low** | 内核态 | 最高 | Mini 系统 | **严重** |

---

## 关键安全发现速查

### 🔴 严重风险 (P0)

| 风险 | 位置 | 影响 |
|-----|------|------|
| HUKS Passthrough 模式 | `huks/v1_1/BUILD.gn:25` | 密钥完全暴露 |
| 明文密钥导入 | `huks/v1_1/IHuks.idl:45` | 密钥泄露 |
| Root Secret 返回 | `user_auth/v4_0/UserAuthTypes.idl:52` | 文件可被解密 |
| APDU 任意传输 | `secure_element/v1_0/ISecureElementInterface.idl:35` | SE 被绕过 |

### 🟡 高危风险 (P1)

| 风险 | 位置 | 影响 |
|-----|------|------|
| 生物认证缺少 PAC | `user_auth/v4_1/BUILD.gn` | 代码执行 |
| PIN 明文传输 | `pin_auth/v3_0/IAllInOneExecutor.idl:31` | PIN 泄露 |

详见 [07_Security.md](./07_Security.md)

---

## 术语速查

| 术语 | 全称 | 说明 |
|-----|------|------|
| HDI | Hardware Device Interface | 硬件设备接口 |
| IDL | Interface Definition Language | 接口定义语言 |
| HDF | Hardware Driver Foundation | 硬件驱动框架 |
| IPC | Inter-Process Communication | 进程间通信 |
| TEE | Trusted Execution Environment | 可信执行环境 |
| SE | Secure Element | 安全元件 |
| ESL | Executor Security Level | 执行器安全等级 (0-3) |
| HUKS | Universal KeyStore | 通用密钥库 |
| PAC | Pointer Authentication Code | ARM 指针认证 |
| APDU | Application Protocol Data Unit | 智能卡协议数据 |

---

*最后更新: 2026-02-07*
