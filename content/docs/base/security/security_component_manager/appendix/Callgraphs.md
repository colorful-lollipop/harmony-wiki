# 附录：调用链分析 - Security Component Manager

> 目的：跟踪关键 API 调用链，理解数据流与执行路径

---

## 适用范围

本文档适用于：
- 需要调试问题的开发者
- 需要理解执行流程的开发者
- 需要性能优化的开发者

---

## 关键结论

1. **组件注册**：应用 → SecCompKit → IPC → SecCompService → SecCompManager → SecCompEntity
2. **点击事件处理**：应用 → SecCompKit → IPC → SecCompService → SecCompManager → SecCompPermManager
3. **权限授予**：SecCompPermManager → AccessToken 服务
4. **权限撤销**：AppStateObserver → SecCompPermManager → AccessToken 服务

---

## 1. RegisterSecurityComponent 调用链

```mermaid
sequenceDiagram
    participant App as 应用进程
    participant Kit as SecCompKit
    participant Client as SecCompClient
    participant Proxy as ISecCompService<br/>Proxy
    participant Service as SecCompService
    participant Manager as SecCompManager
    participant Entity as SecCompEntity
    participant Base as SecCompBase
    participant Enhance as SecCompEnhanceAdapter

    App->>Kit: 1. 调用<br/>RegisterSecurityComponent(type, info, scId)

    Note over Kit: 获取 Proxy

    Kit->>Client: 2. 获取 Proxy<br/>GetProxy(doLoadSa)
    alt Proxy 未缓存
        Client->>SAMgr: 3a. LoadSystemAbility(3506, callback)
        SAMgr-->>Client: 4a. OnLoadSuccess(remoteObject)
        Client->>Client: 5a. iface_cast<ISecCompService>(remote)
    else Proxy 已缓存
        Client-->>Kit: 4b. 返回缓存的 Proxy
    end

    Note over Kit: IPC 调用

    Kit->>Proxy: 6. RegisterSecurityComponent(rawData, rawReply)
    Proxy-->>Service: 7. IPC 调用

    Note over Service: Service 端处理

    Service->>Service: 8. RegisterReadFromRawdata(rawData)
    Service->>Service: 9. ParseParams(info, caller, json)

    Note over Service,Base: 预处理

    Service->>Enhance: 10. EnhanceDataPreprocess(scId, info)
    Enhance-->>Service: 11. 返回处理后的 info

    Note over Service,Entity: 验证组件信息

    Service->>Manager: 12. RegisterSecurityComponent(type, json, caller, scId)
    Manager->>Manager: 13. 生成 scId<br/>CreateScId()
    Manager->>Base: 14. 解析 JSON<br/>SecCompBase::ParseFromJson()
    Manager->>Base: 15. 验证组件信息<br/>SecCompBase::CheckComponentInfo()

    alt 验证失败
        Base-->>Manager: 16a. 返回错误
        Manager-->>Service: 17a. 返回错误
    else 验证成功
        Manager->>Entity: 18b. 创建 SecCompEntity
        Entity-->>Manager: 19b. 返回 Entity
    end

    Note over Service,Manager: 存储组件

    Manager->>Manager: 20. AddSecurityComponentToList(pid, tokenId, entity)
    Manager-->>Service: 21. 返回 scId
    Service->>Service: 22. RegisterWriteToRawdata(res, scId, rawReply)
    Service-->>Proxy: 23. 返回 scId（序列化）
    Proxy-->>Kit: 24. 返回 scId（反序列化）
    Kit-->>App: 25. 返回 scId

    Note over App: 注册成功
    App->>App: 26. 组件注册成功，显示 UI
```

**关键验证点**：

| 步骤 | 验证内容 | 文件:行 |
|------|----------|----------|
| 9 | 调用者 Token/UID 校验 | `sec_comp_service.cpp:78-82` |
| 11 | 增强数据预处理 | `sec_comp_enhance_adapter.cpp:154-166` |
| 14 | JSON 解析 | `sec_comp_info_helper.cpp` |
| 15 | 组件信息验证 | `sec_comp_base.cpp` |

---

## 2. ReportSecurityComponentClickEvent 调用链

```mermaid
sequenceDiagram
    participant App as 应用进程
    participant Kit as SecCompKit
    participant Proxy as ISecCompService<br/>Proxy
    participant Service as SecCompService
    participant Manager as SecCompManager
    participant Entity as SecCompEntity
    participant PermMgr as SecCompPermManager
    participant Window as WindowInfoHelper
    participant Access as AccessToken 服务
    participant Dialog as Permission Manager<br/>对话框

    App->>Kit: 1. 调用<br/>ReportSecurityComponentClickEvent(info, token, callback, msg)

    Note over Kit: IPC 调用

    Kit->>Proxy: 2. ReportSecurityComponentClickEvent(rawData, rawReply)
    Proxy-->>Service: 3. IPC 调用

    Note over Service: Service 端处理

    Service->>Service: 4. ParseParams(info, caller, json)

    Note over Service,Manager: 参数验证

    Service->>Manager: 5. CheckClickEventParams(caller, remote)
    Manager->>Manager: 6. 验证 callerToken 和 dialogCallback
    Manager-->>Service: 7. 返回验证结果

    alt 参数验证失败
        Service-->>Proxy: 8a. 返回错误
        Proxy-->>Kit: 9a. 返回错误
        Kit-->>App: 10a. 返回错误
    else 参数验证成功
        Service->>Manager: 11b. ReportSecurityComponentClickEvent(info, json, caller, remote, msg)

        Note over Manager,Entity: 获取组件并验证

        Manager->>Entity: 12. GetSecurityComponentFromList(pid, scId)
        Manager->>Entity: 13. CheckClickInfo(caller)

        Entity->>Window: 14a. 检查窗口覆盖<br/>CheckWindowCover()
        Window-->>Entity: 15a. 返回覆盖结果

        Entity->>Entity: 16b. 检查点事件<br/>CheckPointEvent(touchX, touchY)
        Entity-->>Entity: 17b. 返回点事件结果

        Entity->>Entity: 18c. 检查时间戳
        Entity-->>Entity: 19c. 返回时间戳结果

        Entity->>Entity: 20d. 检查键盘事件<br/>CheckKeyEvent(keyCode)
        Entity-->>Entity: 20d. 返回键盘事件结果

        Entity->>Manager: 21. 返回验证结果
        Manager-->>Service: 22. 返回验证结果

        Note over Service,Entity: 增强数据验证

        Service->>Enhance: 23. CheckExtraInfo(clickInfo)
        Enhance-->>Service: 24. 返回增强验证结果

        alt 增强验证失败
            Service-->>Proxy: 25a. 返回错误
            Proxy-->>Kit: 26a. 返回错误
            Kit-->>App: 27a. 返回错误
        else 增强验证成功
            Service->>Manager: 28b. 检查恶意应用<br/>SecCompMaliciousApps::CheckMaliciousApps()

            alt 在黑名单
                Manager-->>Service: 29a. 返回错误
                Service-->>Proxy: 30a. 返回错误
                Proxy-->>Kit: 31a. 返回错误
                Kit-->>App: 32a. 返回错误
            else 不在黑名单
                Manager->>PermMgr: 33b. GrantTempPermission(tokenId, componentInfo)

                Note over PermMgr: 权限授予

                alt LocationButton 或 PasteButton
                    PermMgr->>Access: 34a. GrantPermission(tokenId, permissionName)
                    Access-->>PermMgr: 35a. 返回授予成功
                else SaveButton
                    PermMgr->>PermMgr: 36b. GrantTempSavePermission(tokenId)
                    PermMgr->>PermMgr: 37c. 增加引用计数
                    PermMgr->>PermMgr: 38d. 启动 60 秒延迟撤销
                    PermMgr-->>PermMgr: 39b. 返回授予成功
                end

                PermMgr-->>Manager: 40. 返回授予成功
                Manager-->>Service: 41. 返回成功
                Service-->>Proxy: 42. 返回成功
                Proxy-->>Kit: 43. 返回成功
                Kit-->>App: 44. 返回成功

                Note over App: 权限授予成功，访问敏感数据
                App->>App: 45. 访问位置/粘贴/保存数据
            end
        end
    end
```

**关键验证点**：

| 步骤 | 验证内容 | 文件:行 |
|------|----------|----------|
| 6 | callerToken 和 dialogCallback 验证 | `sec_comp_manager.cpp:64-76` |
| 14a | 窗口覆盖检查 | `window_info_helper.cpp` |
| 14b | 坐标范围检查 | `sec_comp_entity.cpp` |
| 14c | 时间戳检查（5000ms 内） | `sec_comp_entity.cpp:150-155` |
| 14d | 键盘事件检查（SPACE/ENTER/NUMPAD_ENTER） | `sec_comp_entity.cpp:156-174` |
| 24 | 增强数据验证（HMAC/Challenge） | `sec_comp_enhance_adapter.cpp:116-129` |

---

## 3. GrantTempPermission 调用链

```mermaid
sequenceDiagram
    participant Manager as SecCompManager
    participant PermMgr as SecCompPermManager
    participant Handler as SecEventHandler
    participant Access as AccessToken 服务

    Note over PermMgr,Access: 权限授予

    Manager->>PermMgr: 1. GrantTempPermission(tokenId, componentInfo)

    Note over PermMgr: 根据组件类型授予

    PermMgr->>PermMgr: 2. 检查组件类型

    alt LocationButton
        PermMgr->>Access: 3a. GrantPermission(tokenId, "ohos.permission.LOCATION")
        Access-->>PermMgr: 4a. 授予成功
        PermMgr->>Access: 5a. GrantPermission(tokenId, "ohos.permission.APPROXIMATELY_LOCATION")
        Access-->>PermMgr: 6a. 授予成功
        PermMgr-->>Manager: 7a. 返回 SC_OK
    else PasteButton
        PermMgr->>Access: 8b. GrantPermission(tokenId, "ohos.permission.SECURE_PASTE")
        Access-->>PermMgr: 9b. 授予成功
        PermMgr-->>Manager: 10b. 返回 SC_OK
    else SaveButton
        PermMgr->>PermMgr: 11c. GrantTempSavePermission(tokenId)

        Note over PermMgr,Handler: Save 权限引用计数

        PermMgr->>PermMgr: 12d. 增加引用计数<br/>applySaveCountMap_[tokenId]++
        PermMgr->>PermMgr: 13e. 创建任务名<br/>"RevokeSavePermission_" + tokenId
        PermMgr->>Handler: 14f. PostDelayedTask(callback, 60000, taskName)
        Handler-->>PermMgr: 15f. 任务已调度
        PermMgr-->>Manager: 16c. 返回 SC_OK
    end

    Manager-->>Service: 17. 返回授予成功
```

**关键逻辑**：

| 组件类型 | 权限名称 | 撤销时机 |
|---------|----------|----------|
| LocationButton | `ohos.permission.LOCATION` | 后台 10 秒 |
| LocationButton | `ohos.permission.APPROXIMATELY_LOCATION` | 后台 10 秒 |
| PasteButton | `ohos.permission.SECURE_PASTE` | 后台 10 秒 |
| SaveButton | 内部引用计数 | 操作完成 60 秒 |

**证据路径**：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:279-323`

---

## 4. RevokeAppPermissions 调用链

```mermaid
sequenceDiagram
    participant Observer as AppStateObserver
    participant Manager as SecCompManager
    participant PermMgr as SecCompPermManager
    participant Handler as SecEventHandler
    participant Access as AccessToken 服务

    Note over Observer,Handler: 应用进入后台

    Observer->>Manager: 1. NotifyProcessBackground(pid)

    Note over Manager,PermMgr: 延迟撤销

    Manager->>PermMgr: 2. RevokeAppPermissionsDelayed(tokenId)
    PermMgr->>Handler: 3. 创建任务名<br/>"RevokePermission_" + tokenId
    PermMgr->>Handler: 4. PostDelayedTask(revokeCallback, 10000, taskName)
    Handler-->>PermMgr: 5. 任务已调度（10 秒后）

    Note over Handler: 延迟 10 秒

    Handler->>PermMgr: 6. 执行延迟任务
    PermMgr->>PermMgr: 7. RevokeAppPermissionsImmediately(tokenId)

    Note over PermMgr,Access: 权限撤销

    alt LocationButton 或 PasteButton
        PermMgr->>Access: 8a. RevokePermission(tokenId, "ohos.permission.LOCATION")
        Access-->>PermMgr: 9a. 撤销成功
        PermMgr->>Access: 10a. RevokePermission(tokenId, "ohos.permission.APPROXIMATELY_LOCATION")
        Access-->>PermMgr: 11a. 撤销成功
        PermMgr->>PermMgr: 12a. 从 grantMap_ 删除
    else SaveButton
        PermMgr->>PermMgr: 13b. RevokeTempSavePermissionCount(tokenId)
        PermMgr->>PermMgr: 14b. 减少引用计数<br/>applySaveCountMap_[tokenId]--
        alt 引用计数 == 0
            PermMgr->>PermMgr: 15c. 清空任务队列<br/>saveTaskDequeMap_[tokenId].clear()
            PermMgr->>PermMgr: 16c. 从 grantMap_ 删除
        else 引用计数 > 0
            PermMgr->>PermMgr: 17d. 跳过撤销（仍有并发保存）
        end
    end

    PermMgr-->>Handler: 18. 返回撤销完成
```

**证据路径**：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:43-59`

---

## 5. VerifySavePermission 调用链

```mermaid
sequenceDiagram
    participant App as 应用进程
    participant Kit as SecCompKit
    participant Proxy as ISecCompService<br/>Proxy
    participant Service as SecCompService
    participant PermMgr as SecCompPermManager

    App->>Kit: 1. 调用<br/>VerifySavePermission(tokenId)

    Note over Kit: IPC 调用

    Kit->>Proxy: 2. VerifySavePermission(tokenId, isGranted)
    Proxy-->>Service: 3. IPC 调用

    Note over Service,PermMgr: 验证保存权限

    Service->>PermMgr: 4. VerifySavePermission(tokenId)

    Note over PermMgr: 引用计数检查

    PermMgr->>PermMgr: 5. 检查引用计数<br/>applySaveCountMap_.find(tokenId)
    alt 引用计数 > 0
        PermMgr-->>Service: 6a. isGranted = true
    else 引用计数 <= 0
        PermMgr-->>Service: 6b. isGranted = false
    end

    Service-->>Proxy: 7. 返回 isGranted
    Proxy-->>Kit: 8. 返回 isGranted
    Kit-->>App: 9. 返回 isGranted

    Note over App: 根据结果执行操作

    alt isGranted == true
        App->>App: 10a. 执行保存操作
    else isGranted == false
        App->>App: 10b. 显示保存按钮或错误提示
    end
```

**证据路径**：`services/security_component_service/sa/sa_main/sec_comp_perm_manager.cpp:140-161`

---

## 调用链总结

| API | 层数 | 关键验证点 | 性能敏感 |
|-----|-------|----------|----------|
| `RegisterSecurityComponent` | 9 层 | 组件验证 | 是 |
| `ReportSecurityComponentClickEvent` | 12 层 | 点击验证 + 权限授予 | 是 |
| `GrantTempPermission` | 2 层（PermMgr → Access） | 无 | 否 |
| `RevokeAppPermissions` | 3 层 | 无 | 否 |
| `VerifySavePermission` | 3 层 | 无 | 否 |

---

## 相关跳转

- [对外 API](./03_Public_APIs.md) - 查看 API 参考
- [内部 API](./04_Internal_APIs.md) - 查看内部接口详情

---

**返回 [主页](./README.md) | [导航](./SUMMARY.md)
