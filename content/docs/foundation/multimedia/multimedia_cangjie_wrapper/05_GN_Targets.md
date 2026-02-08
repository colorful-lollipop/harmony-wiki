# GN Targets 与构建系统

> 本文档详细描述 multimedia_cangjie_wrapper 的 GN 构建目标、依赖关系与配置

---

## 目的与适用范围

**目的**：帮助开发者理解构建系统，添加新的构建目标或排查构建问题

**适用范围**：构建工程师、框架开发者

---

## 构建文件总览

### 文件列表

| 文件路径 | 职责 |
|----------|------|
| `BUILD.gn` | 根构建文件，定义顶层目标 |
| `ohos/multimedia/BUILD.gn` | ohos.multimedia 目标 |
| `ohos/multimedia/camera/BUILD.gn` | ohos.multimedia.camera 目标 |
| `ohos/multimedia/image/BUILD.gn` | ohos.multimedia.image 目标 |
| `ohos/multimedia/media/BUILD.gn` | ohos.multimedia.media 目标 |
| `ohos/file/photo_access_helper/BUILD.gn` | ohos.file.photo_access_helper 目标 |
| `kit/CameraKit/BUILD.gn` | kit.CameraKit 目标 |
| `kit/ImageKit/BUILD.gn` | kit.ImageKit 目标 |
| `kit/MediaKit/BUILD.gn` | kit.MediaKit 目标 |
| `kit/MediaLibraryKit/BUILD.gn` | kit.MediaLibraryKit 目标 |

---

## 根构建文件 (`BUILD.gn`)

### 完整内容

```gn
# Copyright (c) 2025 Huawei Device Co., Ltd.
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

import("//build/templates/cangjie/cjc.gni")

multimedia_cangjie_wrapper_packages_ohos = [
    "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/camera:ohos.multimedia.camera",
    "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/file/photo_access_helper:ohos.file.photo_access_helper",
    "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/media:ohos.multimedia.media",
    "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/image:ohos.multimedia.image",
    "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia:ohos.multimedia"
]

multimedia_cangjie_wrapper_packages_kit = [
    "//foundation/multimedia/multimedia_cangjie_wrapper/kit/MediaKit:kit.MediaKit",
    "//foundation/multimedia/multimedia_cangjie_wrapper/kit/MediaLibraryKit:kit.MediaLibraryKit",
    "//foundation/multimedia/multimedia_cangjie_wrapper/kit/CameraKit:kit.CameraKit",
    "//foundation/multimedia/multimedia_cangjie_wrapper/kit/ImageKit:kit.ImageKit",
]

copy_ohos_cangjie_sdk_api_lib("copy_sdk_multimedia_cangjie_libs") {
  ohos_inputs = multimedia_cangjie_wrapper_packages_ohos
  kit_inputs = multimedia_cangjie_wrapper_packages_kit
}
```

### 目标说明

| 目标名 | 类型 | 输出 | 作用 |
|--------|------|------|------|
| `copy_sdk_multimedia_cangjie_libs` | `copy_ohos_cangjie_sdk_api_lib` | SDK 库文件 | 复制 ohos 和 kit 包到 SDK 目录 |

---

## 模块构建文件

### ohos.multimedia.camera

**文件**: `ohos/multimedia/camera/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.multimedia.camera") {
  if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.multimedia.camera.cj"]
  } else {
    sources = [
      "camera.cj",
      "camera_ability.cj",
      "camera_common.cj",
      "camera_ffi.cj",
      "camera_input.cj",
      "camera_manager.cj",
      "camera_output.cj",
      "camera_session.cj",
      "photo_output.cj",
      "preview_output.cj",
      "video_output.cj",
    ]
  }

  cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "graphic_cangjie_wrapper:ohos.graphics.color_space_manager",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "camera_framework:cj_camera_ffi" ]

  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

| 属性 | 值 |
|------|-----|
| **Target 类型** | `ohos_cangjie_shared_library` |
| **输出名** | `ohos.multimedia.camera` |
| **源文件数** | 10 个（Linux）/ 1 个 Mock（Windows/Mac） |
| **cj_external_deps** | 7 个 Wrapper 依赖 |
| **external_deps** | 1 个底层框架依赖 |

### ohos.multimedia.image

**文件**: `ohos/multimedia/image/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.multimedia.image") {
  if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.multimedia.image.cj"]
  } else {
    sources = [
      "cj_image_common.cj",
      "cj_image_enum.cj",
      "cj_image_log.cj",
      "cj_image_utils.cj",
      "image.cj",
      "image_packer.cj",
      "image_receiver.cj",
      "image_source.cj",
      "pixel_map.cj",
    ]
  }

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "global_cangjie_wrapper:ohos.resource_manager",
    "global_cangjie_wrapper:ohos.raw_file_descriptor",
    "graphic_cangjie_wrapper:ohos.graphics.color_space_manager",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [ "image_framework:cj_image_ffi" ]

  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

| 属性 | 值 |
|------|-----|
| **Target 类型** | `ohos_cangjie_shared_library` |
| **输出名** | `ohos.multimedia.image` |
| **源文件数** | 9 个（Linux）/ 1 个 Mock（Windows/Mac） |
| **cj_external_deps** | 8 个 Wrapper 依赖 |
| **external_deps** | 1 个底层框架依赖 |

### ohos.multimedia.media

**文件**: `ohos/multimedia/media/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.multimedia.media") {
  if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.multimedia.media.cj"]
  } else {
    sources = [
      "avimage_generator.cj",
      "media_common.cj",
      "media_ffi.cj",
    ]
  }

  cj_deps = [
    "../../multimedia/image:ohos.multimedia.image"
  ]

  cj_external_deps = [
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.ffi",
    "cangjie_ark_interop:ohos.labels",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
  ]

  external_deps = [
    "player_framework:cj_avplayer_ffi",
    "player_framework:cj_avscreen_capture_ffi",
    "player_framework:cj_avtranscoder_ffi",
    "player_framework:cj_media_avrecorder_ffi",
    "player_framework:cj_metadatahelper_ffi",
    "player_framework:cj_soundpool_ffi",
  ]

  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

| 属性 | 值 |
|------|-----|
| **Target 类型** | `ohos_cangjie_shared_library` |
| **输出名** | `ohos.multimedia.media` |
| **源文件数** | 3 个（Linux）/ 1 个 Mock（Windows/Mac） |
| **cj_deps** | 1 个内部依赖（image） |
| **cj_external_deps** | 4 个 Wrapper 依赖 |
| **external_deps** | 6 个底层框架依赖 |

### ohos.file.photo_access_helper

**文件**: `ohos/file/photo_access_helper/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.file.photo_access_helper") {
  if (is_mingw || is_mac) {
    sources = ["../../../mock/ohos.file.photo_access_helper.cj"]
  } else {
    sources = [
      "album.cj",
      "fetch_result.cj",
      "media_album_change_request.cj",
      "media_asset_change_request.cj",
      "photo_accesshelper.cj",
      "photo_accesshelper_ffi.cj",
      "photo_accesshelper_utils.cj",
      "photo_asset.cj"
    ]
  }

  cj_deps = [
    "../../multimedia/image:ohos.multimedia.image"
  ]

  cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability",
    "cangjie_ark_interop:ohos.business_exception",
    "cangjie_ark_interop:ohos.callback_invoke",
    "bundlemanager_cangjie_wrapper:ohos.bundle.bundle_manager",
    "distributeddatamgr_cangjie_wrapper:ohos.data.data_share_predicates",
    "cangjie_ark_interop:ohos.ffi",
    "hiviewdfx_cangjie_wrapper:ohos.hilog",
    "global_cangjie_wrapper:ohos.resource_manager",
    "cangjie_ark_interop:ohos.labels",
    "cangjie_ark_interop:ohos.business_exception",
  ]

  external_deps = [ "media_library:cj_photoaccesshelper_ffi" ]

  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

| 属性 | 值 |
|------|-----|
| **Target 类型** | `ohos_cangjie_shared_library` |
| **输出名** | `ohos.file.photo_access_helper` |
| **源文件数** | 8 个（Linux）/ 1 个 Mock（Windows/Mac） |
| **cj_deps** | 1 个内部依赖（image） |
| **cj_external_deps** | 10 个 Wrapper 依赖 |
| **external_deps** | 1 个底层框架依赖 |

### ohos.multimedia

**文件**: `ohos/multimedia/BUILD.gn`

```gn
ohos_cangjie_shared_library("ohos.multimedia") {
  if (is_mingw || is_mac) {
    sources = ["../../mock/ohos.multimedia.cj"]
  } else {
    sources = [ "multimedia.cj" ]
  }

  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

| 属性 | 值 |
|------|-----|
| **Target 类型** | `ohos_cangjie_shared_library` |
| **输出名** | `ohos.multimedia` |
| **源文件数** | 1 个 |
| **依赖** | 无（空壳模块） |

---

## Kit 层构建文件

### Kit 层共性

所有 Kit 层构建文件结构相似：

```gn
ohos_cangjie_shared_library("kit.XxxKit") {
  sources = ["index.cj"]
  cj_deps = ["../../ohos/xxx:ohos.xxx"]
  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

| Kit | Target 名 | 依赖的 ohos 模块 |
|-----|-----------|-----------------|
| CameraKit | `kit.CameraKit` | `ohos.multimedia.camera` |
| ImageKit | `kit.ImageKit` | `ohos.multimedia.image` |
| MediaKit | `kit.MediaKit` | `ohos.multimedia.media` |
| MediaLibraryKit | `kit.MediaLibraryKit` | `ohos.file.photo_access_helper` + `ability_cangjie_wrapper` |

### MediaLibraryKit 特殊依赖

```gn
ohos_cangjie_shared_library("kit.MediaLibraryKit") {
  sources = ["index.cj"]
  cj_deps = [
    "../../ohos/file/photo_access_helper:ohos.file.photo_access_helper",
  ]
  cj_external_deps = [
    "ability_cangjie_wrapper:ohos.app.ability.ui_ability"
  ]
  subsystem_name = "multimedia"
  part_name = "multimedia_cangjie_wrapper"
}
```

---

## 依赖汇总

### Wrapper 依赖（cj_external_deps）

| Wrapper 组件 | 被哪些模块使用 |
|--------------|---------------|
| `ability_cangjie_wrapper` | camera, photo_access_helper, MediaLibraryKit |
| `bundlemanager_cangjie_wrapper` | photo_access_helper |
| `cangjie_ark_interop` | camera, image, media, photo_access_helper |
| `distributeddatamgr_cangjie_wrapper` | photo_access_helper |
| `global_cangjie_wrapper` | image, photo_access_helper |
| `graphic_cangjie_wrapper` | camera, image |
| `hiviewdfx_cangjie_wrapper` | camera, image, media, photo_access_helper |

### 底层框架依赖（external_deps）

| 框架组件 | 被哪些模块使用 |
|----------|---------------|
| `camera_framework:cj_camera_ffi` | camera |
| `image_framework:cj_image_ffi` | image |
| `player_framework:cj_avplayer_ffi` | media |
| `player_framework:cj_avscreen_capture_ffi` | media |
| `player_framework:cj_avtranscoder_ffi` | media |
| `player_framework:cj_media_avrecorder_ffi` | media |
| `player_framework:cj_metadatahelper_ffi` | media |
| `player_framework:cj_soundpool_ffi` | media |
| `media_library:cj_photoaccesshelper_ffi` | photo_access_helper |

---

## 构建目标关系图

```
                                    ┌──────────────────────────────────────┐
                                    │  copy_sdk_multimedia_cangjie_libs    │
                                    │  (SDK 复制目标)                      │
                                    └──────────────┬───────────────────────┘
                                                   │
           ┌───────────────────────────────────────┼───────────────────────────────────────┐
           │                                       │                                       │
           ▼                                       ▼                                       ▼
┌──────────────────────┐             ┌──────────────────────┐             ┌──────────────────────┐
│  ohos.* 目标         │             │  kit.* 目标          │             │  依赖                │
├──────────────────────┤             ├──────────────────────┤             ├──────────────────────┤
│ ohos.multimedia      │             │ kit.CameraKit        │             │ cangjie_ark_interop  │
│ ohos.multimedia.cam- │────────────▶│ kit.ImageKit         │             │ ability_cangjie_     │
│   era                │             │ kit.MediaKit         │             │   wrapper            │
│ ohos.multimedia.ima- │────────────▶│ kit.MediaLibraryKit  │             │ hiviewdfx_cangjie_   │
│   ge                 │             └──────────────────────┘             │   wrapper            │
│ ohos.multimedia.med- │                                                  │ ...                  │
│   ia                 │                                                  └──────────────────────┘
│ ohos.file.photo_acc- │
│   ess_helper         │
└──────────────────────┘
           │
           │ internal_deps
           ▼
┌──────────────────────────────────────┐
│  底层 C++ 框架                        │
│  camera_framework                    │
│  image_framework                     │
│  player_framework                    │
│  media_library                       │
└──────────────────────────────────────┘
```

---

## Bundle.json 中的构建配置

### Sub Component

来源：`bundle.json:39-49`

```json
"sub_component": [
  "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia:ohos.multimedia",
  "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/image:ohos.multimedia.image",
  "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/media:ohos.multimedia.media",
  "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/camera:ohos.multimedia.camera",
  "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/file/photo_access_helper:ohos.file.photo_access_helper",
  "//foundation/multimedia/multimedia_cangjie_wrapper/kit/MediaLibraryKit:kit.MediaLibraryKit",
  "//foundation/multimedia/multimedia_cangjie_wrapper/kit/ImageKit:kit.ImageKit",
  "//foundation/multimedia/multimedia_cangjie_wrapper/kit/MediaKit:kit.MediaKit",
  "//foundation/multimedia/multimedia_cangjie_wrapper/kit/CameraKit:kit.CameraKit"
]
```

### Inner Kits

来源：`bundle.json:50-60`

```json
"inner_kits": [
  {
    "name": "//foundation/multimedia/multimedia_cangjie_wrapper/ohos/multimedia/image:ohos.multimedia.image"
  },
  {
    "name": "//foundation/multimedia/multimedia_cangjie_wrapper:copy_sdk_multimedia_cangjie_libs"
  },
  {
    "name": "//foundation/multimedia/multimedia_cangjie_wrapper:copy_sdk_multimedia_cangjie_libs_kit"
  }
]
```

---

## 关键结论

1. **统一模板**：所有模块使用 `ohos_cangjie_shared_library` 模板
2. **平台适配**：通过 `is_mingw || is_mac` 条件使用 Mock 源文件
3. **依赖清晰**：cj_external_deps（Wrapper）与 external_deps（底层框架）分离
4. **层级构建**：ohos 层先构建，kit 层后构建
5. **SDK 复制**：`copy_ohos_cangjie_sdk_api_lib` 将产物复制到 SDK 目录

---

## 相关跳转

- [目录结构](02_Directory_Structure.md)
- [编译产物](06_Build_Artifacts.md)
- [内部 API](04_Inner_API.md)
