# 原始库简介

## 1.1 库基本信息

### 1.1.1 Khronos Group 与 OpenGL ES

**Khronos Group** 是一个非营利性技术联盟，由包括苹果、谷歌、英特尔、英伟达、AMD 等在内的多家科技公司共同创立。Khronos 负责制定开放图形标准，包括 OpenGL、OpenGL ES、Vulkan、OpenCL、WebGL 等。

**OpenGL ES**（OpenGL for Embedded Systems）是 OpenGL 的嵌入式精简版本，专为移动设备、游戏主机、嵌入式系统等资源受限环境设计。OpenGL ES 移除了桌面 OpenGL 中许多冗余和复杂的功能，同时保持核心图形渲染能力。

### 1.1.2 OpenGL-Registry 仓库

**OpenGL-Registry**（https://github.com/KhronosGroup/OpenGL-Registry）是 Khronos Group 官方维护的 API 注册表仓库，包含以下内容：

| 类别 | 说明 |
|------|------|
| **API 规范** | OpenGL、OpenGL ES、OpenGL SC 的核心 API 定义 |
| **GLSL 规范** | 着色语言（OpenGL Shading Language）规范 |
| **扩展规范** | Khronos 和供应商批准的扩展定义 |
| **头文件** | 与规范对应的 C 语言头文件 |
| **XML 注册表** | 结构化的 API 和枚举值定义 |
| **工具脚本** | 头文件生成、注册表维护等工具 |

### 1.1.3 版本信息

| 项目 | 内容 |
|------|------|
| **当前版本** | 31db35a53f7e64d3e03e35ec47ccfd6079b792d7 |
| **许可证** | Apache-2.0 |
| **上游地址** | https://github.com/KhronosGroup/OpenGL-Registry |
| **维护者** | Khronos Group |

## 1.2 库功能详解

### 1.2.1 头文件结构

```
api/
├── GL/                      # 桌面 OpenGL 头文件
│   ├── gl.h                # OpenGL 1.x 核心头文件
│   ├── glext.h             # OpenGL 扩展头文件
│   ├── glcorearb.h         # OpenGL 核心规范头文件
│   ├── wgl.h               # Windows OpenGL 扩展
│   └── wglext.h            # Windows OpenGL 扩展
├── GLES/                   # OpenGL ES 1.x 头文件
│   ├── gl.h                # OpenGL ES 1.x 核心头文件
│   ├── glplatform.h        # 平台相关定义
│   └── glext.h             # OpenGL ES 扩展头文件
├── GLES2/                  # OpenGL ES 2.x 头文件
│   ├── gl2.h               # OpenGL ES 2.x 核心头文件
│   ├── gl2platform.h      # 平台相关定义
│   └── gl2ext.h            # OpenGL ES 2.x 扩展头文件
├── GLES3/                  # OpenGL ES 3.x 头文件
│   ├── gl3.h               # OpenGL ES 3.x 核心头文件
│   ├── gl3platform.h      # 平台相关定义
│   ├── gl31.h             # OpenGL ES 3.1 头文件
│   ├── gl32.h             # OpenGL ES 3.2 头文件
│   └── glext.h            # OpenGL ES 3.x 扩展头文件
└── GLSC/                  # OpenGL SC（安全关键）头文件
    ├── gl.h               # OpenGL SC 1.x
    └── glsc2.h            # OpenGL SC 2.x
```

### 1.2.2 扩展分类

该库包含 50+ 供应商的扩展规范，按前缀分类：

| 扩展前缀 | 说明 | 示例数量 |
|---------|------|---------|
| **ARB** | OpenGL ARB（Architecture Review Board）批准的扩展 | 186 |
| **EXT** | 多供应商扩展（Extension） | 228 |
| **OES** | OpenGL ES 专用扩展 | 71 |
| **NV** | NVIDIA 供应商扩展 | 172 |
| **AMD** | AMD 供应商扩展 | 44 |
| **INTEL** | Intel 供应商扩展 | 12 |
| **IMG** | Imagination Technologies 供应商扩展 | 15 |
| **QCOM** | Qualcomm 供应商扩展 | 24 |
| **ARM** | ARM 供应商扩展 | 9 |
| **HUAWEI** | 华为供应商扩展 | 4 |
| **MESA** | Mesa3D 开源驱动扩展 | 22 |
| **APPLE** | Apple 供应商扩展 | 27 |
| **ANGLE** | ANGLE 项目扩展 | 9 |
| **其他** | 各类供应商特定扩展 | 50+ |

### 1.2.3 API 版本演进

| 版本 | 发布年份 | 主要特性 |
|------|---------|---------|
| **OpenGL ES 1.0** | 2003 | 固定功能管线（Fixed-Function Pipeline） |
| **OpenGL ES 1.1** | 2004 | 改进的固定功能渲染，多重采样改进 |
| **OpenGL ES 2.0** | 2007 | 可编程着色器（Shader），移除固定管线 |
| **OpenGL ES 3.0** | 2012 | 纹理压缩、变换反馈、Instanced Rendering |
| **OpenGL ES 3.1** | 2014 | 计算着色器（Compute Shader）、高级纹理 |
| **OpenGL ES 3.2** | 2015 | 几何着色器（Geometry Shader）、Tessellation |

## 1.3 在 OpenHarmony 中的作用

### 1.3.1 定位与角色

该库在 OpenHarmony 系统中扮演**图形 API 定义层**的角色：

```
┌─────────────────────────────────────────────────────────────┐
│                    应用层 (Applications)                     │
│            OpenGL ES 游戏、3D 渲染引擎、XComponent            │
├─────────────────────────────────────────────────────────────┤
│                    NDK 层 (NDK API)                          │
│              GLES2/GLES3 NDK 头文件和库                      │
├─────────────────────────────────────────────────────────────┤
│                  OpenGL Wrapper 层                           │
│    libEGL.so、libGLESv1.so、libGLESv2.so、libGLESv3.so       │
│              (foundation/graphic/.../opengl_wrapper)          │
├─────────────────────────────────────────────────────────────┤
│              OpenGL 头文件注册表 (本文档)                     │
│                 third_party/openGLES/api/                    │
│              API 定义、类型定义、函数签名                     │
├─────────────────────────────────────────────────────────────┤
│                  GPU 驱动层 (GPU Driver)                    │
│                厂商特定的 OpenGL 实现                        │
└─────────────────────────────────────────────────────────────┘
```

### 1.3.2 核心价值

| 价值维度 | 说明 |
|---------|------|
| **标准化** | 提供 Khronos 官方标准 API 定义，确保跨平台兼容性 |
| **扩展支持** | 提供 200+ 扩展规范，支持 GPU 厂商特定功能 |
| **版本兼容** | 支持 OpenGL ES 1.x、2.x、3.x 全部版本 |
| **生态连接** | 连接 OpenHarmony 图形栈与 Khronos 标准生态 |

### 1.3.3 与上游的关系

| 维度 | 描述 |
|------|------|
| **同步方式** | Git submodule 或手动同步上游 commit |
| **变更频率** | 按需同步，关注 Breaking Changes |
| **兼容性** | 严格保持向后兼容 |
| **扩展添加** | 按需添加厂商特定扩展 |

## 1.4 技术架构

### 1.4.1 XML 注册表设计

OpenGL-Registry 使用 XML 文件（`xml/gl.xml`）作为 API 注册表的权威数据源：

```xml
<feature api="gles" version="3.0">
    <require>
        <enum name="GL_MAJOR_VERSION" value="0x821B"/>
        <enum name="GL_MINOR_VERSION" value="0x821C"/>
        <command name="glReadPixels"/>
        <command name="glDrawElements"/>
    </require>
</feature>
```

### 1.4.2 头文件生成流程

```
xml/gl.xml (XML 注册表)
       ↓
  genheaders.py (Python 脚本)
       ↓
api/*.h (C 语言头文件)
```

### 1.4.3 扩展注册机制

扩展采用分层注册机制：

| 层级 | 说明 | 示例 |
|------|------|------|
| **预发布扩展** | 供应商实现的实验性扩展 | GL_OES_depth_texture |
| **ARB 扩展** | 经过 ARB 审核的扩展 | GL_ARB_vertex_shader |
| **核心特性** | 融入标准版本的特性 | OpenGL ES 3.0 的 transform feedback |

## 1.5 许可证信息

### 1.5.1 Apache-2.0 许可证

本库采用 Apache-2.0 许可证，该许可证具有以下特点：

| 特点 | 说明 |
|------|------|
| **商业友好** | 可在商业产品中使用 |
| **修改自由** | 可修改源代码 |
| **专利授权** | 包含明确的专利授权条款 |
| **衍生作品** | 需保留版权声明和许可证文本 |

### 1.5.2 许可证文件

| 文件 | 路径 |
|------|------|
| **Apache-2.0 许可证** | LICENSE |
| **行为准则** | CODE_OF_CONDUCT.adoc |

## 1.6 与其他库的关系

### 1.6.1 依赖关系

| 上游库 | 关系 | 说明 |
|-------|------|------|
| **EGL Registry** | 互补 | 提供 EGL API 定义（独立仓库） |
| **Vulkan Registry** | 互补 | 提供 Vulkan API 定义（独立仓库） |
| **ANGLE** | 使用方 | 使用本库头文件实现 GL 到 Vulkan/Metal 的转换 |

### 1.6.2 生态系统位置

```
Khronos 标准生态
    ├── OpenGL Registry (本文档) ──→ OpenGL/GLES/SC 头文件
    ├── EGL Registry ──→ EGL 头文件
    ├── Vulkan Registry ──→ Vulkan 头文件
    ├── OpenCL Registry ──→ OpenCL 头文件
    └── WebGL Registry ──→ WebGL 头文件

OpenHarmony 图形栈
    ├── third_party/openGLES (本文档) ──→ API 定义
    ├── third_party/egl ──→ EGL 头文件
    ├── third_party/angle2 ──→ ANGLE 实现
    └── foundation/graphic/.../opengl_wrapper ──→ OpenGL Wrapper
```
