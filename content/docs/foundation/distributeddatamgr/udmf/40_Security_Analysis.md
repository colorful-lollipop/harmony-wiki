# 安全分析

## 分析概述

本章节对 UDMF（统一数据管理框架）进行系统性的安全分析，涵盖攻击面识别、信任边界划分、数据流分析和安全机制评估。分析基于源码证据，识别潜在安全风险并提供修复建议。

UDMF 作为 OpenHarmony 分布式数据管理的核心组件，负责跨应用、跨设备的敏感数据流转。安全分析聚焦于以下方面：接口层输入验证、IPC 通信安全、权限控制机制、数据存储安全和权限管理。

## 攻击面识别

### 接口层攻击面

UDMF 提供三层接口，每层都存在潜在的攻击面：

**N-API 接口层（`interfaces/jskits/`）**
- 模块注册入口（`napi_module_register`）
- 导出方法（`napi_define_properties`）
- 异步回调（`napi_threadsafe_function`）
- Promise 返回值处理

**NDK 接口层（`interfaces/ndk/`）**
- C API 入口函数（`OH_Udmf_*`、`OH_Utd_*`）
- 回调函数注册（`OH_UdmfRecordProvider_SetData`）
- 字符串参数传递
- 缓冲区操作

**InnerKit 接口层（`interfaces/innerkits/`）**
- 单例获取（`GetInstance`）
- 参数校验
- 返回值状态码

### IPC 攻击面

服务层 IPC 通信（`framework/innerkitsimpl/service/`）：
- 接口码枚举（`UdmfServiceInterfaceCode`）
- 参数序列化（`MessageParcel`）
- 代理调用（`UdmfServiceProxy::SendRequest`）
- 死亡通知（`ServiceDeathRecipient`）

### 存储攻击面

数据持久化层：
- KV 数据库操作
- 临时文件处理
- 序列化/反序列化
- 配置加载（`uniform_data_types.json`）

### 配置文件攻击面

UDMF 配置文件（`conf/`）：
- `uniform_data_types.json` 解析
- UTD 类型定义
- 路径遍历风险

## 信任边界

### 进程边界

```
┌─────────────────────────────────────────────────────────────┐
│                      应用进程                              │
│  ┌─────────────────────────────────────────────────┐   │
│  │  N-API / NDK / InnerKit 接口层                    │   │
│  └─────────────────────────────────────────────────┘   │
│                          ↓ IPC                          │
├─────────────────────────────────────────────────────────────┤
│                    系统服务进程                            │
│  ┌─────────────────────────────────────────────────┐   │
│  │  UdmfService / UtdService                       │   │
│  └─────────────────────────────────────────────────┘   │
│                          ↓ IPC                          │
├─────────────────────────────────────────────────────────────┤
│                    分布式设备                              │
│  ┌─────────────────────────────────────────────────┐   │
│  │  远端 UDMF 服务                                │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 信任模型

**应用进程可信**：
- 已安装的应用包
- 签名验证通过
- 权限声明完整

**系统服务可信**：
- 系统进程
- SAMGR 注册
- 权限策略执行

**远端设备可信**：
- 设备绑定关系已建立
- 分布式安全通道已建立
- 凭证验证通过

### 边界规则

| 边界 | 信任级别 | 检查机制 |
|------|----------|----------|
| 应用 → UDMF 服务 | 中 | AccessToken 校验 |
| UDMF 服务 → 数据库 | 高 | 进程隔离 |
| 远端 → 本地服务 | 高 | 设备认证 + 加密传输 |
| 配置 → 服务 | 高 | 文件完整性校验 |

## 数据流分析

### 数据创建流程

```
用户数据 → UnifiedData → Service → KV Store
            ↓
      权限检查
            ↓
      序列化（TLV）
            ↓
      IPC 传输
            ↓
      持久化
```

**安全检查点**：
1. 输入参数校验（类型、大小、编码）
2. AccessToken 权限检查
3. 共享选项验证
4. 序列化安全检查

### 数据读取流程

```
请求 → KV Store → 反序列化 → Service → IPC → Client
                    ↓
              权限校验
                    ↓
              返回数据
```

**安全检查点**：
1. 请求者身份验证
2. 数据所有权检查
3. 权限范围验证
4. 响应数据脱敏

### 数据同步流程

```
本地数据 → 加密打包 → 传输层 → 远端验证 → 解密存储
                        ↓
                  设备绑定检查
                        ↓
                  签名验证
```

**安全检查点**：
1. 设备凭证验证
2. 数据完整性校验
3. 冲突策略执行

## 安全机制评估

### 访问控制机制

**AccessToken 校验**
- 基于 `access_token` 的权限管理
- 应用权限声明验证
- 敏感操作鉴权

**代码证据**：
```cpp
// frameworks/innerkitsimpl/client/udmf_client.cpp
Status UdmfClient::AddPrivilege(const QueryOption &query,
                                Privilege &privilege)
{
    // 校验调用者权限
    if (!CheckCallingPermission()) {
        return E_NO_PERMISSION;
    }
    // ...
}
```

### 共享控制机制

**ShareOption 枚举**
```cpp
typedef enum Udmf_ShareOption {
    SHARE_OPTIONS_INVALID = 0,
    SHARE_OPTIONS_IN_APP,        // 仅应用内
    SHARE_OPTIONS_CROSS_APP,    // 跨应用
} Udmf_ShareOption;
```

### 符号可见性控制

**API_EXPORT 宏**
```cpp
// visibility.h
#define API_EXPORT __attribute__((visibility ("default")))

// BUILD.gn
cflags_cc = [ "-fvisibility=hidden" ]
```

### 安全编译选项

根据 `BUILD.gn` 配置，所有目标启用以下安全编译选项：

| 选项 | 配置值 | 说明 |
|------|--------|------|
| CFI | `cfi = true` | 控制流完整性 |
| CFI Cross-DSO | `cfi_cross_dso = true` | 跨库 CFI 检查 |
| 边界 Sanitizer | `boundary_sanitize = true` | 边界检查 |
| UBSan | `ubsan = true` | 未定义行为检查 |
| PAC | `branch_protector_ret = "pac_ret"` | 指针认证 |

## 威胁模型

### 资产定义

| 资产 | 价值 | 敏感度 |
|------|------|--------|
| 用户隐私数据 | 高 | 极高 |
| 应用数据 | 中高 | 高 |
| 数据密钥 | 高 | 极高 |
| 配置信息 | 中 | 中 |
| 服务状态 | 中 | 中 |

### 威胁场景

**T1：越权数据访问**
- 攻击者：恶意应用
- 途径：伪造请求参数
- 目标：读取其他应用数据
- 概率：低
- 影响：极高

**T2：数据篡改**
- 攻击者：中间人
- 途径：IPC 通信
- 目标：修改传输数据
- 概率：低
- 影响：高

**T3：拒绝服务**
- 攻击者：恶意应用
- 途径：资源耗尽
- 目标：服务崩溃
- 概率：中
- 影响：中

**T4：路径遍历**
- 攻击者：恶意应用
- 途径：文件 URI 操作
- 目标：访问任意文件
- 概率：低
- 影响：高

**T5：整数溢出**
- 攻击者：恶意应用
- 途径：大小参数操纵
- 目标：内存破坏
- 概率：低
- 影响：极高

## 风险评估矩阵

| 威胁 | 可利用性 | 影响力 | 风险等级 | 对策 |
|------|----------|--------|----------|------|
| 越权访问 | 低 | 极高 | 高 | 权限校验强化 |
| 数据篡改 | 低 | 高 | 中 | IPC 加密 |
| 拒绝服务 | 中 | 中 | 中 | 资源限制 |
| 路径遍历 | 低 | 高 | 中 | 路径标准化 |
| 整数溢出 | 低 | 极高 | 高 | 边界检查 |

## 安全建议

### 短期建议

1. **强化参数校验**
   - 所有接口增加输入验证
   - 字符串编码检查
   - 数值范围检查

2. **增加安全日志**
   - 敏感操作审计
   - 失败请求记录
   - 异常行为检测

### 中期建议

3. **实施最小权限**
   - 细粒度权限控制
   - 运行时权限检查
   - 权限降级机制

4. **增强 IPC 安全**
   - 消息签名验证
   - 加密传输
   - 重放保护

### 长期建议

5. **安全架构演进**
   - 零信任模型
   - 硬件安全支持
   - 形式化验证

## 相关文档

- [00_Overview.md](./00_Overview.md)：项目概述
- [01_Directory_Structure.md](./01_Directory_Structure.md)：目录结构
- [11_NDK_Reference.md](./11_NDK_Reference.md)：NDK 接口
- [41_Security_Risks.md](./41_Security_Risks.md)：安全风险清单
- [31_Build_Artifacts.md](./31_Build_Artifacts.md)：编译产物
- [30_GN_Build_Targets.md](./30_GN_Build_Targets.md)：构建配置
