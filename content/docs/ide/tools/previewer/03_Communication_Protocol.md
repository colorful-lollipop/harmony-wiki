# 通信协议

## 概述

Previewer 与 DevEco Studio 通过双通道通信：

| 通道 | 协议 | 用途 | 方向 |
|------|------|------|------|
| **命令通道** | 命名管道 | JSON 命令传输 | 双向 |
| **图像通道** | WebSocket | JPEG 图像流 | Previewer → IDE |

## 命名管道协议

### 管道名称

```
Windows:  \\.\pipe\{baseName}_commandPipe
Unix:     /tmp/.{baseName}_commandPipe
```

**证据**: `util/LocalSocket.h:46-56`

### 消息格式

#### 请求 (IDE → Previewer)

```json
{
  "version": "1.0.1",
  "type": "set|get|action",
  "command": "<commandName>",
  "args": { /* ... */ }
}
```

#### 响应 (Previewer → IDE)

```json
{
  "version": "1.0.1",
  "command": "<commandName>",
  "result": <value>
}
```

### 命令类型

| 类型 | 含义 | 处理方法 |
|------|------|----------|
| `set` | 设置参数 | `RunSet()` |
| `get` | 查询状态 | `RunGet()` |
| `action` | 执行操作 | `RunAction()` |

**证据**: `cli/CommandLine.h` - `CommandType` 枚举

## WebSocket 协议

### 图像格式

```
[LWS_PRE] + [40字节头] + [JPEG数据]
```

#### 头结构 (40 字节)

| 偏移 | 大小 | 含义 |
|------|------|------|
| 0 | 4 | Magic: 0x12345678 |
| 4 | 4 | 保留 |
| 8 | 4 | 保留 |
| 12 | 4 | 保留 |
| 16 | 4 | 保留 |
| 20 | 4 | 图像宽度 |
| 24 | 4 | 图像高度 |
| 28 | 4 | JPEG 数据大小 |
| 32 | 8 | 时间戳 |

**证据**: `mock/VirtualScreen.h` - `headSize = 40`

### JPEG 质量策略

| 分辨率等级 | 像素阈值 | 质量 |
|-----------|----------|------|
| LOW | < 100,000 | 100% |
| MIDDLE | 100,000-300,000 | 90% |
| HIGH | 300,000-500,000 | 85% |
| DEFAULT | > 500,000 | 75% |

**证据**: `mock/VirtualScreen.h` - `JpgPixCountLevel`, `JpgQualityLevel`

## 命令清单

### 通用命令

| 命令 | 类型 | 描述 |
|------|------|------|
| `MousePress` | ACTION | 鼠标按下 |
| `MouseRelease` | ACTION | 鼠标释放 |
| `MouseMove` | ACTION | 鼠标移动 |
| `PointEvent` | ACTION | 点事件 |
| `Language` | SET/GET | 语言设置 |
| `Resolution` | SET | 分辨率 |
| `exit` | ACTION | 退出 |

### Rich 专用命令

| 命令 | 类型 | 描述 |
|------|------|------|
| `BackClicked` | ACTION | 返回点击 |
| `inspector` | ACTION | 组件检查器 |
| `ColorMode` | SET | 颜色模式 |
| `Orientation` | SET | 屏幕方向 |
| `ResolutionSwitch` | SET | 分辨率切换 |
| `FoldStatus` | SET | 折叠状态 |
| `LoadDocument` | SET | 加载文档 |

### Lite 专用命令

| 命令 | 类型 | 描述 |
|------|------|------|
| `Power` | SET/GET | 电源状态 |
| `Brightness` | SET/GET | 亮度 |
| `Barometer` | SET/GET | 气压 |
| `Location` | SET/GET | 位置 |
| `HeartRate` | SET/GET | 心率 |
| `StepCount` | SET/GET | 步数 |

**完整命令清单**: `cli/CommandLineFactory.cpp`

## 参数规范

### 坐标范围

| 参数 | 最小值 | 最大值 |
|------|--------|--------|
| X/Y 坐标 | 0 | 宽度/高度 |
| 屏幕宽度 | 50 | 3000 |
| 屏幕高度 | 50 | 3000 |
| 屏幕密度 | 120 | 640 dpi |

### 键盘参数

| 参数 | 最小值 | 最大值 |
|------|--------|--------|
| KeyCode | 2000 | 2119 |
| KeyAction | 0 | 2 |

## SID 认证

WebSocket 连接支持可选的 SID (Session ID) 验证：

```cpp
// 验证 URI 中的 SID
bool WebSocketServer::CheckSid(struct lws* wsi)
{
    std::string uri(sidMaxLength, '\0');
    int uriLength = lws_hdr_copy(wsi, &uri[0], uri.size(), WSI_TOKEN_GET_URI);
    // 比较 SID
}
```

**证据**: `util/WebSocketServer.cpp:52-71`

---

## 相关文档

- 架构: [02_Architecture.md](./02_Architecture.md)
- 内部 API: [04_Inner_API.md](./04_Inner_API.md)
- 安全评审: [07_Security_Review.md](./07_Security_Review.md)
