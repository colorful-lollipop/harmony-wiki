# 01_原始库概述

## 1.1 库基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | QR-Code-generator |
| **上游地址** | https://www.nayuki.io/page/qr-code-generator-library |
| **上游版本** | 1.8.0 |
| **许可证** | MIT License |
| **作者** | Project Nayuki |
| **OH 组件版本** | 3.1 |

## 1.2 原始功能简介

QR-Code-generator 是一个高性能、零依赖的 QR 码生成库，完全符合 ISO/IEC 18004:2015 QR Code Model 2 标准。

### 核心能力

- **多版本支持**：QR Code 版本 1-40（尺寸 21x21 到 177x177 模块）
- **四种错误纠正级别**：L（~7%）、M（~15%）、Q（~25%）、H（~30%）
- **四种字符编码模式**：Numeric、Alphanumeric、Byte、Kanji
- **自动优化**：自动选择最优版本、掩码和编码模式
- **零依赖**：仅使用 C++ 标准库
- **多语言实现**：C++、Java、Python、Rust、TypeScript、Go、C 等

### 典型使用方式（上游原版）

```cpp
#include "qrcodegen.hpp"

using qrcodegen::QrCode;
using qrcodegen::QrSegment;

// 简单使用：文本转 QR 码
QrCode qr = QrCode::encodeText("Hello, World!", QrCode::Ecc::MEDIUM);

// 高级使用：自定义段编码
std::vector<QrSegment> segs = QrSegment::makeSegments(text);
QrCode qr = QrCode::encodeSegments(segs, QrCode::Ecc::HIGH, 1, 40, -1, true);

// 访问生成的模块
for (int y = 0; y < qr.getSize(); y++) {
    for (int x = 0; x < qr.getSize(); x++) {
        bool dark = qr.getModule(x, y);
        // dark == true 表示黑色模块
    }
}
```

## 1.3 目录结构

```
qrcodegen/
├── cpp/                          # C++ 实现（OpenHarmony 使用）
│   ├── qrcodegen.hpp             # 主头文件
│   ├── qrcodegen.cpp             # 实现文件
│   └── QrCodeGeneratorDemo.cpp   # 示例程序
├── c/                            # C 语言实现
├── java/                         # Java 实现
├── java-fast/                    # 高速 Java 实现
├── python/                       # Python 实现
├── rust/                         # Rust 实现
├── rust-no-heap/                 # 无堆分配 Rust 实现
├── typescript-javascript/        # TypeScript/JavaScript 实现
├── BUILD.gn                      # OpenHarmony 构建配置
├── CMakeLists.txt                # CMake 构建配置
├── README.OpenSource             # 开源声明
└── bundle.json                   # OH 组件配置
```

## 1.4 核心类说明

### QrCode 类

QR 码生成的主类，提供以下主要接口：

| 方法 | 功能 |
|------|------|
| `encodeText(const char*, Ecc)` | 从文本生成 QR 码 |
| `encodeSegments(const std::vector<QrSegment>&, Ecc, ...)` | 从段列表生成 QR 码 |
| `getVersion()` | 获取 QR 码版本号（1-40） |
| `getSize()` | 获取 QR 码尺寸（21-177） |
| `getErrorCorrectionLevel()` | 获取错误纠正级别 |
| `getMask()` | 获取使用的掩码编号（0-7） |
| `getModule(int x, int y)` | 获取指定位置的模块颜色 |

### QrSegment 类

数据段类，用于构建复杂的 QR 码数据：

| 方法 | 功能 |
|------|------|
| `makeSegments(const char*)` | 自动选择最优编码模式 |
| `makeBytes(const std::vector<uint8_t>&)` | 字节模式编码 |
| `makeNumeric(const char*)` | 数字模式编码 |
| `makeAlphanumeric(const char*)` | 字母数字模式编码 |
| `makeEci(long)` | ECI 通道编码 |

### 错误处理（原版）

上游版本使用 C++ 异常机制：

```cpp
try {
    QrCode qr = QrCode::encodeText(text, QrCode::Ecc::LOW);
} catch (const data_too_long& e) {
    // 数据过长，无法在 QR 码版本限制内编码
}
```

## 1.5 在 OpenHarmony 中的定位

### 作用

qrcodegen 是 **OpenHarmony Ace Engine UI 框架**的核心依赖，为 QRCode 组件提供二维码图像生成能力。

### 依赖链

```
应用层
   ↓ 使用
QRCode 组件 (ArkUI)
   ↓ 依赖
ace_engine
   ↓ 依赖
qrcodegen (third_party)
```

### 为什么选择 qrcodegen

1. **零依赖**：不引入额外依赖，符合 OH 轻量化原则
2. **标准兼容**：完全符合 ISO/IEC 18004 标准
3. **高性能**：纯计算实现，无需额外内存分配
4. **多端支持**：同时支持 standard、small、mini 系统类型

## 1.6 版本对应关系

| OH 版本 | qrcodegen OH 版本 | 上游版本 | 备注 |
|---------|-------------------|----------|------|
| OpenHarmony 3.1 | 3.1 | 1.8.0 | 初始集成 |
| OpenHarmony 4.0 | 3.1 | 1.8.0 | 保持稳定 |
| 后续版本 | 待定 | 待定 | 需同步更新 |

## 1.7 参考资源

- **上游文档**：https://www.nayuki.io/page/qr-code-generator-library
- **GitHub 仓库**：https://github.com/nayuki/QR-Code-generator
- **ISO/IEC 18004 标准**：QR Code 码国际标准
