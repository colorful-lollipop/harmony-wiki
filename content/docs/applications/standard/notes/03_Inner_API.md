# 内部 API

## 1. 模块概览

| 模块 | 路径 | 稳定性 | 导出类/函数 |
|------|------|--------|-----------|
| 数据库 | `common/utils/src/main/ets/default/baseUtil/RdbStoreUtil.ets` | 稳定 | `RdbStoreUtil` |
| 笔记工具 | `common/utils/src/main/ets/default/baseUtil/NoteUtil.ets` | 稳定 | `NoteUtil` |
| 文件夹工具 | `common/utils/src/main/ets/default/baseUtil/FolderUtil.ets` | 稳定 | `FolderUtil` |
| 日志工具 | `common/utils/src/main/ets/default/baseUtil/LogUtil.ets` | 稳定 | `LogUtil` |
| 日期工具 | `common/utils/src/main/ets/default/baseUtil/DateUtil.ets` | 稳定 | `DateUtil` |

**稳定性说明**: stable - 生产可用；unstable - 可能在未来版本中变更

## 2. RdbStoreUtil - 数据库操作

### 2.1 类信息

| 属性 | 值 |
|------|-----|
| **路径** | `common/utils/src/main/ets/default/baseUtil/RdbStoreUtil.ets` |
| **类型** | class |
| **稳定性** | stable |

### 2.2 核心方法

#### createRdbStore()

```typescript
createRdbStore(context: common.UIAbilityContext): void
```

**功能**: 初始化 relationalStore 数据库和表结构

**参数**:
- `context`: UIAbilityContext - 应用上下文

**实现位置**: `RdbStoreUtil.ets:52-100`

**代码片段**:
```typescript
createRdbStore(context: common.UIAbilityContext) {
  try {
    relationalStore.getRdbStore(context, SysDefData.dbInfo.db_name)
      .then(async (store) => {
        // 创建表
        store.executeSql(TableSql.CREATE_TABLE[TableName.NOTE_TABLE]);
        store.executeSql(TableSql.CREATE_TABLE[TableName.FOLDER_TABLE]);
        // 初始化系统默认数据
      })
  } catch (err) {
    LogUtil.error(TAG, 'createRdbStore failed: ' + err);
  }
}
```

#### insertNote()

```typescript
insertNote(noteData: NoteData): Promise<number>
```

**功能**: 插入笔记数据

**参数**:
- `noteData`: NoteData - 笔记数据对象

**返回值**:
- `Promise<number>` - 插入行的 ID

**代码片段**:
```typescript
async insertNote(noteData: NoteData): Promise<number> {
  let rowId = await this.rdbStore.insert(
    TableName.NOTE_TABLE,
    noteData.toNoteObject()
  );
  return rowId;
}
```

#### updateNote()

```typescript
updateNote(noteData: NoteData): Promise<number>
```

**功能**: 更新笔记数据

**参数**:
- `noteData`: NoteData - 笔记数据对象

**返回值**:
- `Promise<number>` - 更新的行数

#### deleteNote()

```typescript
deleteNote(id: number): Promise<number>
```

**功能**: 软删除笔记 (设置 is_deleted 标记)

**参数**:
- `id`: number - 笔记 ID

**返回值**:
- `Promise<number>` - 更新的行数

#### queryAllNotes()

```typescript
queryAllNotes(): Promise<NoteData[]>
```

**功能**: 查询所有未删除的笔记

**返回值**:
- `Promise<NoteData[]>` - 笔记数组

## 3. NoteUtil - 笔记工具

### 3.1 类信息

| 属性 | 值 |
|------|-----|
| **路径** | `common/utils/src/main/ets/default/baseUtil/NoteUtil.ets` |
| **类型** | class |
| **稳定性** | stable |

### 3.2 核心方法

#### createNote()

```typescript
createNote(title: string, folderUuid: string, content: string): NoteData
```

**功能**: 创建新笔记

**参数**:
- `title`: string - 笔记标题
- `folderUuid`: string - 所属文件夹 UUID
- `content`: string - 内容

**返回值**:
- `NoteData` - 新创建的笔记对象

#### updateNote()

```typescript
updateNote(note: NoteData): void
```

**功能**: 更新笔记

**参数**:
- `note`: NoteData - 笔记对象

#### deleteNote()

```typescript
deleteNote(note: NoteData): void
```

**功能**: 删除笔记 (软删除)

**参数**:
- `note`: NoteData - 笔记对象

#### toggleTop()

```typescript
toggleTop(note: NoteData): void
```

**功能**: 切换笔记置顶状态

**参数**:
- `note`: NoteData - 笔记对象

#### toggleFavorite()

```typescript
toggleFavorite(note: NoteData): void
```

**功能**: 切换笔记收藏状态

**参数**:
- `note`: NoteData - 笔记对象

## 4. FolderUtil - 文件夹工具

### 4.1 类信息

| 属性 | 值 |
|------|-----|
| **路径** | `common/utils/src/main/ets/default/baseUtil/FolderUtil.ets` |
| **类型** | class |
| **稳定性** | stable |

### 4.2 核心方法

#### createFolder()

```typescript
createFolder(name: string, color: string, type: FolderType): FolderData
```

**功能**: 创建新文件夹

**参数**:
- `name`: string - 文件夹名称
- `color`: string - 图标颜色
- `type`: FolderType - 文件夹类型

**返回值**:
- `FolderData` - 新创建的文件夹对象

#### updateFolder()

```typescript
updateFolder(folder: FolderData): void
```

**功能**: 更新文件夹

**参数**:
- `folder`: FolderData - 文件夹对象

#### deleteFolder()

```typescript
deleteFolder(folder: FolderData): void
```

**功能**: 删除文件夹

**参数**:
- `folder`: FolderData - 文件夹对象

#### getNotesInFolder()

```typescript
getNotesInFolder(folderUuid: string): NoteData[]
```

**功能**: 获取文件夹下的所有笔记

**参数**:
- `folderUuid`: string - 文件夹 UUID

**返回值**:
- `NoteData[]` - 笔记数组

## 5. 数据模型

### 5.1 NoteData

```typescript
export default class NoteData {
  id: number                    // 主键
  title: string                 // 标题
  uuid: string                  // 唯一标识
  folder_uuid: string           // 文件夹 UUID
  content_text: string          // 文本内容
  content_img: string           // 图片路径
  note_type: NoteType           // 笔记类型
  is_top: Top                   // 是否置顶
  is_favorite: Favorite         // 是否收藏
  is_deleted: Delete           // 是否删除
  created_time: number          // 创建时间
  modified_time: number         // 修改时间
  deleted_time: number          // 删除时间
  slider_value: number          // 字体大小
}
```

**证据位置**: `common/utils/src/main/ets/default/model/databaseModel/NoteData.ets:22-36`

### 5.2 FolderData

```typescript
export default class FolderData {
  id: number                    // 主键
  name: string                  // 名称
  uuid: string                  // 唯一标识
  color: string                 // 图标颜色
  folder_type: FolderType       // 文件夹类型
  is_deleted: Delete            // 是否删除
  created_time: number           // 创建时间
  modified_time: number         // 修改时间
}
```

**证据位置**: `common/utils/src/main/ets/default/model/databaseModel/FolderData.ets:22-30`

### 5.3 枚举定义

| 枚举 | 值 | 说明 |
|------|-----|------|
| `NoteType` | TEXT, IMAGE, AUDIO | 笔记类型 |
| `Favorite` | YES, NO | 是否收藏 |
| `Delete` | YES, NO | 是否删除 |
| `Top` | YES, NO | 是否置顶 |
| `FolderType` | SYSTEM, USER | 文件夹类型 |

**证据位置**: `common/utils/src/main/ets/default/model/databaseModel/EnumData.ets`

## 6. 依赖方向

```
┌─────────────────────────────────────────────────────────────┐
│                        依赖关系                              │
│                                                             │
│  pages/                                                    │
│     │                                                       │
│     ├── NoteListComp ──────┐                               │
│     │                      │                               │
│     ├── NoteContentComp ───┼──► RdbStoreUtil              │
│     │                      │                               │
│     └── FolderListComp ────┘                               │
│                                                             │
│  RdbStoreUtil ──► NoteData                                 │
│              ──► FolderData                                 │
│              ──► NoteUtil                                   │
│              ──► FolderUtil                                 │
└─────────────────────────────────────────────────────────────┘
```

## 7. 相关跳转

| 目标 | 链接 |
|------|------|
| 目录结构 | [01_Directory_Structure.md](01_Directory_Structure.md) |
| 架构设计 | [02_Architecture.md](02_Architecture.md) |
| 构建配置 | [04_GN_Targets.md](04_GN_Targets.md) |
| 安全评估 | [06_Security_Review.md](06_Security_Review.md) |
