# alguiconfig.json 配置说明

## 顶层字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `version` | string | 配置版本号，改不改都行，每次打开App会拉最新 |
| `app_update` | object | 应用更新配置（见下方） |
| `channels` | array | 渠道列表，显示在"目标游戏"选择器里 |
| `detect_keywords` | array | 自动检测游戏进程用的关键字（包名前缀） |
| `features` | object | 功能列表，每个key就是一个开关 |

---

## app_update（应用更新）

```json
"app_update": {
    "version_code": 3,
    "version_name": "3.0",
    "force": true,
    "download_url": "https://xxx.apk",
    "update_msg": "发现新版本，请更新！"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `version_code` | int | 远程版本号（整数），比App的versionCode大就触发更新 |
| `version_name` | string | 版本名，显示在弹窗标题 |
| `force` | boolean | true=强制更新（不可取消），false=可点"稍后"跳过 |
| `download_url` | string | APK下载链接，点击"立即更新"会跳转浏览器 |
| `update_msg` | string | 弹窗正文内容 |

**触发逻辑：** App启动时比较 `app_update.version_code` 和 App内置的 `versionCode`，远程更大就弹窗。

**强制更新（force=true）：** 弹窗不可取消，用户只能点"立即更新"，不更新打不开。

**非强制更新（force=false）：** 弹窗可以点"稍后"跳过，正常进入。

---

## channels（渠道列表）

```json
{"name": "4399", "package": "org.cocos2dx.zmxy.m4399"}
```

| 字段 | 说明 |
|------|------|
| `name` | 显示在按钮上的名字 |
| `package` | 该游戏渠道的Android包名 |

新增渠道：直接往数组里加一个对象就行。

---

## detect_keywords（检测关键字）

```json
"org.cocos2dx.zmxy"
```

自动检测运行中的游戏时，用这些前缀匹配包名。
比如 `org.cocos2dx.zmxy` 能匹配到 `org.cocos2dx.zmxy.m4399`、`org.cocos2dx.zmxy.taptap` 等。

新增渠道时如果包名前缀不在列表里，记得加上。

---

## features（功能）

每个功能的结构：

```json
"功能key": {
    "name": "显示名称",
    "description": "说明文字",
    "enable_msg": "开启时的提示",
    "disable_msg": "关闭时的提示",
    "depends_on": "依赖的功能key",
    "memory_range": "内存范围",
    "search": { ... },
    "offsets": [ ... ],
    "writes": [ ... ]
}
```

### 基本信息

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 菜单里显示的功能名 |
| `description` | string | ❌ | 开关下方的说明文字，默认空 |
| `enable_msg` | string | ❌ | 开启成功时的气泡通知，默认"开启成功！" |
| `disable_msg` | string | ❌ | 关闭时的气泡通知，默认"关闭成功！" |
| `depends_on` | string | ❌ | 依赖的其他功能key，执行前会自动先开启 |

### depends_on（依赖）

```json
"depends_on": "anti_ban"
```

- 填另一个功能的 key（不是名字）
- 执行本功能前，会自动先执行依赖功能
- 依赖功能只开一次，不会重复执行
- 典型用法：所有功能依赖 `anti_ban`（防封）

### memory_range（内存范围）

决定搜索哪块内存区域。值如下：

| 值 | 缩写 | 说明 |
|------|------|------|
| `RANGE_ALL` | - | 全部内存（范围最大，最慢） |
| `RANGE_C_BSS` | CB | C层BSS段 |
| `RANGE_C_HEAP` | CH | C层堆 |
| `RANGE_C_ALLOC` | CA | C层分配（**最常用**，大部分功能用这个） |
| `RANGE_C_DATA` | CD | C层数据段 |
| `RANGE_STACK` | S | 栈 |
| `RANGE_B_BAD` | B | B内存 |
| `RANGE_ANONYMOUS` | A | 匿名内存 |
| `RANGE_JAVA_HEAP` | JH | Java堆 |
| `RANGE_CODE_APP` | XA | 应用代码段 |
| `RANGE_CODE_SYSTEM` | XS | 系统代码段 |
| `RANGE_VIDEO` | V | 显存 |
| `RANGE_ASHMEM` | AS | 共享内存 |
| `RANGE_OTHER` | O | 其他内存（**范围大，低配机慎用**） |

> `RANGE_C_ALLOC` 范围小、速度快、稳定，优先用这个。
> `RANGE_OTHER` 范围大、速度慢、低配机可能崩，只有找不到时才用。

### search（搜索）

```json
"search": {
    "value": "8423627",
    "type": "DWORD"
}
```

| 字段 | 说明 |
|------|------|
| `value` | 要搜索的数值（字符串格式，支持负数如 `"-343597384"`） |
| `type` | 数值类型 |

**type 可选值：**

| 值 | 说明 | 示例 |
|------|------|------|
| `DWORD` | 4字节整数（最常用） | `"8423627"`, `"-2003330816"` |
| `FLOAT` | 4字浮点数 | `"3.14"` |
| `DOUBLE` | 8字节浮点数 | `"8888888"` |

### offsets（偏移链）

从搜索结果出发，逐级偏移定位到目标地址。

```json
"offsets": [
    {"value": "9635", "type": "DWORD", "offset": -16},
    {"value": "19636", "type": "DWORD", "offset": -8}
]
```

每个对象：

| 字段 | 说明 |
|------|------|
| `value` | 这一级偏移处应该有的值（用于验证指针链是否正确） |
| `type` | 值的类型（DWORD/FLOAT/DOUBLE） |
| `offset` | 相对偏移量（正数往后，负数往前） |

**执行逻辑：**
1. 搜索到一批结果
2. 第1个offset：在结果地址 + offset处读值，和 value 对比，只保留匹配的
3. 第2个offset：在上一步结果 + offset处再读值对比
4. 逐级筛选，直到找到唯一目标地址

**不需要偏移就写空数组：** `"offsets": []`

### writes（写入）

在偏移链定位到的地址处写入数值。

```json
"writes": [
    {"value": "30000", "type": "DWORD", "offset": -10},
    {"value": "999999999", "type": "DWORD", "offset": 24}
]
```

每个对象：

| 字段 | 说明 |
|------|------|
| `value` | 要写入的值 |
| `type` | 值的类型（DWORD/FLOAT/DOUBLE） |
| `offset` | 相对于偏移链最终地址的偏移 |

**执行逻辑：**
从偏移链最终定位到的地址开始，按 offset 写入 value。

---

## 完整示例

```json
{
    "version": "1.2",
    "channels": [
        {"name": "4399", "package": "org.cocos2dx.zmxy.m4399"},
        {"name": "TapTap", "package": "org.cocos2dx.zmxy.taptap"}
    ],
    "detect_keywords": [
        "org.cocos2dx.zmxy"
    ],
    "features": {
        "anti_ban": {
            "name": "防封",
            "description": "在长安城开启",
            "enable_msg": "防封已开启，安全了！",
            "disable_msg": "关闭成功！",
            "depends_on": "",
            "memory_range": "RANGE_C_ALLOC",
            "search": {"value": "8423627", "type": "DWORD"},
            "offsets": [],
            "writes": [
                {"value": "-2003330816", "type": "DWORD", "offset": -4},
                {"value": "0", "type": "DWORD", "offset": -8}
            ]
        },
        "top_value": {
            "name": "顶级数值",
            "description": "9178",
            "enable_msg": "数值已改！",
            "disable_msg": "关闭成功！",
            "depends_on": "anti_ban",
            "memory_range": "RANGE_C_ALLOC",
            "search": {"value": "-343597384", "type": "DWORD"},
            "offsets": [
                {"value": "1067366481", "type": "DWORD", "offset": 4}
            ],
            "writes": [
                {"value": "917800000", "type": "DWORD", "offset": 16}
            ]
        }
    }
}
```

---

## 常见操作

### 加新功能
往 `features` 里加一个新 key，填好 search/offsets/writes，push 即可。

### 加新渠道
往 `channels` 里加一行，如果包名前缀新的还要加到 `detect_keywords`。

### 改提示语
改对应功能的 `enable_msg` 和 `disable_msg`。

### 改搜索值（游戏更新后）
改对应功能的 `search.value`、`offsets` 里的 `value`、`writes` 里的 `value`。

### 功能之间加依赖
给功能加 `"depends_on": "anti_ban"`，执行前会自动先开防封。
