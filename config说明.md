# alguiconfig.json 配置说明

> 本文件是辅助的云配置中心。改 JSON → push → 用户打开 App 自动生效，不用重装。
> 
> 配置仓库：https://gitee.com/lizidda/peizhi

---

## 顶层字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `version` | string | 配置版本号，每次打开App会拉最新，可忽略 |
| `app_update` | object | 应用强制更新配置 |
| `channels` | array | 渠道列表（目标游戏选择器里的按钮） |
| `detect_keywords` | array | 自动检测游戏进程的包名前缀 |
| `features` | object | 功能列表，每个 key 就是一个开关 |

---

## app_update（应用强制更新）

控制 App 启动时是否弹窗提示更新。

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
| `version_code` | int | 远程版本号（整数），比 App 内置的 `versionCode` 大才触发 |
| `version_name` | string | 版本名，显示在弹窗标题 |
| `force` | boolean | `true`=强制更新（不可取消），`false`=可点"稍后"跳过 |
| `download_url` | string | APK 下载链接，点"立即更新"跳转浏览器下载 |
| `update_msg` | string | 弹窗正文 |

### 触发逻辑

```
JSON 的 version_code > App 的 versionCode → 弹更新窗
JSON 的 version_code <= App 的 versionCode → 不弹
```

### 发版流程

1. 新 APK 上传，拿到下载链接
2. JSON 里 `version_code` 改大（比如 3），`force` 设 `true`，填 `download_url`
3. push JSON → 旧版用户打开 App 弹窗，强制更新
4. 新 APK 的 `build.gradle` 里 `versionCode` 也改成 3
5. 更新后的用户不再弹窗

---

## channels（渠道列表）

显示在"目标游戏"选择器里的渠道按钮。支持云更新，加渠道不用改代码。

```json
"channels": [
    {"name": "4399", "package": "org.cocos2dx.zmxy.m4399"},
    {"name": "TapTap", "package": "org.cocos2dx.zmxy.taptap"}
]
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 按钮上显示的名字 |
| `package` | string | 该游戏渠道的 Android 包名 |

### 加新渠道

1. 往数组里加一行 `{"name": "渠道名", "package": "包名"}`
2. 如果包名前缀不在 `detect_keywords` 里，也要加上
3. push 即可

---

## detect_keywords（自动检测关键字）

开启功能前，自动扫描运行中的进程，用这些前缀匹配游戏包名。

```json
"detect_keywords": [
    "org.cocos2dx.zmxy",
    "com.m4399.zmxy",
    "com.zmsy.zmxy",
    "com.tencent.tmgp.zmxy"
]
```

- 用 `startsWith` 匹配，比如 `org.cocos2dx.zmxy` 能匹配到 `org.cocos2dx.zmxy.m4399`、`org.cocos2dx.zmxy.taptap` 等
- 新增渠道时如果包名前缀不在列表里，记得加上
- 匹配到后自动切换到对应渠道，用户不用手动选

---

## features（功能列表）

每个 key 就是一个功能开关，菜单里自动生成一个开关按钮。

### 完整字段

```json
"功能key": {
    "name": "显示名称",
    "description": "说明文字",
    "enable_msg": "开启成功时的提示",
    "disable_msg": "关闭时的提示",
    "depends_on": "依赖的功能key",
    "memory_range": "RANGE_C_ALLOC",
    "search": {
        "value": "8423627",
        "type": "DWORD"
    },
    "offsets": [
        {"value": "9635", "type": "DWORD", "offset": -16}
    ],
    "writes": [
        {"value": "30000", "type": "DWORD", "offset": -10}
    ]
}
```

### 基本信息字段

| 字段 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `name` | string | ✅ | - | 菜单里显示的功能名 |
| `description` | string | ❌ | 空 | 开关下方的说明文字 |
| `enable_msg` | string | ❌ | "开启成功！" | 开启成功时的气泡通知 |
| `disable_msg` | string | ❌ | "关闭成功！" | 关闭时的气泡通知 |
| `depends_on` | string | ❌ | 空 | 依赖的其他功能 key |

### depends_on（功能依赖）

```json
"depends_on": "anti_ban"
```

- 填另一个功能的 **key**（不是 name）
- 执行本功能前，会**自动先执行**依赖功能（静默，不弹通知）
- 依赖功能只开一次，不会重复执行
- 典型用法：除防封外的所有功能都加 `"depends_on": "anti_ban"`，防止用户忘记开防封

### memory_range（内存范围）

决定搜索哪块内存区域。

| 值 | 缩写 | 说明 | 建议 |
|------|------|------|------|
| `RANGE_C_ALLOC` | CA | C层分配区 | ⭐ **首选，范围小、速度快、稳定** |
| `RANGE_C_HEAP` | CH | C层堆 | 备选 |
| `RANGE_C_DATA` | CD | C层数据段 | 备选 |
| `RANGE_C_BSS` | CB | C层BSS段 | 备选 |
| `RANGE_JAVA_HEAP` | JH | Java堆 | Java层数据用 |
| `RANGE_OTHER` | O | 其他内存 | ⚠️ 范围大、慢、低配机可能崩 |
| `RANGE_ALL` | - | 全部内存 | ⚠️ 最慢，不建议 |
| `RANGE_ANONYMOUS` | A | 匿名内存 | - |
| `RANGE_STACK` | S | 栈 | - |
| `RANGE_B_BAD` | B | B内存 | - |
| `RANGE_VIDEO` | V | 显存 | - |
| `RANGE_ASHMEM` | AS | 共享内存 | - |
| `RANGE_CODE_APP` | XA | 应用代码段 | - |
| `RANGE_CODE_SYSTEM` | XS | 系统代码段 | - |

> **经验：** 优先用 `RANGE_C_ALLOC`。搜不到再试 `RANGE_OTHER`。`RANGE_OTHER` 低配机容易崩。

### search（搜索参数）

在指定内存范围内搜索一个初始值。

```json
"search": {
    "value": "8423627",
    "type": "DWORD"
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `value` | string | 要搜索的数值，**字符串格式**，支持负数 |
| `type` | string | 数值类型 |

**type 可选值：**

| 值 | 说明 | value 示例 |
|------|------|------|
| `DWORD` | 4字节整数 | `"8423627"`, `"-343597384"`, `"0"` |
| `FLOAT` | 4字节浮点数 | `"3.14"`, `"100.0"` |
| `DOUBLE` | 8字节浮点数 | `"8888888"`, `"100000"` |

### offsets（偏移链）

从搜索结果出发，逐级偏移 + 验证，定位到目标地址。

```json
"offsets": [
    {"value": "9635", "type": "DWORD", "offset": -16},
    {"value": "19636", "type": "DWORD", "offset": -8}
]
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `value` | string | 这级偏移处应该有的值（用于验证指针链是否正确） |
| `type` | string | 值的类型（DWORD/FLOAT/DOUBLE） |
| `offset` | long | 相对偏移量（正数=往后，负数=往前） |

**执行逻辑：**

```
搜索结果 [地址A, 地址B, 地址C, ...]
  → 地址A + offset1 处读值 == value1？保留 / 丢弃
  → 保留的地址 + offset2 处读值 == value2？保留 / 丢弃
  → 逐级筛选，直到找到目标地址
```

**不需要偏移就写空数组：** `"offsets": []`

> ⚠️ 偏移链越长（超过 2 级），低配机越容易崩。代码会自动走子线程 + 安全检查。

### writes（写入）

在偏移链最终定位到的地址处写入数值。

```json
"writes": [
    {"value": "30000", "type": "DWORD", "offset": -10},
    {"value": "999999999", "type": "DWORD", "offset": 24}
]
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `value` | string | 要写入的值 |
| `type` | string | 值的类型（DWORD/FLOAT/DOUBLE） |
| `offset` | long | 相对于偏移链最终地址的偏移 |

---

## 完整示例

```json
{
    "version": "1.2",
    "app_update": {
        "version_code": 3,
        "version_name": "3.0",
        "force": true,
        "download_url": "https://example.com/algui-v3.apk",
        "update_msg": "发现新版本3.0，修复了崩溃问题，请更新！"
    },
    "channels": [
        {"name": "4399", "package": "org.cocos2dx.zmxy.m4399"},
        {"name": "TapTap", "package": "org.cocos2dx.zmxy.taptap"},
        {"name": "小米", "package": "org.cocos2dx.zmxy.mi"}
    ],
    "detect_keywords": [
        "org.cocos2dx.zmxy",
        "com.m4399.zmxy"
    ],
    "features": {
        "anti_ban": {
            "name": "防封",
            "description": "在长安城开启",
            "enable_msg": "防封已开启，安全了！",
            "disable_msg": "防封已关闭",
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
            "enable_msg": "数值已修改！",
            "disable_msg": "关闭成功！",
            "depends_on": "anti_ban",
            "memory_range": "RANGE_C_ALLOC",
            "search": {"value": "-343597384", "type": "DWORD"},
            "offsets": [
                {"value": "1067366481", "type": "DWORD", "offset": 4},
                {"value": "-2147483648", "type": "DWORD", "offset": 12}
            ],
            "writes": [
                {"value": "917800000", "type": "DWORD", "offset": 16},
                {"value": "8888888", "type": "DOUBLE", "offset": 72}
            ]
        }
    }
}
```

---

## 常见操作速查

### 加新功能
往 `features` 加一个新 key，填好 name/search/offsets/writes，push。

### 加新渠道
`channels` 加一行，检查 `detect_keywords` 是否包含该包名前缀。

### 改通知文案
改对应功能的 `enable_msg` 和 `disable_msg`。

### 防止忘开防封
除 `anti_ban` 外的功能都加 `"depends_on": "anti_ban"`。

### 游戏更新后改值
改对应功能的 `search.value`、`offsets[].value`、`writes[].value`。

### 强制用户更新 App
`app_update.version_code` 改大，`force` 设 `true`，填 `download_url`。

### 新增功能的安全保护
不用额外操作，所有功能自动享有：
- 进程检测（游戏没运行会提示）
- 搜索结果逐级检查（搜不到安全退出）
- 子线程执行（不卡UI）
- 主线程通知（不会崩溃）
