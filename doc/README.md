# mpv-menu-plugin-next — mpv 跨平台右键选项菜单插件

## `input.conf` 如何定义菜单

菜单项目通过 `#menu:` 注释定义，命令本身仍然是普通 mpv `input.conf` 命令。例如：

```text
Ctrl+1  add contrast -1       #menu: 视频 > 调色 > 对比度 -1
Ctrl+2  add contrast  1       #menu: 视频 > 调色 > 对比度 +1
_       ignore                #menu: 视频 > 调色 > -
Ctrl+3  add brightness -1     #menu: 视频 > 调色 > 亮度 -1
Ctrl+4  add brightness  1     #menu: 视频 > 调色 > 亮度 +1
```

这里：

- `Ctrl+1`：菜单中显示的`快捷键`。
- `add contrast -1`：真正执行的 mpv 命令。
- `视频 > 调色 > 对比度 -1`：菜单层级和显示文字。
- `-`：表示分隔线。
- [完整示例：Full example](https://github.com/mpvConfig/mpv-menu-plugin-next/tree/main/example)

再例如：

```text
Enter   cycle fullscreen  #menu: 窗口 > 全屏 #@state=(fullscreen and 'checked')
```

会把 `窗口 > 全屏` 做成一个带动态勾选状态的菜单项。

### 动态菜单标记

`input.conf` 可以使用动态标记，`dyn_menu.lua` 会根据当前 mpv 状态重新生成菜单内容：

| 标记                     | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `#@tracks`               | 轨道总菜单                                                   |
| `#@tracks/video`         | 视频轨道                                                     |
| `#@tracks/audio`         | 音频轨道                                                     |
| `#@tracks/sub`           | 主字幕轨道                                                   |
| `#@tracks/sub-secondary` | 次字幕轨道                                                   |
| `#@chapters`             | 章节                                                         |
| `#@editions`             | 版本/edition                                                 |
| `#@audio-devices`        | 音频输出设备                                                 |
| `#@playlist`             | 播放列表                                                     |
| `#@profiles`             | mpv 配置文件/profiles                                        |
| `#@state=(...)`          | 根据 mpv 属性动态生成 `checked` / `disabled` / `hidden` 状态 |

#### 动态`#@state=(...)`写法

1. 属性简写模式（yes/no）

```
Enter   cycle fullscreen  #menu: 窗口 > 全屏 #@state=(fullscreen and 'checked')
Ctrl+t  cycle ontop       #menu: 窗口 > 置顶 #@state=(ontop and 'checked')
```

2. `#@state` 表达式里，**所有 mpv 选项名中的 `-` 都要写成 `_`**。例如：

| mpv 选项名       | `#@state` 里写法 |
| ---------------- | ---------------- |
| `video-sync`     | `video_sync`     |
| `audio-file`     | `audio_file`     |
| `sub-file`       | `sub_file`       |
| `audio-channels` | `audio_channels` |
| `sub-pos`        | `sub_pos`        |

例子：

```
_  set video-sync audio             #menu: 视频 > 帧同步模式 > audio #@state=(video_sync == "audio" and 'checked')
_  set video-sync display-resample  #menu: 视频 > 帧同步模式 > display-resample #@state=(video_sync == "display-resample" and 'checked')
_  set video-sync display-tempo     #menu: 视频 > 帧同步模式 > display-tempo #@state=(video_sync == "display-tempo" and 'checked')
```

3. 带有百分比数字

例子：`40%` > `40%x0`

```
_   ignore             #menu: 窗口 > 窗口大小 > 启动时窗口默认大小 #@state=('disabled')
_   ignore             #menu: 窗口 > 窗口大小 > -
_   set autofit ""     #menu: 窗口 > 窗口大小 > 自动（跟随视频分辨率） #@state=(autofit == "" and 'checked')
_   set autofit "40%"  #menu: 窗口 > 窗口大小 > 40% #@state=(autofit == "40%x0" and 'checked')
_   set autofit "50%"  #menu: 窗口 > 窗口大小 > 50% #@state=(autofit == "50%x0" and 'checked')
_   set autofit "60%"  #menu: 窗口 > 窗口大小 > 60% #@state=(autofit == "60%x0" and 'checked')
_   set autofit "70%"  #menu: 窗口 > 窗口大小 > 70% #@state=(autofit == "70%x0" and 'checked')
_   set autofit "80%"  #menu: 窗口 > 窗口大小 > 80% #@state=(autofit == "80%x0" and 'checked')
```

4. disabled 禁用，可作为提示，也可作为状态管理

例子：

```
_   ignore        #menu: 回放 > 提示：下边选项播放后才会启用，停止播放时会禁用 #@state=('disabled')
Space cycle pause #menu: 回放 > 播放/暂停 #@state=(idle_active and 'disabled' or (pause or 'hidden'))
```

5. 自定义属性绑定（非 mpv 自带属性）

```lua
# 在lua脚本中设置属性值
local auto_skip_start = true
mp.set_property_native('user-data/auto-skip-start', auto_skip_start)
```

然后在 `input.conf`中的写法如下:

需要转换

- `user-data/` → `ud_`
- `-` → `_`
- 拼接完整即：`user-data/auto-skip-start` → `ud_auto_skip_start`

```
Ctrl+A script-message-to skip_intro toggle-auto-start  #menu: 标记 > 自动跳过片头 #@state=(ud_auto_skip_start and 'checked')
```

## `menu_style.conf` 配置说明

配置文件位置：

```text
portable_config/script-opts/menu_style.conf
```

如果 `font_name=` 留空，菜单会按照平台自动选择字体：

- Windows：`Microsoft YaHei UI`
- macOS：`PingFang SC`
- Linux/其他：`Noto Sans CJK SC`

### 完整配置表

- 在 `menu_style.conf` 文件中的 Boolean 值需要写成 yes/no 即：以下表格的值(true ⇢ yes | false ⇢ no)

| 参数                           |    当前值 | 功能                                                |
| ------------------------------ | --------: | --------------------------------------------------- |
| `font`                         |        '' | 字体（留空则按平台自动选择）                        |
| `font_name`                    |        '' | 指定字体名称（非空时覆盖 `font`）                   |
| `font_size`                    |        29 | 菜单字号                                            |
| `bold`                         |     false | 是否粗体                                            |
| `italic`                       |     false | 是否斜体                                            |
| `menu_item_height`             |        47 | 普通菜单行高（`auto` 时按 OSD 高度相对 1080p 缩放） |
| `menu_item_height_fullscreen`  |        50 | 全屏菜单行高（`auto` 时按 OSD 高度相对 1080p 缩放） |
| `menu_font_scale`              |         1 | 菜单文字缩放倍数                                    |
| `separator_height`             |        13 | 分隔线高度                                          |
| `padding_x`                    |        12 | 菜单内容左右内边距                                  |
| `padding_y`                    |        13 | 菜单上下内边距                                      |
| `root_indent_chars`            |         1 | 一级菜单文字缩进                                    |
| `child_indent_chars`           |         1 | 子菜单文字缩进                                      |
| `playlist_header_indent_chars` |         2 | 播放列表顶部计数缩进                                |
| `background`                   |   #2C2C2E | 菜单背景色                                          |
| `bg_alpha`                     |         0 | 菜单背景透明度                                      |
| `radius`                       |        13 | 菜单圆角                                            |
| `border`                       |   #5A5A5F | 菜单外框色                                          |
| `border_width`                 |         2 | 外框宽度                                            |
| `text`                         |   #F2F2F7 | 普通文字颜色                                        |
| `disabled_text`                |   #8E8E93 | 禁用文字颜色                                        |
| `hover`                        |   #505050 | Hover 背景色                                        |
| `hover_bg`                     |   #505050 | Hover 背景色（非空时覆盖 `hover`）                  |
| `hover_text`                   |   #FFFFFF | Hover 文字色（空则回退到 `text`）                   |
| `hover_border`                 | #00000000 | Hover 边框颜色                                      |
| `hover_border_width`           |         0 | Hover 边框宽度                                      |
| `hover_corner_radius`          |         0 | Hover 圆角                                          |
| `hover_margin_x`               |         2 | Hover 背景左右留白                                  |
| `hover_margin_y`               |         0 | Hover 背景上下留白                                  |
| `shortcut`                     |   #D1D1D6 | 普通快捷键颜色                                      |
| `hover_shortcut`               |   #FFFFFF | Hover 快捷键颜色（空则回退到 `shortcut`）           |
| `shortcut_gap`                 |         7 | 标题和快捷键布局间距                                |
| `shortcut_right_gap`           |         4 | 快捷键右侧对齐基准                                  |
| `playlist_shortcut_right_gap`  |         4 | 播放列表右侧格式后缀额外距离                        |
| `arrow`                        |   #E5E5EA | 子菜单箭头颜色（非空时覆盖 `submenu_arrow`）        |
| `submenu_arrow`                |   #E5E5EA | 子菜单箭头颜色                                      |
| `arrow_char`                   |       '›' | 子菜单箭头字符号（比如： »,→, ->）                  |
| `arrow_alpha`                  |         0 | 普通箭头透明度                                      |
| `hover_arrow`                  |   #FFFFFF | Hover 箭头颜色（空则回退到 `submenu_arrow`）        |
| `hover_arrow_alpha`            |         0 | Hover 箭头透明度                                    |
| `arrow_width`                  |        32 | 箭头区域宽度                                        |
| `arrow_font_size`              |        43 | 箭头字号                                            |
| `separator`                    |   #636366 | 分隔线颜色                                          |
| `check`                        |   #F2F2F7 | 勾选颜色                                            |
| `shadow`                       |   #000000 | 阴影颜色                                            |
| `shadow_alpha`                 |        90 | 阴影透明度                                          |
| `shadow_blur`                  |         7 | 阴影模糊                                            |
| `submenu_gap`                  |         7 | 子菜单间距                                          |
| `screen_margin`                |        11 | 菜单距离 OSD 边缘的最小距离                         |
| `min_width`                    |       350 | 菜单最小宽度                                        |
| `max_width`                    |      1080 | 菜单最大宽度                                        |
| `max_visible_rows`             |         0 | 最大可见行数配置项                                  |
| `click_to_show_submenus`       |     false | 是否要求点击才展开子菜单                            |
| `hide_root_separators`         |      true | 是否隐藏一级菜单内部的分隔线                        |
| `modal_mask`                   |      true | 是否启用全屏模态遮罩                                |
| `modal_mask_alpha`             |       255 | 模态遮罩透明度                                      |
| `modal_z`                      |   1000000 | 模态遮罩层级                                        |
| `scroll_threshold`             |      0.80 | 超过窗口高度多少比例后进入滚动                      |
| `scrollbar_width`              |         9 | 滚动条宽度                                          |
| `scrollbar_gap`                |         0 | 滚动条与内容区域间距                                |
| `scrollbar_right_gap`          |         2 | 滚动条与右边框间距                                  |
| `scrollbar_min_thumb`          |        36 | 滚动条滑块最小高度                                  |
| `scrollbar_track`              |   #48484A | 滚动条轨道颜色                                      |
| `scrollbar_thumb`              |   #8E8E93 | 滚动条滑块颜色                                      |
| `scrollbar_track_alpha`        |        70 | 滚动轨道透明度                                      |
| `scrollbar_thumb_alpha`        |       145 | 滚动滑块透明度                                      |
| `scroll_step`                  |         2 | 单次滚动步长                                        |
| `macos_font_scale`             |       1.4 | macOS 字体视觉补偿                                  |
| **Liquid Glass**               |           |                                                     |
| `glass_enabled`                |     false | 总开关；关闭后回退到原始菜单样式                    |
| `glass_color`                  |   #17181C | 玻璃底色                                            |
| `glass_opacity`                |        38 | 着色强度：0=清透，100=实色                          |
| `glass_border_opacity`         |         8 | 边缘亮度/不透明度：0=不显示，100=完全显示           |
| `glass_shadow`                 |      true | 软阴影                                              |
| `glass_blur`                   |      true | GLSL 真模糊（需 vo=gpu-next）                       |
| `glass_refraction`             |      true | 边缘折射                                            |
| `glass_increase_contrast`      |     false | 无障碍：黑玻璃 + 可见描边                           |
| `glass_reduce_transparency`    |     false | 无障碍：磨砂近不透明                                |
| `reduce_motion`                |     false | 关闭弹簧/缩放动画（当前菜单本身不使用此类动画）     |

- `script-opts/dyn_menu.conf` 当前包含 `max_playlist_items`：设置为 `0` 表示不截断播放列表，完整生成并交给菜单滚动容器处理。

## 方法调用

第三方插件可调用组件实时更新面板样式，或者通过快捷键绑定

```
key script-message-to menu reload-menu #menu: 刷新菜单样式
```

```lua
------------------------------------------------------------
-- Reload 相关；reload‑menu
------------------------------------------------------------
local function reload_menu_scripts()
    pcall(function()
        mp.commandv('script-message-to', 'menu', 'reload-menu')
    end)
    return true
end
```

## 组件职责

#### `menu.lua`

负责菜单绘制、布局、Hover、子菜单、滚动、播放列表定位和 Modal 层。

#### `dyn_menu.lua`

负责解析 `input.conf` 并生成动态菜单，例如轨道、章节、播放列表和配置文件。

#### `dialog.lua`

负责文件选择、文件夹选择、保存以及剪贴板等辅助功能。

### Windows

当前首选路径采用 `mp.utils.subprocess()` 调用 PowerShell，再使用 .NET 图形文件对话框；文件选择、文件夹选择、保存对话框均通过独立 subprocess 完成。打开文件时会返回 UTF-8 路径，并按 `input.conf` 选择 `replace` / `append` 等动作。

### macOS

使用 `osascript` 调用系统的 `choose file` / `choose folder` / `choose file name`。

### Linux

按可用性自动查找：

1. `zenity`
2. `kdialog`
3. `yad`

如果系统没有这些工具，文件对话框功能会提示缺少依赖。

## 剪贴板

剪贴板后端按平台选择：

| 平台    | 读取                          | 写入                         |
| ------- | ----------------------------- | ---------------------------- |
| Windows | PowerShell `Get-Clipboard`    | PowerShell `Set-Clipboard`   |
| macOS   | `pbpaste`                     | `pbcopy`                     |
| Linux   | `wl-paste` → `xclip` → `xsel` | `wl-copy` → `xclip` → `xsel` |

Linux 需要系统中至少存在一组可用的剪贴板命令。

## 故障排查

### 菜单完全不显示

确认：

1. `scripts/menu/menu.lua`、`dyn_menu.lua`、`dialog.lua` 在 `portable_config/scripts/menu` 下。
2. 没有同时加载旧 `menu.dll`。
3. `input.conf` 存在并包含 `#menu:` 定义。

### 右键菜单打开，但动态项目为空

播放文件后重新打开菜单。当前版本会在 `file-loaded`、`start-file` 和右键打开前刷新动态菜单。

### Windows 文件选择框失败

先确认 PowerShell 可执行文件可用；当前 Windows 路径会优先寻找系统 PowerShell，并通过 `mp.utils.subprocess()` 调用。

### Linux 文件选择框失败

确认至少安装：

```text
zenity
```

或：

```text
kdialog
```

或：

```text
yad
```
