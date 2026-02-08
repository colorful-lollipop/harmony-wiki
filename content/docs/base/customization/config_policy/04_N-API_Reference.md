# N-API 接口文档

## 模块注册

### 模块信息

| 属性 | 值 |
|------|-----|
| 模块名称 | `configPolicy` |
| 模块版本 | 1 |
| 命名空间 | `OHOS.Customization.ConfigPolicy` |

### 注册代码

```cpp
static napi_module g_configPolicyModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = ConfigPolicyInit,
    .nm_modname = "configPolicy",
    .nm_priv = ((void *)0),
    .reserved = { 0 }
};

extern "C" __attribute__((constructor)) void ConfigPolicyRegister()
{
    napi_module_register(&g_configPolicyModule);
}
```

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp:533-546`

---

## API 清单

### 1. getOneCfgFile

获取指定配置文件的最高优先级路径。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getOneCfgFile` |
| C++ 实现 | `ConfigPolicyNapi::NAPIGetOneCfgFile` |
| 位置 | `config_policy_napi.cpp:93-97` |
| 同步/异步 | 异步 |
| 返回类型 | Promise\<string\> |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| relPath | string | 是 | 配置文件的相对路径，如 `"etc/xml/config.xml"` |
| followMode | FollowXMode | 否 | FollowX 模式，默认 `FollowXMode.DEFAULT` |
| extra | string | 否 | 用户自定义 Follow 路径（当 followMode 为 USER_DEFINED 时必填） |
| callback | AsyncCallback\<string\> | 否 | 回调函数（替代 Promise） |

#### 返回值

- Promise\<string\>：最高优先级的配置文件路径
- callback 模式：void（结果在回调中返回）

#### 错误码

| 错误码 | 条件 |
|--------|------|
| 401 | 参数类型错误（如 relPath 不是 string） |
| 401 | followMode 为 USER_DEFINED 但未提供 extra |

#### 示例

```typescript
// Promise 模式
configPolicy.getOneCfgFile('etc/xml/config.xml')
    .then((path: string) => {
        console.log('配置文件路径:', path);
    })
    .catch((err: BusinessError) => {
        console.error('获取失败:', err.code, err.message);
    });

// Callback 模式
configPolicy.getOneCfgFile('etc/xml/config.xml', 
    (err: BusinessError, path: string) => {
        if (err) {
            console.error('获取失败:', err.code, err.message);
        } else {
            console.log('配置文件路径:', path);
        }
    });

// 带 FollowX 模式
configPolicy.getOneCfgFile('etc/xml/config.xml', 
    configPolicy.FollowXMode.SIM_1)
    .then((path: string) => {
        console.log('SIM1 配置文件路径:', path);
    });
```

---

### 2. getOneCfgFileSync

同步获取指定配置文件的最高优先级路径。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getOneCfgFileSync` |
| C++ 实现 | `ConfigPolicyNapi::NAPIGetOneCfgFileSync` |
| 位置 | `config_policy_napi.cpp:99-103` |
| 同步/异步 | 同步 |
| 返回类型 | string |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| relPath | string | 是 | 配置文件的相对路径 |
| followMode | FollowXMode | 否 | FollowX 模式 |
| extra | string | 否 | 用户自定义 Follow 路径 |

#### 返回值

string：最高优先级的配置文件路径，未找到时返回空字符串

#### 示例

```typescript
const path = configPolicy.getOneCfgFileSync('etc/xml/config.xml');
if (path) {
    console.log('配置文件路径:', path);
} else {
    console.log('未找到配置文件');
}
```

---

### 3. getCfgFiles

获取指定配置文件的所有层级路径（按优先级从低到高排序）。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getCfgFiles` |
| C++ 实现 | `ConfigPolicyNapi::NAPIGetCfgFiles` |
| 位置 | `config_policy_napi.cpp:194-198` |
| 同步/异步 | 异步 |
| 返回类型 | Promise\<string[]\> |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| relPath | string | 是 | 配置文件的相对路径 |
| followMode | FollowXMode | 否 | FollowX 模式 |
| extra | string | 否 | 用户自定义 Follow 路径 |
| callback | AsyncCallback\<string[]> | 否 | 回调函数 |

#### 返回值

Promise\<string[]\>：所有层级的配置文件路径数组

#### 示例

```typescript
configPolicy.getCfgFiles('etc/xml/config.xml')
    .then((paths: string[]) => {
        console.log('找到', paths.length, '个配置文件');
        paths.forEach((path, index) => {
            console.log(`层级 ${index}: ${path}`);
        });
    });
```

---

### 4. getCfgFilesSync

同步获取指定配置文件的所有层级路径。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getCfgFilesSync` |
| C++ 实现 | `ConfigPolicyNapi::NAPIGetCfgFilesSync` |
| 位置 | `config_policy_napi.cpp:200-204` |
| 同步/异步 | 同步 |
| 返回类型 | string[] |

#### 示例

```typescript
const paths = configPolicy.getCfgFilesSync('etc/xml/config.xml');
paths.forEach((path) => {
    console.log(path);
});
```

---

### 5. getCfgDirList

获取配置目录列表。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getCfgDirList` |
| C++ 实现 | `ConfigPolicyNapi::NAPIGetCfgDirList` |
| 位置 | `config_policy_napi.cpp:206-224` |
| 同步/异步 | 异步 |
| 返回类型 | Promise\<string[]\> |

#### 参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| callback | AsyncCallback\<string[]> | 否 | 回调函数 |

#### 返回值

Promise\<string[]\>：配置目录路径数组

#### 示例

```typescript
configPolicy.getCfgDirList()
    .then((dirs: string[]) => {
        console.log('配置目录数量:', dirs.length);
        dirs.forEach((dir) => {
            console.log(dir);
        });
    });
```

---

### 6. getCfgDirListSync

同步获取配置目录列表。

| 属性 | 值 |
|------|-----|
| JS 方法名 | `getCfgDirListSync` |
| C++ 实现 | `ConfigPolicyNapi::NAPIGetCfgDirListSync` |
| 位置 | `config_policy_napi.cpp:226-230` |
| 同步/异步 | 同步 |
| 返回类型 | string[] |

#### 示例

```typescript
const dirs = configPolicy.getCfgDirListSync();
console.log('配置目录:', dirs);
```

---

## FollowXMode 枚举

FollowXMode 用于指定配置文件选择策略。

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `FollowXMode.DEFAULT` | 0 | 使用默认 Follow 规则 |
| `FollowXMode.NO_RULE_FOLLOWED` | 1 | 不使用任何 Follow 规则 |
| `FollowXMode.SIM_DEFAULT` | 10 | 根据默认 SIM 卡选择 |
| `FollowXMode.SIM_1` | 11 | 根据 SIM 1 卡选择 |
| `FollowXMode.SIM_2` | 12 | 根据 SIM 2 卡选择 |
| `FollowXMode.USER_DEFINED` | 100 | 用户自定义规则（需配合 extra 参数） |

**代码证据**: `interfaces/kits/js/src/config_policy_napi.cpp:68-91`

---

## 错误处理

### 参数校验

所有 API 都会进行严格的参数校验：

| 参数 | 校验规则 |
|------|----------|
| relPath | 必须是 string 类型 |
| followMode | 必须是 number 类型，且必须在 FollowXMode 枚举范围内 |
| extra | 必须是 string 类型，且当 followMode=USER_DEFINED 时必填 |
| callback | 必须是 function 类型 |

**代码证据**: `config_policy_napi.cpp:464-520`

### 错误抛送

```cpp
napi_throw_error(env, std::to_string(errCode).c_str(), errMessage.c_str());
```

**代码证据**: `config_policy_napi.cpp:522-526`

---

## 相关文档

- [C++ 内部 API](./05_Cpp_Inner_API.md)
- [架构设计](./03_Architecture.md)
