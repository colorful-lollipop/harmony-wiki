# 常见问题

## 目的

本文档汇总 PrintSpooler 项目的常见问题、构建、运行、调试问题和定位路径。

## 适用范围

本文档适用于：

- 新人开发者
- 运维人员
- 需要问题排查的开发者

## 关键结论

1. **构建问题**：主要是签名和权限问题
2. **运行问题**：权限未授予、服务未启动
3. **调试问题**：日志查看和过滤
4. **常见错误**：参数错误、权限错误、服务错误

---

## 构建问题

### Q1: 构建失败，提示签名错误？

**症状**：
```
Build failed. Signature verification failed.
```

**原因**：
- 签名文件配置错误
- 签名文件路径不正确

**解决方案**：
1. 检查签名文件配置
   - 打开 DevEco Studio
   - File → Project Structure → Signing Configs
2. 确认签名文件存在
   - `signature/spooler.p7b`
   - 密码是否正确（默认 123456）
3. 重新构建

**证据**：README.md:295-308

---

### Q2: 构建失败，提示找不到模块？

**症状**：
```
Module not found: @ohos/common
```

**原因**：
- 模块依赖配置错误
- HAR 包未正确构建

**解决方案**：
1. 检查 `oh-package.json5` 依赖配置
2. 确认依赖路径正确：
   ```json5
   "dependencies": {
     "@ohos/common": "file:../common"
   }
   ```
3. 先构建依赖的 HAR 模块：
   ```bash
   hvigorw --mode module -p module=common
   hvigorw --mode module -p module=ippPrint
   hvigorw --mode module -p module=entry
   ```

**证据**：`entry/oh-package.json5`

---

### Q3: 如何切换不同产品的构建？

**问题**：需要构建 tablet 产品版本

**解决方案**：
```bash
# 构建 tablet 产品
hvigorw --mode module -p module=entry@tablet

# 构建 default 产品
hvigorw --mode module -p module=entry@default
```

**证据**：`build-profile.json5:5-20`

---

## 运行问题

### Q4: 应用安装后无法启动？

**症状**：
- 应用安装后，重启系统后应用没有自动启动
- 启动时报错

**原因**：
- 权限未授予
- 签名不匹配
- 之前安装的缓存未清理

**解决方案**：
1. 检查权限是否授予
   ```bash
   hdc shell pm list granted
   ```
2. 清理缓存（如果之前安装过）：
   ```bash
   hdc shell rm -rf /data/misc_de/0/mdds/0/default/bundle_manager_service
   hdc shell rm -rf /data/accounts
   ```
3. 重新签名并安装
4. 重启系统：
   ```bash
   hdc shell
   reboot
   ```

**证据**：README.md:357-361

---

### Q5: 无法发现打印机？

**症状**：
- 打印机列表为空
- 扫描超时

**原因**：
- WiFi 未开启
- 权限未授予
- 打印机不支持 Mopria 协议

**解决方案**：
1. 检查 WiFi 是否开启
   ```bash
   hdc shell
   ```
   在系统设置中查看 WiFi 状态
2. 检查权限
   - `ohos.permission.GET_WIFI_INFO`
   - `ohos.permission.SET_WIFI_INFO`
3. 确认打印机支持 IPP 协议
4. 查看日志：
   ```bash
   hdc shell hilog -t PrintSpooler
   ```

**证据**：`feature/ippPrint/src/main/ets/common/discovery/`

---

### Q6: 打印任务卡在"运行中"状态？

**症状**：
- 打印任务状态一直显示"运行中"
- 打印机实际没有工作

**原因**：
- 打印机连接断开
- IPP 通信失败
- 打印机出错

**解决方案**：
1. 检查打印机连接状态
   ```bash
   hdc shell hilog -t P2pPrinterConnection
   ```
2. 检查 IPP 通信日志
   ```bash
   hdc shell hilog -t Backend
   ```
3. 尝试断开并重新连接打印机
4. 取消任务并重新发送

**证据**：`entry/src/main/ets/Controller/PrintJobController.ets:177-201`

---

## 调试问题

### Q7: 如何查看应用日志？

**方法 1：实时查看**
```bash
hdc shell hilog -t PrintSpooler
```

**方法 2：输出到文件**
```bash
hdc shell hilog > hilog.log
# 按 Ctrl+C 停止
```

**方法 3：过滤关键字**
```bash
hdc shell hilog | grep PrintSpooler
hdc shell hilog | grep "error"
```

**证据**：README.md:369-394

---

### Q8: 如何过滤特定模块的日志？

**方法**：使用 TAG 过滤
```bash
# 过滤 MainAbility 日志
hdc shell hilog | grep "MainAbility"

# 过滤 P2P 发现日志
hdc shell hilog | grep "P2PDiscovery"
```

**可用 TAG 列表**（从代码推断）：
- `[MainAbility]:` - 主 Ability
- `PrintExtension` - 打印扩展
- `P2pPrinterConnection` - P2P 连接
- `MdnsDiscovery` - mDNS 发现
- `Backend` - IPP 后端

**证据**：
- MainAbility.ets:34 - `const TAG = '[MainAbility]:';`
- PrintExtension.ts:34 - `const TAG = 'PrintExtension';`

---

### Q9: 如何启用详细日志？

**方法**：修改日志级别

1. 找到日志工具类
   - `common/src/main/ets/utils/Log.ts`
2. 修改日志输出
   - 当前使用 `Log.info/error/debug`
   - 确认这些方法正确实现
3. 重新构建并安装

**证据**：`common/src/main/ets/utils/Log.ts`

---

## 常见错误码

### E_PRINT_INVALID_PARAMETER (401)

**原因**：参数类型或范围错误

**常见场景**：
- 文件路径为空
- 打印机 URI 格式错误
- 打印属性 JSON 格式错误

**定位路径**：
1. 查看错误发生位置的代码
2. 检查参数传递
3. 查看 Want 参数：
   ```bash
   hdc shell hilog | grep "onSessionCreate"
   ```

**证据**：`common/src/main/ets/model/Constants.ts:36`

### E_PRINT_SERVER_FAILURE (13100003)

**原因**：打印服务失败

**常见场景**：
- 打印服务未启动
- CUPS 服务异常
- 打印机连接断开

**定位路径**：
1. 检查打印服务状态：
   ```bash
   hdc shell ps -A | grep print
   ```
2. 查看 CUPS 日志：
   ```bash
   hdc shell hilog | grep "cups"
   ```

**证据**：`common/src/main/ets/model/Constants.ts:39`

### E_PRINT_INVALID_PRINTER (13100005)

**原因**：无效打印机

**常见场景**：
- 打印机未连接
- 打印机不支持 IPP
- 打印机能力查询失败

**定位路径**：
1. 检查打印机连接状态
2. 查看发现日志：
   ```bash
   hdc shell hilog | grep -E "P2P|Mdns"
   ```
3. 尝试手动连接打印机

**证据**：`common/src/main/ets/model/Constants.ts:40`

---

## 性能问题

### Q10: 预览界面卡顿？

**症状**：
- 图片预览滚动不流畅
- 参数切换响应慢

**原因**：
- 图片文件过大
- 频繁重新渲染
- 主线程阻塞

**解决方案**：
1. 检查图片大小
   - 常量：`Constants.MAX_PIXELMAP = 33554432`（32MB）
2. 使用 Worker 处理图片
   - 已有：`entry/src/main/ets/workers/PrintWorker.ts`
3. 减少预览图片数量
4. 优化图像处理算法

**证据**：
- Constants.ts:120
- PrintWorker.ts - Worker 使用

---

## 权限问题

### Q11: 提示"无权限"错误？

**症状**：
```
Error: E_PRINT_NO_PERMISSION (201)
```

**原因**：
- 权限未授予
- 权限声明错误

**解决方案**：
1. 检查权限列表：
   ```bash
   hdc shell pm list permissions
   ```
2. 手动授予权限
3. 检查 `module.json5` 权限声明

**证据**：
- Constants.ts:35
- entry/src/main/module.json5:66-161

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目整体介绍
- [对外 API](04_External_API.md) - OpenHarmony 系统 API 使用
- [安全风险评审](08_Security_Review.md) - 安全问题和加固建议
