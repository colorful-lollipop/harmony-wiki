# 常见问题

## 目的

本文档列出 CalendarData 组件的常见构建、运行、调试问题和定位路径。

## 适用范围

- 目标读者：所有开发者、运维人员
- 知识级别：初级到高级

## 构建问题

### Q1: 编译报错 "找不到 napi/native_api.h"

**症状**:
```
error: 'napi/native_api.h' file not found
```

**原因**: 缺少 N-API 头文件路径

**解决**:
1. 检查 `calendarmanager/BUILD.gn` 中的 `include_dirs` 配置
2. 确保 `napi:ace_napi` 在 `external_deps` 中
3. 检查 OpenHarmony SDK 是否正确安装

**证据**: calendarmanager/BUILD.gn:18-23

---

### Q2: 找不到 hiappevent 组件

**症状**:
```
error: Unable to find "hiappevent:hiappevent_innerapi"
```

**原因**: `device_usage_hiappevent_enabled` 未正确设置

**解决**:
1. 检查 `calendarmanager/calendardata.gni` 配置
2. 确保系统包含 `hiviewdfx_hiappevent` 组件
3. 手动设置 `device_usage_hiappevent_enabled = true`

**证据**: calendarmanager/calendardata.gni:14-20

---

### Q3: 符号未定义错误

**症状**:
```
error: Use of undeclared identifier 'EXTERN_C_START'
```

**原因**: 缺少宏定义

**解决**:
1. 确保 `#include <pthread.h>` 在相关头文件中
2. 检查 `calendarmanager/common/calendar_define.h` 是否被包含

---

## 运行问题

### Q4: DataShare Ability 未启动

**症状**: 应用无法连接到日历数据

**原因**:
1. HAP 未安装
2. Ability 配置错误
3. 数据库初始化失败

**定位步骤**:
```bash
# 1. 检查 HAP 安装
hdc_std shell pm list | grep com.ohos.calendardata

# 2. 检查 Ability 状态
hdc_std shell aa dump -a com.ohos.calendardata

# 3. 查看日志
hdc_std shell hilog | grep DataShareExtAbility
```

**解决**:
1. 重新安装 HAP
2. 检查 `module.json` 中的 Ability 配置
3. 查看 `datamanager/src/main/ets/utils/CalendarDataHelper.ets` 初始化代码

---

### Q5: 权限被拒绝

**症状**:
```
Error code: 1003 - Permission denied
```

**原因**: 应用未声明或授予所需权限

**定位步骤**:
```bash
# 检查应用权限
hdc_std shell bm dump -a com.ohos.calendardata

# 检查权限授予状态
hdc_std shell aa dump -a com.ohos.calendardata | grep permission
```

**解决**:
1. 在 `module.json5` 中声明权限
2. 安装时授予权限
3. 验证权限常量名称正确：
   - `ohos.permission.READ_CALENDAR`
   - `ohos.permission.WRITE_CALENDAR`
   - `ohos.permission.READ_WHOLE_CALENDAR`
   - `ohos.permission.WRITE_WHOLE_CALENDAR`

---

### Q6: 数据库初始化失败

**症状**:
```
Error: Database initialization failed
```

**原因**:
1. 数据库文件权限问题
2. 数据库损坏
3. 存储空间不足

**定位步骤**:
```javascript
// 在应用中检查
try {
  const manager = calendarManager.getCalendarManager(getContext(this));
  await manager.getAllCalendars();
} catch (error) {
  console.error('Database error:', error.code, error.message);
}
```

**解决**:
1. 清除应用数据重新安装
2. 检查 `/data/storage/el2/` 权限
3. 查看日志获取详细错误信息

**证据**: datamanager/src/main/ets/utils/CalendarDataHelper.ets

---

## 调试问题

### Q7: N-API 调用失败

**症状**:
```
Error: calendarManager is not defined
```

**原因**: N-API 模块未正确加载

**定位步骤**:
```javascript
// 检查 N-API 可用性
try {
  const calendarManager = requireNapi('calendarManager');
  console.log('N-API loaded successfully');
} catch (error) {
  console.error('N-API load failed:', error);
}
```

**解决**:
1. 确保 `libcalendarmanager.z.so` 已安装
2. 检查 N-API 注册函数：
   ```cpp
   // calendarmanager/napi/src/module_register.cpp
   extern "C" void RegisterModule(void) {
       napi_module_register(&_module);
   }
   ```
3. 查看 `bm dump` 输出确认模块加载

---

### Q8: 事件查询返回空结果

**症状**: `getEvents()` 返回空数组，但数据存在

**原因**:
1. 查询条件不匹配
2. 权限不足
3. EventFilter 配置错误

**定位步骤**:
```javascript
// 添加调试日志
const filter = new EventFilter();
filter.time = { begin: startTime, end: endTime };

console.log('Query filter:', JSON.stringify(filter));

calendar.getEvents(filter)
  .then(events => {
    console.log('Query result:', events.length, 'events');
  })
  .catch(error => {
    console.error('Query error:', error);
  });
```

**解决**:
1. 验证时间戳格式（毫秒）
2. 检查权限是否足够
3. 查看 `datamanager/src/main/ets/processor/events/EventsProcessor.ets` 查询逻辑

---

### Q9: 批量插入失败

**症状**:
```
Error: Batch insert failed
```

**原因**:
1. 单个事件数据错误
2. 批量大小超过限制
3. 数据库约束冲突

**定位步骤**:
```javascript
// 分批测试
const batchSize = 100;
for (let i = 0; i < events.length; i += batchSize) {
  const batch = events.slice(i, i + batchSize);
  try {
    await calendar.addEvents(batch);
    console.log(`Batch ${i / batchSize} succeeded`);
  } catch (error) {
    console.error(`Batch ${i / batchSize} failed:`, error);
    break;
  }
}
```

**解决**:
1. 验证事件数据格式
2. 检查重复事件 ID
3. 查看 `dataprovider/src/main/ets/DataShareAbilityDelegate.ets` 批量插入逻辑

---

## 性能问题

### Q10: 查询响应慢

**症状**:
```
getEvents() 耗时超过 1 秒
```

**原因**:
1. 查询范围过大
2. 未使用索引
3. 重复事件实例展开慢

**定位步骤**:
```javascript
// 测量查询时间
const start = Date.now();
await calendar.getEvents(filter);
const duration = Date.now() - start;
console.log(`Query duration: ${duration}ms`);
```

**解决**:
1. 缩小查询时间范围
2. 检查 `datastructure/src/main/ets/events/EventIndexes.ets` 索引定义
3. 查看 `datamanager/src/main/ets/processor/instances/InstancesProcessor.ets` 展开逻辑

---

### Q11: 内存占用高

**症状**:
应用内存持续增长

**原因**:
1. 事件对象未释放
2. 查询结果集过大
3. 循环引用

**定位步骤**:
```bash
# 查看内存使用
hdc_std shell ps -A | grep calendardata
hdc_std shell dumpsys memoryinfo calendardata
```

**解决**:
1. 限制查询结果大小
2. 及时释放对象引用
3. 检查观察者/订阅者是否正确注销

**证据**: common/src/main/ets/observer/Observer.ets

---

## 集成问题

### Q12: 与其他应用集成失败

**症状**:
其他应用无法访问日历数据

**原因**:
1. 权限未授予
2. URI 格式错误
3. 数据未正确共享

**定位步骤**:
```javascript
// 测试其他应用访问
import calendarManager from '@ohos.calendarManager';

const manager = calendarManager.getCalendarManager(getContext(this));
manager.getAllCalendars()
  .then(calendars => {
    console.log('Calendars:', calendars.length);
  })
  .catch(error => {
    console.error('Integration error:', error);
  });
```

**解决**:
1. 确保其他应用声明正确权限
2. 验证 URI 格式：
   ```
   datashare:///com.ohos.calendarData/Events/<bundleName>/<tokenId>
   ```
3. 检查 DataShare 权限配置

---

## 日志调试

### 启用详细日志

```javascript
// common/src/main/ets/utils/Log.ets
Log.log(TAG, 'Debug message');
Log.info(TAG, 'Info message');
Log.warn(TAG, 'Warning message');
Log.error(TAG, 'Error message');
```

### 查看 HILog

```bash
# 实时查看日志
hdc_std shell hilog

# 过滤特定标签
hdc_std shell hilog | grep DataShareExtAbility
hdc_std shell hilog | grep CalendarData

# 保存日志到文件
hdc_std shell hilog > calendardata.log
```

### 日志级别

| 级别 | 说明 | 使用场景 |
|------|------|---------|
| LOG_DEBUG | 调试信息 | 开发阶段 |
| LOG_INFO | 一般信息 | 正常运行 |
| LOG_WARN | 警告信息 | 异常但可恢复 |
| LOG_ERROR | 错误信息 | 需要处理 |

## 开发工具

### IDE 调试

**DevEco Studio**:
1. 设置断点在 N-API 代码
2. 使用 Profiler 分析性能
3. 使用 Network Inspector 查看数据流

### 命令行工具

```bash
# 安装 HAP
hdc_std install CalendarData.hap

# 卸载 HAP
hdc_std uninstall com.ohos.calendardata

# 查看 Ability 信息
hdc_std shell aa dump -a com.ohos.calendardata

# 清除应用数据
hdc_std shell bm clean -n com.ohos.calendardata
```

## 更多资源

### 文档

- [OpenHarmony N-API 文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/application-dev-guide-V5/)
- [DataShare 开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/data-guides-V5/)
- [权限管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/security-guides-V5/)

### 社区

- OpenHarmony 开发者论坛
- GitHub Issues

---

返回 [目录](SUMMARY.md) | [首页](README.md)
