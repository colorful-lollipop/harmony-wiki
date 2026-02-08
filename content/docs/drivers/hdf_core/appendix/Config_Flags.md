# HDF Core 配置项与宏定义

## 1. 编译配置

### 1.1 GN 参数 (uhdf.gni)

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| hdf_core_default_hicollie_config | bool | false | 默认 hicollie 配置 |
| hicollie_enabled | bool | false | 启用 hicollie 看门狗 |
| with_sample | bool | false | 包含示例驱动 |
| hdf_core_default_peripheral_config | bool | true | 包含外设配置 |

### 1.2 Feature 开关 (bundle.json)

| Feature | 说明 |
|---------|------|
| hdf_core_khdf_test_support | KHDF 测试支持 |
| hdf_core_platform_test_support | 平台测试支持 |
| hdf_core_platform_rtc_test_support | RTC 测试支持 |
| hdf_core_default_peripheral_config | 默认外设配置 |
| hdf_core_default_hicollie_config | 默认 hicollie 配置 |

## 2. 编译宏定义

### 2.1 系统类型宏

| 宏 | 说明 | 定义位置 |
|----|------|----------|
| `__USER__` | 用户态构建 | BUILD.gn defines |
| `__OHOS_USER__` | OpenHarmony 用户态 | 自动定义 |
| `__OHOS_STANDARD_SYS__` | 标准系统 | 自动定义 |
| `__LITEOS__` | LiteOS 系统 | 自动定义 |
| `__LINUX__` | Linux 系统 | 自动定义 |

### 2.2 架构宏

| 宏 | 说明 |
|----|------|
| `__ARCH64__` | 64 位架构 |
| `__ARM64__` | ARM64 架构 |
| `__X86_64__` | x86_64 架构 |
| `__RISCV__` | RISC-V 架构 |

### 2.3 功能宏

| 宏 | 说明 | 默认值 |
|----|------|--------|
| `WITH_SELINUX` | 启用 SELinux | 根据构建配置 |
| `HDFHICOLLIE_ENABLE` | 启用 hicollie | false |
| `LOSCFG_DRIVERS_HDF` | 启用 HDF | true |
| `LOSCFG_DRIVERS_HDF_CONFIG_MACRO` | 使用宏配置 | false |

## 3. 配置常量

### 3.1 设备类 (hdf_device_class.h)

```c
enum DeviceClass {
    DEVICE_CLASS_DEFAULT = 0,
    DEVICE_CLASS_AUDIO,
    DEVICE_CLASS_VIDEO,
    DEVICE_CLASS_CAMERA,
    DEVICE_CLASS_DISPLAY,
    DEVICE_CLASS_INPUT,
    DEVICE_CLASS_NETWORK,
    DEVICE_CLASS_SENSOR,
    DEVICE_CLASS_STORAGE,
    DEVICE_CLASS_MISC,
    DEVICE_CLASS_UNKNOWN,
};
```

### 3.2 服务策略 (hdf_device_desc.h)

```c
typedef enum {
    SERVICE_POLICY_NONE = 0,      // 不对外提供服务
    SERVICE_POLICY_PUBLIC,        // 对内核应用提供服务
    SERVICE_POLICY_CAPACITY,      // 对内核和用户应用提供服务
    SERVICE_POLICY_FRIENDLY,      // 不对外但可被订阅
    SERVICE_POLICY_PRIVATE,       // 仅内部使用
    SERVICE_POLICY_INVALID,
} ServicePolicy;
```

### 3.3 设备加载策略 (hdf_device_desc.h)

```c
typedef enum {
    DEVICE_PRELOAD_ENABLE = 0,       // 系统启动时加载
    DEVICE_PRELOAD_ENABLE_STEP2,     // 快速启动后加载
    DEVICE_PRELOAD_DISABLE,          // 不自动加载
    DEVICE_PRELOAD_INVALID,
} DevicePreload;
```

### 3.4 错误码 (hdf_base.h)

```c
#define HDF_SUCCESS              0
#define HDF_FAILURE             (-1)
#define HDF_ERR_INVALID_PARAM   (-2)
#define HDF_ERR_MALLOC_FAIL     (-3)
#define HDF_ERR_TIMEOUT         (-4)
#define HDF_ERR_NOT_SUPPORT     (-5)
#define HDF_ERR_NOT_IMPLEMENT   (-6)
#define HDF_ERR_INVALID_OBJECT  (-7)
#define HDF_ERR_NOPERM          (-8)
#define HDF_ERR_IO              (-9)
#define HDF_ERR_BAD_FD          (-10)
#define HDF_ERR_NODATA          (-11)
#define HDF_ERR_NOMEM           (-12)
#define HDF_ERR_BUSY            (-13)
#define HDF_ERR_NOT_FOUND       (-14)

// 设备错误码
#define HDF_DEV_ERR_NO_DEVICE           (-100)
#define HDF_DEV_ERR_NO_DEVICE_SERVICE   (-101)
#define HDF_DEV_ERR_NO_SPACE            (-102)
#define HDF_DEV_ERR_NO_RESOURCE         (-103)
```

## 4. 系统能力 ID

| SA ID | 名称 | 说明 |
|-------|------|------|
| 5100 | DEVICE_SERVICE_MANAGER_SA_ID | 设备服务管理器 |

## 5. 接口描述符

| 描述符 | 说明 |
|--------|------|
| "HDI.IServiceManager.V1_0" | 服务管理器接口 |
| "HDI.IDeviceManager.V1_0" | 设备管理器接口 |

## 6. 路径常量

| 常量 | 默认值 | 说明 |
|------|--------|------|
| HDF_MODULE_DIR | "/vendor/lib/modules" | 内核模块路径 |
| HDF_SECURE_PATH | "/dev/hdf_secure" | 安全设备路径 |

## 7. 限制值

| 常量 | 值 | 说明 |
|------|-----|------|
| MAX_PRIORITY_NUM | 200 | 最大优先级 |
| ID_MAX_SIZE | 64 | ID 最大长度 |
| PATH_MAX | 4096 | 路径最大长度 |
| MAX_SERVICE_NAME_LEN | 64 | 服务名最大长度 |
| MAX_DEVICE_NAME_LEN | 64 | 设备名最大长度 |

## 8. 安全相关常量

### 8.1 权限类型 (hdf_security.h)

```c
typedef enum {
    PAL_I2C_TYPE = 0,
    PAL_SPI_TYPE,
    PAL_GPIO_TYPE,
    PAL_PINCTRL_TYPE,
    PAL_CLOCK_TYPE,
    PAL_REGULATOR_TYPE,
    PAL_MIPI_TYPE,
    PAL_UART_TYPE,
    PAL_SDIO_TYPE,
    PAL_MDIO_TYPE,
    PAL_APB_TYPE,
    PAL_PCIE_TYPE,
    PAL_PCM_TYPE,
    PAL_I2S_TYPE,
    PAL_PWM_TYPE,
    PAL_DMA_TYPE,
    PAL_EFUSE_TYPE,
    PAL_FLASH_TYPE,
    PAL_EMMC_TYPE,
    PAL_RTC_TYPE,
    PAL_ADC_TYPE,
    PAL_WDT_TYPE,
    PAL_I3C_TYPE,
    PAL_MAX_TYPE,
} PalType;
```

### 8.2 IOCTL 命令 (hdf_security.h)

```c
#define HDF_SECURE_SET_INFO      _IOW('H', 0, struct SecInfo)
#define HDF_SECURE_SET_CURRENT_ID _IOW('H', 1, struct SecInfo)
#define HDF_SECURE_DELETE_INFO   _IOW('H', 2, struct SecInfo)
```

## 9. 调试相关

### 9.1 日志标签

| 标签 | 说明 |
|------|------|
| HDF_LOG_TAG | 通用 HDF 日志 |
| devsvc_manager_stub | 服务管理器 Stub |
| devmgr_service_stub | 设备管理器 Stub |
| hdf_security | 安全模块 |
| hdf_ipc | IPC 模块 |

### 9.2 日志级别

```c
#define HDF_LOG_LEVEL_ERROR   0
#define HDF_LOG_LEVEL_WARN    1
#define HDF_LOG_LEVEL_INFO    2
#define HDF_LOG_LEVEL_DEBUG   3
```

## 10. HCS 配置节点

### 10.1 常用节点名

| 节点名 | 说明 |
|--------|------|
| host | 主机配置 |
| device | 设备配置 |
| driver | 驱动配置 |
| permission | 权限配置 |
| module | 模块配置 |

### 10.2 常用属性名

| 属性名 | 类型 | 说明 |
|--------|------|------|
| moduleName | string | 模块名称 |
| serviceName | string | 服务名称 |
| deviceMatchAttr | string | 设备匹配属性 |
| preload | uint32 | 加载策略 |
| priority | uint32 | 优先级 |
| permission | uint16 | 权限值 |

## 11. 相关文档

- [GN 构建](./06_GN_Build.md)
- [安全风险](./07_Security.md)
- [HDI 接口](./04_HDI_API.md)
