# 常见问题解答 (FAQ)

本文档收集开发者在使用 Hisilicon Vendor 仓库时的常见问题。

---

## 构建相关

### Q1: 如何编译特定产品？

```bash
# 编译 wifiiot 产品 (Pegasus)
python build.py --product-name wifiiot

# 编译 taurus 产品
python build.py --product-name hispark_taurus

# 编译标准系统
python build.py --product-name hispark_taurus_standard
```

### Q2: 构建失败如何排查？

1. 检查 Python 环境（需要 Python 3.8+）
2. 确认依赖安装完整
3. 查看错误日志定位问题
4. 尝试清理构建缓存：

```bash
python build.py --product-name wifiiot --build-target clean
```

### Q3: 如何添加新的子系统？

在 `config.json` 的 `subsystems` 数组中添加：

```json
{
  "subsystem": "your_subsystem",
  "components": [
    { "component": "your_component", "features": [] }
  ]
}
```

---

## 配置相关

### Q4: 如何修改 WiFi 配置？

**LiteOS-M**：`hispark_pegasus/config.json`

```json
{
  "subsystem": "communication",
  "components": [
    { "component": "wifi_lite", "features": [] }
  ]
}
```

### Q5: 如何添加预装应用？

在 `preinstall-config/install_list.json` 中添加：

```json
{
  "bundleName": "com.example.app",
  "version": {
    "code": 1,
    "name": "1.0.0"
  }
}
```

### Q6: 如何配置 HDF 驱动？

在 `hdf_config/khdf/` 目录下添加或修改 `.hcs` 文件：

```hcs
#include "device_info/device_info.hcs"

root {
    module = "your_driver_module";
}
```

---

## Demo 相关

### Q7: 如何编译运行 Demo？

```bash
# 1. 修改 build/lite/product/wifiiot.json
# 将 "//applications/sample/wifi-iot/app" 替换为 "easy_wifi:app"

# 2. 编译
python build.py wifiiot

# 3. 烧录运行
```

### Q8: WiFi Demo 如何切换 STA/AP 模式？

修改 `demo/easy_wifi_demo/demo/BUILD.gn`：

```gn
# STA 模式
sources = [
  "wifi_connect_demo.c",   # 取消注释
  # "wifi_hotspot_demo.c",   # 注释掉
]

# AP 模式
sources = [
  # "wifi_connect_demo.c",   # 注释掉
  "wifi_hotspot_demo.c",   # 取消注释
]
```

---

## 硬件相关

### Q9: 支持哪些开发板？

请参考 [产品系列](../03_Products.md) 文档。

### Q10: 如何添加新硬件支持？

1. 在 `hals/` 目录下添加 HAL 代码
2. 在 `hdf_config/` 目录下添加驱动配置
3. 在 `config.json` 中注册子系统组件

---

## 系统相关

### Q11: 各系统类型的区别？

| 系统 | RAM 要求 | 特点 |
|-----|---------|------|
| LiteOS-M | < 64KB | 无 MMU，超低功耗 |
| LiteOS | < 1MB | 轻量设备 |
| 标准系统 | > 128MB | 完整功能 |
| Linux | > 256MB | 通用 Linux |

### Q12: 如何选择合适的系统？

根据资源限制和功能需求选择：
- 资源受限 → LiteOS-M
- 中等资源 → LiteOS
- 功能完整 → 标准系统
- 需要 Linux → Linux

---

## 开发相关

### Q13: 如何调试 Demo？

1. 串口连接开发板
2. 查看串口日志输出
3. Demo 中使用 `printf()` 打印调试信息

### Q14: 如何添加日志？

```c
#include "stdio.h"

void demo_function(void) {
    printf("Demo: debug message\r\n");
}
```

---

## 相关文档

- [项目概述](../01_Overview.md)
- [目录结构](../02_Directory_Structure.md)
- [产品系列](../03_Products.md)
- [配置体系](../04_Configuration.md)
- [Demo 示例](../05_Demos.md)
- [构建指南](../06_Build.md)
- [安全指南](../07_Security.md)
