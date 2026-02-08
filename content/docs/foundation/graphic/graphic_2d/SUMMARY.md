# graphic_2d Wiki 导航

## 文档概览

```
graphic_2d Wiki
├── README.md                    # 文档说明、贡献指南
├── SUMMARY.md                   # 本导航文件
├── 01_Overview.md              # 项目定位、边界、核心能力
├── 02_Directory_Structure.md    # 目录结构与模块职责
├── 03_Architecture.md           # 架构说明（组件图、数据流、线程模型）
├── 04_N-API.md                  # 对外 N-API 接口
├── 05_Inner_API.md              # 内部模块接口
├── 06_Build.md                  # GN Targets 与编译产物
├── 07_Security.md               # 安全风险评审
├── 08_FAQ.md                    # 常见问题
└── appendix/
    ├── Callgraphs.md            # 关键调用链
    └── Config_Flags.md          # 关键宏/Feature Flags
```

---

## 新人阅读路线

### 路线 A：应用开发者
1. `01_Overview.md` → 了解 graphic_2d 是什么
2. `04_N-API.md` → 查找可用的 JS API
3. `08_FAQ.md` → 常见问题

### 路线 B：Native 框架开发者
1. `01_Overview.md` → 项目定位
2. `03_Architecture.md` → 架构设计
3. `05_Inner_API.md` → 模块接口
4. `06_Build.md` → 构建配置

### 路线 C：系统集成/移植
1. `01_Overview.md` → 依赖关系
2. `06_Build.md` → GN 构建
3. `03_Architecture.md` → 线程模型
4. `07_Security.md` → 安全考量

### 路线 D：安全审计
1. `07_Security.md` → 安全风险评审
2. `04_N-API.md` → N-API 攻击面
3. `03_Architecture.md` → 信任边界

### 路线 E：深入理解
1. `appendix/Callgraphs.md` → 关键调用链
2. `appendix/Config_Flags.md` → 配置开关

---

## 模块速查

| 模块 | 路径 | 职责 | 文档章节 |
|------|------|------|----------|
| Render Service | `rosen/modules/render_service/` | 渲染服务器 | 03, 05 |
| Render Service Base | `rosen/modules/render_service_base/` | IPC 接口定义 | 03, 05 |
| Render Service Client | `rosen/modules/render_service_client/` | 客户端 API | 04, 05 |
| 2D Graphics | `rosen/modules/2d_graphics/` | Canvas/Paint/Path | 04, 05 |
| Effect | `rosen/modules/effect/` | 图像效果 | 04, 05 |
| Composer | `rosen/modules/composer/` | 显示合成 | 03, 05 |
| N-API | `interfaces/kits/napi/` | JS 接口绑定 | 04 |

---

## API 速查

### 2D 绘图 N-API
- `drawing` - Canvas, Paint, Path, Bitmap, Matrix
- 位置: `interfaces/kits/napi/graphic/drawing/`

### 颜色管理 N-API
- `colorManager` - ColorSpace, ColorSpaceManager
- 位置: `interfaces/kits/napi/graphic/color_manager/`

### 动画 N-API
- `windowAnimation` - 窗口动画控制器
- 位置: `interfaces/kits/napi/graphic/animation/`

### WebGL N-API
- `webgl` - WebGL 1.0/2.0 API
- 位置: `interfaces/kits/napi/graphic/webgl/`

### UI 效果 N-API
- `uieffect` - Filter, VisualEffect, Mask
- 位置: `interfaces/kits/napi/graphic/ui_effect/`

### 效果工具 N-API
- `effectKit` - ColorPicker, Filter
- 位置: `interfaces/kits/napi/graphic/effect_kit/`

---

## 版本信息

- **当前版本**: v3.1 (bundle.json)
- **最后更新**: 2026-02-06
- **维护团队**: OpenHarmony Graphic Team
