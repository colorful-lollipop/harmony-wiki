# 内部模块与 API

## 模块概览

safwk_lite 的内部结构非常简洁，核心职责由 Samgr Lite 框架承担：

```
┌─────────────────────────────────────┐
│         safwk_lite (foundation)      │
├─────────────────────────────────────┤
│  main.c                             │
│  ├── OHOS_SystemInit() [弱符号]     │
│  └── main()                         │
├─────────────────────────────────────┤
│  Samgr Lite (外部依赖)              │
│  ├── SAMGR_Bootstrap()              │
│  ├── 服务注册接口                    │
│  └── IPC 路由                       │
└─────────────────────────────────────┘
```

## 核心入口点

### OHOS_SystemInit()

**文件**: `src/main.c:30-36`

```c
void __attribute__((weak)) OHOS_SystemInit(void)
{
    SAMGR_Bootstrap();
}
```

| 属性 | 值 |
|------|-----|
| 类型 | 弱符号函数 |
| 目的 | 提供可覆盖的初始化入口 |
| 默认行为 | 调用 SAMGR_Bootstrap() |

### main()

**文件**: `src/main.c:38-62`

```c
int main(int argc, char * const argv[])
{
    OHOS_SystemInit();

    while (1) {
        (void)pause();  // 无限循环等待信号
    }
}
```

| 属性 | 值 |
|------|-----|
| 目的 | 进程入口函数 |
| 行为 | 初始化后进入无限阻塞 |

## Samgr Lite 接口

### SAMGR_Bootstrap()

**声明位置**: `//foundation/systemabilitymgr/samgr_lite/interfaces/kits/samgr/samgr_lite.h`

**功能**: 初始化 Samgr 框架

```c
void SAMGR_Bootstrap(void);
```

**依赖来源**: 通过 `BUILD.gn` 引入

```gn
deps = [
  "//foundation/systemabilitymgr/samgr_lite/samgr_server:server",
]
```

## 依赖方向

### 入向依赖 (safwk_lite 依赖其他)

| 组件 | 依赖类型 | 说明 |
|------|---------|------|
| samgr_lite | 强依赖 | 系统能力管理框架 |
| hilog_lite | 强依赖 | 日志框架 |
| ability_lite | 可选 | 能力管理服务 |
| bundle_framework_lite | 可选 | 包管理服务 |
| dmsfwk_lite | 可选 | 分布式调度服务 |

### 出向依赖 (被其他依赖)

`safwk_lite` 作为基础进程，被以下组件构建依赖：

```gn
deps = [
  "//foundation/systemabilitymgr/safwk_lite:foundation",
]
```

## 稳定性标注

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| `OHOS_SystemInit()` | **不稳定** | 弱符号设计，可能被覆盖 |
| `SAMGR_Bootstrap()` | 稳定 | Samgr Lite 官方接口 |
| `main()` | 稳定 | 标准 C 入口 |

## 可替换点

| 替换点 | 替换方式 | 影响 |
|--------|----------|------|
| `OHOS_SystemInit()` | 定义同名强符号函数 | 自定义初始化逻辑 |
| feature 开关 | 修改 BUILD.gn | 控制包含哪些子服务 |
