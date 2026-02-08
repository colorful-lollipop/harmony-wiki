# 常见问题与调试指南

> **目的**: 收集常见构建、运行和调试问题及其解决方案  
> **适用范围**: 所有使用位置服务组件的开发者  
> **最后更新**: 2026-02-05

---

## 1. 构建问题

### 1.1 GN 构建失败

**问题**: 执行 `hb build` 时构建失败

**可能原因**:
1. 依赖组件未正确配置
2. Feature 开关配置错误
3. 路径变量未定义

**排查步骤**:

```bash
# 1. 检查 GN 变量定义
cat config.gni | grep LOCATION_ROOT_DIR

# 2. 检查依赖组件
cat bundle.json | grep components

# 3. 检查 feature 开关
cat config.gni | grep location_feature
```

**解决方案**:
1. 确认所有依赖组件已安装
2. 检查 `config.gni` 语法正确性
3. 清理构建缓存后重试

```bash
# 清理构建缓存
rm -rf out/
hb build -f
```

---

### 1.2 头文件找不到

**问题**: 编译时提示头文件找不到

**错误示例**:
```
fatal error: 'locator_ability.h' file not found
```

**解决方案**:
1. 检查 include 路径配置
2. 确认依赖关系正确
3. 检查头文件是否存在于 `interfaces/inner_api/include/`

```bash
# 查找头文件
find . -name "locator_ability.h" -not -path "*/test/*"
```

---

### 1.3 SA Profile 构建失败

**问题**: SA 配置文件构建失败

**错误示例**:
```
error: SA profile JSON parse error
```

**解决方案**:
1. 检查 JSON 语法正确性
2. 确认 SA ID 格式正确
3. 验证 libpath 路径

```bash
# 验证 JSON 语法
cat sa_profile/2801.json | python3 -m json.tool
```

---

## 2. 运行问题

### 2.1 定位服务不可用

**问题**: 调用 API 返回 `LOCATION_SERVICE_UNAVAILABLE` (3301000)

**可能原因**:
1. 定位服务 SA 未启动
2. SA 依赖的 HDI 服务异常
3. 定位开关未开启

**排查步骤**:

```bash
# 1. 检查 SA 状态
hdb shell sa ConnUafind 2801
hdb shell sa ConnUafind 2802
hdb shell sa ConnUafind 2803

# 2. 检查定位开关状态
hdb shell param get const.location开关状态

# 3. 查看日志
hdb shell hidump -tag location
```

**解决方案**:
1. 确认设备定位功能正常
2. 重启定位服务
3. 检查 HDI 服务状态

```bash
# 重启定位服务
hdb shell stop location_service
hdb shell start location_service
```

---

### 2.2 权限被拒绝

**问题**: 调用 API 返回 `LOCATION_PERMISSION_DENIED` (201)

**可能原因**:
1. 未在配置文件中声明权限
2. 未动态申请权限
3. 权限被用户拒绝

**解决方案**:

1. 在 `module.json5` 中声明权限：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.LOCATION",
        "usedScene": {
          "when": "inuse"
        }
      }
    ]
  }
}
```

2. 运行时动态申请权限（JS）：

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

let atManager = abilityAccessCtrl.createAtManager();
let bundleInfo = bundleManager.getBundleInfoForSelfSync(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
);
let tokenId = bundleInfo.appInfo.accessTokenId;

atManager.requestPermissionFromUser(
    ['ohos.permission.LOCATION'],
    (error, result) => {
        if (error) {
            console.error('Request permission error:', error);
            return;
        }
        console.log('Permission result:', result);
    }
);
```

---

### 2.3 定位开关关闭

**问题**: 调用 API 返回 `LOCATION_SWITCH_OFF` (3301100)

**解决方案**:
1. 引导用户开启定位开关
2. 使用 API 检查开关状态

```typescript
import geolocation from '@ohos.geolocation';

let isEnabled = geolocation.isLocationEnabled();
if (!isEnabled) {
    // 引导用户开启定位
    geolocation.enableLocation();
}
```

---

## 3. 调试方法

### 3.1 日志查看

**位置服务日志标签**: `location`

```bash
# 查看所有 location 日志
hdb shell hidump -tag location

# 查看调试级别日志
hdb shell hidump -tag location -v debug

# 过滤关键字
hdb shell hidump -tag location | grep "locator"
```

### 3.2 HiTrace 追踪

```bash
# 开启追踪
hdb shell hitsTrace -t 0x1000 -b 4096 -n location

# 执行定位操作

# 停止追踪并保存
hdb shell hitsTrace -s
```

### 3.3 性能分析

```bash
# CPU 性能分析
hdb shell perfprofiler location_service

# 内存分析
hdb shell memprof -p $(pidof locationhub)
```

---

## 4. 常见错误码

| 错误码 | 常量名 | 说明 | 解决方案 |
|--------|--------|------|----------|
| 0 | `LOCATION_SUCCESS` | 成功 | - |
| 201 | `LOCATION_PERMISSION_DENIED` | 权限拒绝 | 申请权限 |
| 401 | `LOCATION_INVALID_PARAM` | 参数错误 | 检查输入参数 |
| 801 | `LOCATION_NOT_SUPPORTED` | 能力不支持 | 检查设备能力 |
| 3301000 | `LOCATION_SERVICE_UNAVAILABLE` | 服务不可用 | 检查 SA 状态 |
| 3301100 | `LOCATION_SWITCH_OFF` | 开关关闭 | 开启定位开关 |

---

## 5. 调试技巧

### 5.1 模拟位置数据

**适用于**: 测试环境

```typescript
// 开启模拟位置
geolocation.setMockMode(true);

// 设置模拟位置
geolocation.setMockLocation({
    latitude: 39.9042,
    longitude: 116.4074,
    accuracy: 10
});
```

### 5.2 单元测试

```bash
# 运行定位相关单元测试
hdb shell run -p /system/etc/ut(location_*) -t unit
```

### 5.3 压力测试

```bash
# 高频定位测试
for i in {1..100}; do
    geolocation.getCurrentLocation()
    sleep 0.1
done
```

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [C/N-API 接口](02_C_NAPI.md) | 错误码参考 |
| [JS API 接口](03_JS_API.md) | API 使用示例 |
| [GN 构建配置](05_Build.md) | 构建配置说明 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
