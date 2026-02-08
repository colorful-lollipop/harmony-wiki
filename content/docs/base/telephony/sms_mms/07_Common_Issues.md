# 常见问题 (FAQ)

## 目的

本文档汇总短彩信模块开发、调试、测试过程中的常见问题及解决方案，帮助开发者快速定位和解决问题。

## 适用范围

本文档覆盖：
- 构建和编译问题
- 运行和调试问题
- API 使用问题
- 权限和配置问题
- 网络和协议问题

## 构建问题

### Q1: 编译时提示找不到 `glib` 依赖

**问题描述**:
```
ERROR: //base/telephony/sms_mms:tel_sms_mms missing dependency on glib
```

**原因**:
短彩信模块依赖 glib 库，需要确保系统环境中已安装 glib。

**解决方案**:
1. 检查系统是否安装了 glib:
```bash
pkg-config --exists glib-2.0 && echo "glib found" || echo "glib not found"
```

2. 如果缺失，安装 glib:
```bash
# Ubuntu/Debian
sudo apt-get install libglib2.0-dev

# openSUSE
sudo zypper install glib2-devel
```

**证据**: `README_zh.md:58` 明确说明依赖 glib。

---

### Q2: MMS 功能未编译进产物

**问题描述**:
编译后的 `libtel_sms_mms.z.so` 不包含 MMS 相关符号。

**原因**:
MMS 是可选特性，默认启用，但可能被配置关闭。

**解决方案**:
1. 检查 `smsmms.gni` 配置:
```gn
# smsmms.gni:17
sms_mms_feature_support_mms = true  # 确保为 true
```

2. 检查编译产物中是否包含 MMS 源文件:
```bash
# 检查编译日志中是否包含 MMS 源文件
ninja -C out/telephony sms_mms 2>&1 | grep -i mms
```

3. 重新编译:
```bash
# 清理并重新编译
rm -rf out/telephony/sms_mms
ninja -C out/telephony tel_sms_mms
```

**证据**: `BUILD.gn:167-179` 根据 `sms_mms_feature_support_mms` 条件编译 MMS 代码。

---

### Q3: 编译警告：CFI 相关警告

**问题描述**:
```
warning: Control Flow Integrity is not enabled for this target
```

**原因**:
某些第三方库或特定配置下 CFI (Control Flow Integrity) 可能未启用。

**解决方案**:
1. 检查 `BUILD.gn` 中的 sanitize 配置:
```gn
# BUILD.gn:37-41
sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
}
```

2. 确保使用标准系统构建配置，CFI 警告通常不影响功能。

**证据**: `BUILD.gn:37-41` 定义了 CFI 配置。

## 运行问题

### Q4: SA 4008 未启动或崩溃

**问题描述**:
应用调用短信 API 时返回错误，检查日志发现 SA 4008 未运行。

**排查步骤**:

1. 检查 SA 是否已注册:
```bash
# 查看 SA 列表
hdc shell "ls -la /system/profile/ | grep 4008"
```

2. 检查 telephony 进程是否运行:
```bash
hdc shell "ps -ef | grep telephony"
```

3. 查看 SA 启动日志:
```bash
hdc shell "hilog | grep -i 'sms\|4008'"
```

**常见原因及解决方案**:

| 原因 | 现象 | 解决方案 |
|------|------|----------|
| core_service (SA 4010) 未启动 | 日志显示 "depend SA 4010 timeout" | 确保 core_service 已启动 |
| 权限问题 | 日志显示 "permission denied" | 检查进程权限配置 |
| 库文件缺失 | 日志显示 "cannot load library" | 确保 libtel_sms_mms.z.so 存在于 /system/lib64/ |

**证据**: `sa_profile/4008.json` 显示依赖 SA 4010。

---

### Q5: 短信发送失败，返回 PERMISSION_ERR (201)

**问题描述**:
调用 `sendMessage` 返回错误码 201。

**原因**:
应用缺少 `ohos.permission.SEND_MESSAGES` 权限。

**解决方案**:
1. 在 `module.json5` 中声明权限:
```json
{
    "module": {
        "requestPermissions": [
            {
                "name": "ohos.permission.SEND_MESSAGES",
                "reason": "$string:send_sms_permission_reason"
            }
        ]
    }
}
```

2. 对于系统应用，还需要在系统权限配置中授权。

**证据**: `services/sms/sms_service.cpp:339` 检查 `SEND_MESSAGES` 权限。

---

### Q6: MMS API 返回 ILLEGAL_USE_OF_SYSTEM_API (202)

**问题描述**:
调用 `encodeMms` 或 `sendMms` 返回错误码 202。

**原因**:
MMS API 仅限系统应用使用。

**解决方案**:
1. 确保应用是系统应用（签名包含系统证书）
2. 检查调用者身份:
```cpp
// 服务端的检查逻辑
if (!TelephonyPermission::CheckCallerIsSystemApp()) {
    return TELEPHONY_ERR_ILLEGAL_USE_OF_SYSTEM_API;
}
```

3. 对于非系统应用，无法直接调用 MMS API。

**证据**: `frameworks/js/napi/src/napi_mms.cpp:305` 检查系统应用。

## API 使用问题

### Q7: `sendMessage` 回调不执行

**问题描述**:
调用 `sendMessage` 后，`sendCallback` 没有被调用。

**排查步骤**:

1. 检查参数是否正确:
```javascript
let msg = {
    slotId: 0,                    // 必须是有效卡槽 ID (0 或 1)
    destinationHost: '10086',     // 目标地址
    content: 'test',              // 内容
    sendCallback: (err, data) => {  // 回调必须提供
        console.log('result:', data.result);
    }
};
sms.sendMessage(msg);
```

2. 检查 SIM 卡状态:
```javascript
// 确保 SIM 卡已插入并可用
import sim from '@ohos.telephony.sim';
let state = await sim.getSimState(0);
console.log('Sim state:', state);  // 应为 SIM_STATE_READY
```

3. 检查网络状态:
```javascript
import radio from '@ohos.telephony.radio';
let state = await radio.getNetworkState(0);
console.log('Network state:', state);
```

4. 查看日志:
```bash
hdc shell "hilog | grep -E 'SmsSendManager|GsmSmsSender|sendMessage'"
```

**证据**: `services/sms/sms_send_manager.cpp:109` 检查地址和内容非空。

---

### Q8: `createMessage` 解析 PDU 失败

**问题描述**:
调用 `createMessage` 返回错误，无法解析 PDU。

**常见原因**:

1. PDU 格式不正确:
```javascript
// 错误的 PDU 格式
let pdu = '0011000B91...';  // 十六进制字符串，不能直接传入

// 正确的 PDU 格式
let pdu = [0x00, 0x11, 0x00, 0x0B, 0x91, ...];  // 数字数组
```

2. PDU 长度超过限制:
```javascript
// PDU 最大长度 255 字节
if (pdu.length > 255) {
    console.error('PDU too long');
}
```

3. 规范参数错误:
```javascript
// 规范必须是 "3gpp" 或 "3gpp2"
sms.createMessage(pdu, '3gpp', callback);  // ✓ 正确
sms.createMessage(pdu, 'GSM', callback);   // ✗ 错误
```

**证据**: `services/sms/gsm/gsm_sms_message.cpp:415` 检查 PDU 长度。

---

### Q9: 长短信分段后接收端无法重组

**问题描述**:
发送长短信（超过 160 字符），接收端显示为多条独立短信。

**原因分析**:

1. **发送端**: 自动分段，添加 UDH (User Data Header)
2. **接收端**: 需要识别 UDH 并重组

**排查步骤**:

1. 检查发送端是否正确分段:
```bash
hilog | grep -i 'split'
# 应看到 "split message into N segments"
```

2. 检查接收端是否识别为分段短信:
```bash
hilog | grep -i 'concat\|combine'
# 应看到 "CombineMessagePart" 相关日志
```

3. 确保接收端支持 UDH 解析:
```cpp
// 检查 SmsBaseMessage::GetIsConcat() 返回值
bool isConcat = smsMessage->GetIsConcat();
```

**证据**: `services/sms/sms_receive_handler.cpp:CombineMessagePart()` 实现重组逻辑。

## 权限和配置问题

### Q10: 如何设置短信中心号码 (SMSC)

**问题描述**:
需要手动设置 SMSC 地址。

**解决方案**:

1. 获取当前 SMSC:
```javascript
import sms from '@ohos.telephony.sms';

// 需要权限: ohos.permission.GET_TELEPHONY_STATE
sms.getSmscAddr(0).then((smsc) => {
    console.log('Current SMSC:', smsc);
}).catch((err) => {
    console.error('Failed to get SMSC:', err);
});
```

2. 设置 SMSC:
```javascript
// 需要权限: ohos.permission.SET_TELEPHONY_STATE
sms.setSmscAddr(0, '+8613012345678').then(() => {
    console.log('SMSC set successfully');
}).catch((err) => {
    console.error('Failed to set SMSC:', err);
});
```

**注意**:
- SMSC 地址格式应为国际格式，如 `+8613012345678`
- 长度限制: 最大 21 位数字
- 某些运营商可能禁止修改 SMSC

**证据**: `services/sms/sms_service.cpp:415` 实现 SMSC 设置。

---

### Q11: 双卡设备如何选择发送卡槽

**问题描述**:
设备有两张 SIM 卡，需要指定使用哪张卡发送短信。

**解决方案**:

1. **单次发送指定卡槽**:
```javascript
// slotId: 0 = 卡槽1, 1 = 卡槽2
let msg = {
    slotId: 1,  // 使用卡槽2
    destinationHost: '10086',
    content: 'test',
    sendCallback: callback
};
sms.sendMessage(msg);
```

2. **设置默认卡槽**:
```javascript
// 设置默认短信卡槽
await sms.setDefaultSmsSlotId(1);

// 后续发送如果不指定 slotId，使用默认卡槽
```

3. **查询默认卡槽**:
```javascript
let defaultSlotId = await sms.getDefaultSmsSlotId();
console.log('Default SMS slot:', defaultSlotId);
```

**证据**: `services/sms/sms_service.cpp:564` 实现默认卡槽设置。

## 网络和协议问题

### Q12: IMS 短信无法发送

**问题描述**:
设备支持 IMS，但短信仍通过 CS 域发送。

**排查步骤**:

1. 检查 IMS 是否注册:
```javascript
let supported = await sms.isImsSmsSupported(0);
console.log('IMS SMS supported:', supported);
```

2. 检查网络策略:
```bash
hdc shell "hilog | grep -i 'ims\|networktype'"
```

3. 强制使用 IMS 发送:
```cpp
// 在代码中，网络策略管理器决定发送域
// SmsNetworkPolicyManager::GetNetWorkType() 返回 NET_TYPE_IMS
```

**证据**: `services/sms/sms_network_policy_manager.cpp:108-144` 实现网络类型决策。

---

### Q13: CDMA 网络短信发送失败

**问题描述**:
在 CDMA 网络下短信发送失败。

**可能原因**:

1. 网络类型识别错误:
```bash
# 检查网络类型识别日志
hdc shell "hilog | grep -i 'isCTSimCard\|netWorkType'"
```

2. CDMA 短信中心配置问题:
- CDMA 通常不需要显式配置 SMSC
- 检查运营商配置

3. 漫游状态:
```cpp
// services/sms/sms_network_policy_manager.cpp
bool isRoaming = false;
CoreManagerInner::GetInstance().IsCTSimCard(slotId_, isCTSimCard);
if (isCTSimCard && !isRoaming) {
    netWorkType_ = NET_TYPE_CDMA;
}
```

**证据**: `services/sms/cdma/cdma_sms_sender.cpp` 实现 CDMA 短信发送。

---

### Q14: MMS 下载失败

**问题描述**:
收到彩信通知后，下载 MMS 失败。

**排查步骤**:

1. 检查网络连接:
```bash
# 确保数据连接可用
hdc shell "hilog | grep -i 'network\|datacall'"
```

2. 检查 MMSC 配置:
```javascript
// MMS 需要正确的 MMSC URL
let params = {
    slotId: 0,
    mmsc: 'http://mmsc.example.com',  // 必须正确
    data: '/path/to/save.mms',
    ua: 'OpenHarmony MMS',
    uaprof: 'http://profile.example.com'
};
```

3. 检查代理配置:
```bash
# 查看 APN 代理配置
hdc shell "hilog | grep -i 'GetMmsApnPorxy'"
```

4. 检查文件权限:
```bash
# 确保应用有写入权限
hdc shell "ls -la /path/to/save.mms"
```

**证据**: `services/mms/mms_network_client.cpp:252-302` 实现 HTTP 请求。

## 调试技巧

### 开启详细日志

```bash
# 设置日志级别为 DEBUG
hdc shell "param set persist.sys.hilog.debug.on 1"

# 查看短信模块日志
hdc shell "hilog | grep -iE 'SmsMms|SmsService|SmsSendManager|SmsReceiveManager'"
```

### 查看 SA 状态

```bash
# 查看 SA 4008 状态
hdc shell "sa dump 4008"

# 查看所有 telephony 相关 SA
hdc shell "sa list | grep -i telephony"
```

### 分析崩溃日志

```bash
# 如果 telephony 进程崩溃，查看 tombstone
hdc shell "ls -la /data/tombstones/"
hdc shell "cat /data/tombstones/tombstone_00"
```

### 网络抓包

```bash
# 抓取 RIL 层通信
hdc shell "tcpdump -i any -w /data/ril.pcap"

# 抓取 MMS HTTP 流量
hdc shell "tcpdump -i any port 80 or port 443 -w /data/mms.pcap"
```

## 相关跳转链接

- [项目概览](00_Overview.md) - 了解项目定位
- [N-API 接口](03_NAPI_Interface.md) - 了解 API 使用方法
- [架构说明](02_Architecture.md) - 了解内部工作原理
- [安全评审](06_Security_Review.md) - 了解权限要求
