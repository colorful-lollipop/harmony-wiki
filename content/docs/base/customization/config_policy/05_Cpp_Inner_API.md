# C++ 内部 API

## 头文件

| 头文件 | 路径 | 用途 |
|--------|------|------|
| `config_policy_utils.h` | `interfaces/inner_api/include/` | 公共 API 头文件 |
| `config_policy_impl.h` | `interfaces/inner_api/include/` | 实现相关宏定义 |

**代码证据**: `interfaces/inner_api/include/config_policy_utils.h`, `interfaces/inner_api/include/config_policy_impl.h`

---

## 常量定义

### 数组大小限制

```c
#define MAX_CFG_POLICY_DIRS_CNT   32   // 配置目录/文件最大数量
#define MAX_PATH_LEN              256  // 路径字符串最大长度
```

**代码证据**: `config_policy_utils.h:25-26`

### FollowX 模式常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `FOLLOWX_MODE_DEFAULT` | 0 | 默认 Follow 规则 |
| `FOLLOWX_MODE_NO_RULE_FOLLOWED` | 1 | 不使用 Follow 规则 |
| `FOLLOWX_MODE_SIM_DEFAULT` | 10 | 默认 SIM 卡 |
| `FOLLOWX_MODE_SIM_1` | 11 | SIM 卡 1 |
| `FOLLOWX_MODE_SIM_2` | 12 | SIM 卡 2 |
| `FOLLOWX_MODE_USER_DEFINED` | 100 | 用户自定义 |

**代码证据**: `config_policy_utils.h:44-55`

### 系统参数键

| 常量 | 值 | 用途 |
|------|-----|------|
| `CUST_KEY_POLICY_LAYER` | `"const.cust.config_dir_layer"` | 配置层级参数 |
| `CUST_FOLLOW_X_RULES` | `"const.cust.follow_x_rules"` | FollowX 规则参数 |
| `CUST_OPKEY0` | `"telephony.sim.opkey0"` | SIM 0 运营商密钥 |
| `CUST_OPKEY1` | `"telephony.sim.opkey1"` | SIM 1 运营商密钥 |

**代码证据**: `config_policy_impl.h:30-31`, `config_policy_utils.c:40-41`

### 默认配置层级

```c
#define DEFAULT_LAYER ROOT_PREFIX "/system:" ROOT_PREFIX "/chipset:" \
    ROOT_PREFIX "/sys_prod:" ROOT_PREFIX "/chip_prod"
```

**代码证据**: `config_policy_impl.h:37`

---

## 数据结构

### CfgFiles

配置文件路径列表结构体。

```c
struct CfgFiles {
    char *paths[MAX_CFG_POLICY_DIRS_CNT];  // 路径数组
};
```

**代码证据**: `config_policy_utils.h:58-60`

### CfgDir

配置目录列表结构体。

```c
struct CfgDir {
    char *paths[MAX_CFG_POLICY_DIRS_CNT];   // 目录路径数组
    char *realPolicyValue;                   // 实际的配置层级值
};
```

**代码证据**: `config_policy_utils.h:63-66`

---

## API 函数

### 1. GetOneCfgFile

获取最高优先级的配置文件路径。

```c
char *GetOneCfgFile(const char *pathSuffix, char *buf, unsigned int bufLength);
```

#### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| pathSuffix | const char* | 配置文件相对路径，如 `"etc/xml/config.xml"` |
| buf | char* | 输出缓冲区，建议长度 `MAX_PATH_LEN` |
| bufLength | unsigned int | 缓冲区长度 |

#### 返回值

- 成功：返回缓冲区指针
- 失败：返回 NULL

**代码证据**: `config_policy_utils.c:474-477`

---

### 2. GetOneCfgFileEx

带 FollowX 模式的获取最高优先级配置文件路径。

```c
char *GetOneCfgFileEx(const char *pathSuffix, char *buf, unsigned int bufLength,
                      int followMode, const char *extra);
```

#### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| pathSuffix | const char* | 配置文件相对路径 |
| buf | char* | 输出缓冲区 |
| bufLength | unsigned int | 缓冲区长度 |
| followMode | int | FollowX 模式 |
| extra | const char* | 用户自定义 Follow 路径 |

#### 返回值

- 成功：返回缓冲区指针
- 失败：返回 NULL

**代码证据**: `config_policy_utils.c:434-472`

---

### 3. GetCfgFiles

获取所有层级的配置文件路径（按优先级从低到高排序）。

```c
CfgFiles *GetCfgFiles(const char *pathSuffix);
```

#### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| pathSuffix | const char* | 配置文件相对路径 |

#### 返回值

- 成功：`CfgFiles*` 指针（需调用 `FreeCfgFiles` 释放）
- 失败：返回 NULL

**代码证据**: `config_policy_utils.c:520-523`

---

### 4. GetCfgFilesEx

带 FollowX 模式获取所有层级的配置文件路径。

```c
CfgFiles *GetCfgFilesEx(const char *pathSuffix, int followMode, const char *extra);
```

#### 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| pathSuffix | const char* | 配置文件相对路径 |
| followMode | int | FollowX 模式 |
| extra | const char* | 用户自定义 Follow 路径 |

#### 返回值

- 成功：`CfgFiles*` 指针
- 失败：返回 NULL

**代码证据**: `config_policy_utils.c:479-518`

---

### 5. GetCfgDirList

获取配置目录列表。

```c
CfgDir *GetCfgDirList(void);
```

#### 返回值

- 成功：`CfgDir*` 指针（需调用 `FreeCfgDirList` 释放）
- 失败：返回 NULL

**代码证据**: `config_policy_utils.c:525-548`

---

### 6. FreeCfgFiles

释放 `GetCfgFiles`/`GetCfgFilesEx` 返回的内存。

```c
void FreeCfgFiles(CfgFiles *res);
```

**代码证据**: `config_policy_utils.c:408-420`

---

### 7. FreeCfgDirList

释放 `GetCfgDirList` 返回的内存。

```c
void FreeCfgDirList(CfgDir *res);
```

**代码证据**: `config_policy_utils.c:422-432`

---

## 使用示例

```cpp
#include "config_policy_utils.h"

// 示例 1: 获取单个配置文件路径
char buf[MAX_PATH_LEN] = {0};
char *path = GetOneCfgFile("etc/xml/config.xml", buf, MAX_PATH_LEN);
if (path != NULL) {
    printf("配置文件路径: %s\n", path);
}

// 示例 2: 获取所有配置文件路径
CfgFiles *files = GetCfgFiles("etc/xml/config.xml");
if (files != NULL) {
    for (size_t i = 0; i < MAX_CFG_POLICY_DIRS_CNT; i++) {
        if (files->paths[i] != NULL) {
            printf("层级 %zu: %s\n", i, files->paths[i]);
        }
    }
    FreeCfgFiles(files);
}

// 示例 3: 获取配置目录列表
CfgDir *dirs = GetCfgDirList();
if (dirs != NULL) {
    for (size_t i = 0; i < MAX_CFG_POLICY_DIRS_CNT; i++) {
        if (dirs->paths[i] != NULL) {
            printf("目录 %zu: %s\n", i, dirs->paths[i]);
        }
    }
    FreeCfgDirList(dirs);
}
```

---

## 内存管理规则

### 必须释放的 API

| API | 对应释放函数 |
|-----|--------------|
| `GetCfgFiles()` | `FreeCfgFiles()` |
| `GetCfgFilesEx()` | `FreeCfgFiles()` |
| `GetCfgDirList()` | `FreeCfgDirList()` |

### 注意事项

1. **立即释放**：获取结果后应立即释放，避免内存泄漏
2. **NULL 检查**：释放前应检查指针是否为 NULL
3. **置空指针**：释放后可将指针置为 NULL，防止二次释放

**代码证据**: `config_policy_utils.c:408-432`

---

## 平台差异

### LiteOS-M 特殊处理

```c
#ifdef __LITEOS_M__
#define MINI_CONFIG_POLICY_BUF_SIZE 256
void SetMiniConfigPolicy(const char *policy);
__WEAK void TrigSetMiniConfigPolicy();
#endif
```

LiteOS-M 平台使用静态缓冲区存储配置策略，避免动态内存碎片。

**代码证据**: `config_policy_impl.h:39-44`, `config_policy_utils.c:56-71`

---

## 相关文档

- [N-API 接口文档](./04_N-API_Reference.md)
- [架构设计](./03_Architecture.md)
