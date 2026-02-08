# 05_Usage - 使用说明

## 概述

本文档介绍如何在 OpenHarmony 应用中使用 advanced_ui_component 提供的各种组件。

---

## 导入组件

### 自动导入

在 OpenHarmony 应用中，advanced_ui_component 会随系统自动部署，无需手动导入。

### 手动导入 (如需)

```typescript
// 方式 1: 通过包名导入
import { AtomicServiceNavigation } from '@ohos/atomicService';

// 方式 2: 通过完整路径导入
import { AtomicServiceTabs } from 'package:@ohos/atomicService/atomicServiceTabs';
```

---

## 组件使用

### AtomServiceNavigation

```typescript
import { AtomicServiceNavigation } from '@ohos/atomicService';

@Entry
@Component
struct Index {
  build() {
    Column() {
      AtomicServiceNavigation({
        // 组件属性
        title: '页面标题',
        onNavigationChange: (isShown: boolean) => {
          console.info('Navigation shown:', isShown);
        }
      }) {
        // 导航内容
        Column() {
          Text('导航内容区域')
          Button('返回')
            .onClick(() => {
              // 返回操作
            })
        }
        .width('100%')
        .height('100%')
      }
      .width('100%')
      .height('100%')
    }
  }
}
```

### AtomServiceSearch

```typescript
import { AtomicServiceSearch } from '@ohos/atomicService';

@Entry
@Component
struct SearchPage {
  @State searchText: string = '';

  build() {
    Column() {
      AtomicServiceSearch({
        placeholder: '搜索内容',
        searchText: this.searchText,
        onSearch: (text: string) => {
          console.info('Search:', text);
          // 执行搜索
        },
        onChange: (text: string) => {
          this.searchText = text;
        }
      })
    }
  }
}
```

### AtomServiceTabs

```typescript
import { AtomicServiceTabs } from '@ohos/atomicService';

@Entry
@Component
struct TabsPage {
  @State selectedIndex: number = 0;

  build() {
    Column() {
      AtomicServiceTabs({
        selectedIndex: this.selectedIndex,
        onTabChange: (index: number) => {
          this.selectedIndex = index;
          console.info('Tab changed to:', index);
        }
      }) {
        TabContent() {
          Text('标签页 1 内容')
        }
        .tabText('标签 1')

        TabContent() {
          Text('标签页 2 内容')
        }
        .tabText('标签 2')

        TabContent() {
          Text('标签页 3 内容')
        }
        .tabText('标签 3')
      }
    }
  }
}
```

### AtomServiceWeb

```typescript
import { AtomicServiceWeb } from '@ohos/atomicService';

@Entry
@Component
struct WebPage {
  @State url: string = 'https://example.com';
  @State bundleName: string = 'com.example.app';
  @State domainType: string = 'news';

  build() {
    Column() {
      AtomicServiceWeb({
        src: this.url,
        bundleName: this.bundleName,
        domainType: this.domainType
      })
      .width('100%')
      .height('80%')

      Button('访问新页面')
        .onClick(() => {
          this.checkAndNavigate('https://newsite.com');
        })
    }
  }

  checkAndNavigate(newUrl: string) {
    // 使用 checkUrl 校验 URL
    AtomicServiceWeb.checkUrl(this.bundleName, this.domainType, newUrl)
      .then((result: number) => {
        if (result === 0) {
          // 校验通过
          this.url = newUrl;
        } else {
          console.error('URL check failed:', result);
        }
      });
  }
}
```

**注意**: `checkUrl` 需要系统 API 策略库支持。

---

### FullScreenLaunchComponent

```typescript
import { FullScreenLaunchComponent } from '@ohos/atomicService';

@Entry
@Component
struct LaunchPage {
  // 启动页资源
  @State launchImage: Resource = $r('app.media.launch_image');

  build() {
    Stack() {
      FullScreenLaunchComponent({
        image: this.launchImage,
        duration: 3000,  // 显示时长 (ms)
        onFinish: () => {
          // 启动完成，跳转到主页面
          router.replaceUrl({ url: 'pages/MainPage' });
        }
      })
    }
  }
}
```

### HalfScreenLaunchComponent

```typescript
import { HalfScreenLaunchComponent } from '@ohos/atomicService';

@Entry
@Component
struct LaunchPage {
  @State logo: Resource = $r('app.media.logo');

  build() {
    Column() {
      HalfScreenLaunchComponent({
        logo: this.logo,
        appName: '我的应用',
        companyName: '公司名称',
        duration: 2000,
        onFinish: () => {
          router.replaceUrl({ url: 'pages/MainPage' });
        }
      })
    }
  }
}
```

### InterstitialDialogAction

```typescript
import { InterstitialDialogAction } from '@ohos/atomicService';

@Entry
@Component
struct MainPage {
  @State dialogVisible: boolean = false;

  build() {
    Column() {
      Button('显示插屏广告')
        .onClick(() => {
          this.dialogVisible = true;
        })

      InterstitialDialogAction({
        visible: this.dialogVisible,
        type: 'advertisement',  // 弹窗类型
        content: $r('app.media.ad_content'),
        onClose: () => {
          this.dialogVisible = false;
        },
        onClick: () => {
          // 点击广告处理
        }
      })
    }
  }
}
```

---

## NavPushPathHelper 使用

### HSP 静默安装

```typescript
import { NavPushPathHelper } from '@ohos/atomicService';

@Entry
@Component
struct MainPage {
  moduleName: string = 'DynamicFeature';

  aboutToAppear() {
    this.installHsp();
  }

  async installHsp() {
    // 方式 1: Promise 方式
    try {
      await NavPushPathHelper.silentInstall(this.moduleName);
      console.info('HSP installed successfully');
      // 跳转到 HSP 页面
      NavPushPathHelper.updateRouteMap({});
    } catch (error) {
      console.error('HSP install failed:', error);
    }

    // 方式 2: 回调方式
    NavPushPathHelper.silentInstall(this.moduleName, (errCode, errorMessage) => {
      if (errCode === 0) {
        console.info('HSP installed');
      } else {
        console.error('Install failed:', errorMessage);
      }
    });
  }
}
```

### 检查 HSP 存在性

```typescript
import { NavPushPathHelper } from '@ohos/atomicService';

function checkHspExists(moduleName: string): boolean {
  return NavPushPathHelper.isHspExist(moduleName);
}
```

### 更新路由表

```typescript
import { NavPushPathHelper } from '@ohos/atomicService';

interface RouteMap {
  [key: string]: string;
}

const newRouteMap: RouteMap = {
  'DynamicFeature': 'pages/DynamicPage',
  'PluginPage': 'pages/PluginPage'
};

NavPushPathHelper.updateRouteMap(newRouteMap, (errCode) => {
  if (errCode === 0) {
    console.info('Route map updated');
  }
});
```

---

## CustomAppBar 使用

```typescript
import { CustomAppBar } from '@ohos/atomicService';

@Entry
@Component
struct MainPage {
  @State menuVisible: boolean = false;

  build() {
    Column() {
      CustomAppBar({
        title: '页面标题',
        menuItems: [
          { id: 'settings', title: '设置' },
          { id: 'about', title: '关于' }
        ],
        onMenuClick: (menuId: string) => {
          console.info('Menu clicked:', menuId);
        }
      })

      // 页面内容
      Text('页面内容')
        .padding(16)
    }
  }
}
```

---

## 权限要求

### 最小权限

大多数组件无需特殊权限即可使用。

### NavPushPathHelper 权限

使用 `NavPushPathHelper.silentInstall` 需要以下权限：

| 权限 | 说明 |
|------|------|
| `ohos.permission.INSTALL_BUNDLE` | 静默安装 HSP 包 |

**在 module.json5 中声明**:
```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INSTALL_BUNDLE",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

---

## 注意事项

### 1. 兼容性

- 确保使用 API 10+ 的 SDK
- 组件在不同设备上表现可能略有差异

### 2. 性能

- 避免在启动组件中加载过多资源
- HSP 静默安装应在合适时机调用

### 3. 错误处理

```typescript
// 推荐的错误处理模式
try {
  // 调用可能失败的 API
} catch (error) {
  console.error('Error:', error);
  // 显示错误提示
}

// 异步错误处理
async function safeCall() {
  const result = await someApi().catch((err) => {
    console.error('API error:', err);
    return null;
  });
  if (result) {
    // 处理结果
  }
}
```

### 4. 资源释放

对于涉及原生资源的组件，确保正确释放：

```typescript
// 组件销毁时清理
onDisapper() {
  // 释放资源
}
```

---

## 调试技巧

### 1. 查看日志

```bash
# 过滤相关日志
hilog | grep -E "AtomicService|NavPushPathHelper"
```

### 2. 检查组件状态

```typescript
// 添加状态监听
this.controller.on('stateChange', (state) => {
  console.info('Component state:', state);
});
```

---

## 相关文档

- [02_Architecture](02_Architecture.md) - 架构设计
- [03_Components](03_Components.md) - 组件 API 详情
- [06_Security](06_Security.md) - 安全注意事项