# 目录结构

## 源码目录

```
safwk_lite/
├── BUILD.gn                    # GN 构建入口
├── README.md                   # 英文说明
├── README_zh.md                # 中文说明
├── LICENSE                    # Apache 2.0 许可证
├── OAT.xml                    # 许可证检查配置
├── bundle.json               # 子系统配置
├── figures/                  # 架构图目录
│   ├── en-us_image_0000001128146921.png
│   ├── zh-cn_image_0000001128146921.png
│   ├── en-us_image_0000001081285004.png
│   └── zh-cn_image_0000001081285004.png
└── src/                      # 源码目录
    └── main.c                # 进程入口
```

## 模块职责

| 目录/文件 | 职责 |
|----------|------|
| `src/main.c` | 进程入口，调用 `SAMGR_Bootstrap()` 后进入无限循环 |
| `BUILD.gn` | 定义 `foundation` 可执行文件和 feature 开关 |
| `bundle.json` | 子系统元数据配置 |
| `figures/` | 架构示意图 |

## src/ 目录详解

```
src/
└── main.c (69 行)
    ├── OHOS_SystemInit() - 弱符号函数，默认调用 SAMGR_Bootstrap()
    └── main() - 入口函数，调用 OHOS_SystemInit() 后 pause()
```

## 关键代码证据

**文件**: `src/main.c:30-36`

```c
void __attribute__((weak)) OHOS_SystemInit(void)
{
    SAMGR_Bootstrap();
#ifdef DEBUG_SERVICES_SAFWK_LITE
    printf("[Foundation][D] Default OHOS_SystemInit is called! \n");
#endif
}
```

**说明**: 使用 GCC 弱符号机制，允许其他模块覆盖 `OHOS_SystemInit()` 实现。

## 排除的目录

以下目录/文件不包含在 Wiki 分析范围内：

- `test/` - 测试代码
- `unittest/` - 单元测试
- `*_test.*` - 测试文件
- `.git/` - Git 版本控制
- `.gitee/` - Gitee 配置
