# 项目概览

## 1. 仓库信息

| 属性 | 值 |
|------|-----|
| 仓库名称 | kernel_linux_common_modules |
| 仓库用途 | Linux 内核通用模块容器仓 |
| 许可证 | GPL-2.0-or-later |

**证据来源**: `README.md:1-4`, `LICENSE:1-12`

---

## 2. 仓库定位

### 2.1 核心目标

本仓库用于**集中存放各内核领域的独立模块**，实现：

| 目标 | 说明 |
|------|------|
| 模块隔离 | 各模块独立目录，便于维护和评审 |
| 通用性 | 模块可在 OpenHarmony 支持的任意 Linux 内核版本上运行 |
| 可构建 | 基于 GN 构建系统，支持生成内核模块（.ko） |

### 2.2 适用范围

| 类型 | 说明 |
|------|------|
| **适用** | 通用内核模块、无特定平台/硬件依赖 |
| **不适用** | 特定芯片平台驱动、特定硬件驱动 |

**证据来源**: `README.md:5-6`

```
README.md:5-6
适用模块：通用内核模块，可在OpenHarmony支持的任何Linux内核版本上使用。
对特定平台或硬件等依赖的模块不适合合入该仓。
```

---

## 3. 目录结构

### 3.1 顶层目录

```
kernel/linux/common_modules/
├── LICENSE                     # 许可证配置
├── OAT.xml                     # OAT扫描配置
├── README.md                   # 项目主文档
├── README.en.md               # 英文版项目文档
├── README.OpenSource          # 开源软件引用说明
├── BUILD.gn                   # GN构建入口
├── code_sign/                 # 代码签名模块
├── container_escape_detection/ # 容器逃逸检测模块
├── memory_security/           # 内存安全模块
├── module_sample/             # 模块示例
├── newip/                     # 新IP协议模块
├── pac/                      # 指针认证模块
├── qos_auth/                 # QoS认证模块
├── tzdriver/                 # TrustZone驱动模块
├── ucollection/              # 集合操作模块
└── xpm/                      # 包管理器模块
```

### 3.2 模块标准目录结构

每个模块应遵循：

```
模块名/
├── include/          # 头文件目录
├── src/             # 源文件目录
├── third_party/     # 三方引入文件目录
│   └── LICENSES/   # 三方许可证
├── README.md        # 模块自简介
├── README_en.md     # 英文版模块简介
└── BUILD.gn         # 模块构建配置
```

**证据来源**: `README.md:12-18`

---

## 4. 模块清单

### 4.1 安全核心模块

| 模块 | 功能 | 复杂度 | 证据来源 |
|------|------|--------|----------|
| tzdriver | TrustZone 驱动、REE-TEE 通信 | 高 | `tzdriver/README_zh.md` |
| memory_security | 内存安全、KASLR/SMEP/SMAP 防护 | 高 | `memory_security/README_zh.md` |
| pac | 指针认证码、控制流保护 | 高 | `pac/README_zh.md` |
| container_escape_detection | 容器逃逸检测 | 高 | `container_escape_detection/core/` |

### 4.2 功能模块

| 模块 | 功能 | 复杂度 | 证据来源 |
|------|------|--------|----------|
| code_sign | 代码签名、证书链管理 | 中 | `code_sign/` |
| qos_auth | QoS 认证、资源调度 | 中 | `qos_auth/README_zh.md` |
| newip | 新 IP 协议栈 | 高 | `newip/README_zh.md` |
| xpm | 可执行权限管理 | 中 | `xpm/README_zh.md` |

### 4.3 基础模块

| 模块 | 功能 | 复杂度 | 证据来源 |
|------|------|--------|----------|
| ucollection | 性能采集、CPU 维测 | 低 | `ucollection/` |
| module_sample | 示例模块 | 低 | `module_sample/BUILD.gn` |

---

## 5. 构建系统

### 5.1 GN 构建配置

**入口文件**: `BUILD.gn:14-16`

```gn
group("ko_build") {
  deps = [ "module_sample:ko_sample" ]
}
```

### 5.2 ko 模块编译模板

使用 `ohos_build_ko` 模板：

```gn
import("//build/templates/kernel/ohos_kernel_build.gni")

ohos_build_ko("ko_sample") {
  sources = [ "ko_sample.c", "sample_fun.c" ]
  target_ko_name = "kosample"
  device_name = device_name
  device_arch = "arm64"
}
```

### 5.3 产物位置

| 产物类型 | 位置 |
|----------|------|
| .ko 文件 | `out/{device}/packages/phone/chip_ckm/` |
| 镜像文件 | `out/{device}/packages/phone/images/chip_ckm.img` |

**证据来源**: `README.md:99-133`

---

## 6. 许可证

### 6.1 许可证配置

所有模块使用 **GPL-2.0-or-later**：

| 目录/模块 | 许可证 |
|----------|--------|
| newip/ | GPL-2.0-or-later |
| tzdriver/ | GPL-2.0-or-later |
| xpm/ | GPL-2.0-or-later |
| qos_auth/ | GPL-2.0-or-later |
| ucollection/ | GPL-2.0-or-later |
| memory_security/ | GPL-2.0-or-later |
| code_sign | GPL-2.0-or-later |
| container_escape_detection | GPL-2.0-or-later |
| module_sample | GPL-2.0-or-later |
| pac/ | GPL-2.0-or-later |

**证据来源**: `LICENSE:1-12`

### 6.2 合入规则

| 规则 | 说明 |
|------|------|
| 正向依赖 | 模块只能正向依赖内核，不得反向依赖 |
| 编译方案 | 合入时必须提供可编译方案 |
| 平台无关 | 不得依赖特定芯片平台、产品、硬件 |
| GPL 协议 | 使用 GPL 系列协议 |

**证据来源**: `README.md:70-78`

---

## 7. 相关链接

| 链接 | 说明 |
|------|------|
| [OpenHarmony 内核 SIG](https://gitee.com/openharmony/community/blob/master/sig/sig_kernel/sig_kernel_cn.md) | 内核社区 |
| [OAT Tool](https://gitee.com/openharmony-sig/tools_oat/blob/master/README_zh.md) | 开源审视工具 |
| [ko 构建指导](README.md#ko模块指导) | 内核模块构建 |
| [NewIP 开发手册](https://gitee.com/openharmony/docs/blob/master/zh-cn/device-dev/kernel/kernel-standard-newip.md) | NewIP 协议文档 |

---

## 8. N-API 说明

**本仓库为内核模块仓库，不涉及用户态 N-API**：

| 检查项 | 结果 |
|--------|------|
| N-API 入口 | 无 |
| NAPI_MODULE 宏 | 无 |
| napi_define_properties | 无 |
| 用户态 IPC | 无 |

**结论**: 本仓库模块运行于内核态，与用户态 N-API 无直接关联。
