# mpv-menu-plugin-next — mpv 跨平台右键选项菜单插件

> - 一个基于 **Lua + ASS OSD** 的 mpv 右键选项菜单后端。菜单内容由 `input.conf` 决定，多层级、快捷键显示和交互方式，支持 Windows、macOS、Linux
> - An mpv configurable context menu backend based on **Lua + ASS OSD**. Menu content is determined by `input.conf` & Can be used on Windows、macOS、Linux。

## 1. 功能概览(Features)

本插件把 `input.conf` 中的 `#menu:` 定义转换成可交互的右键菜单，并继续执行每一行原本对应的 mpv 命令。

主要功能：

- `input.conf` 驱动菜单内容，不需要另写一套菜单数据。
- 支持多级菜单，例如 `视频 > 调色 > 对比度 +1`。
- 支持样式完全自定义，默认 mpvnet 风格
- 支持动态菜单：轨道、章节、版本、播放列表、配置文件、音频设备等。
- 支持长标题自动省略号 `…`，并为快捷键、子菜单箭头预留空间。
- Windows / macOS / Linux 均提供文件、文件夹、保存对话框路径。
- 🎯 [预览 / Preview](https://github.com/mpvConfig/mpv-menu-plugin-next#%E9%A2%84%E8%A7%88preview)

## 2. 安装（Installation）

将`src`文件文件夹内容复制 mpv 的配置目录，例如便携版：

- 一般 mpv 安装版（非 Portable 便携版）配置文件目录：

```
Linux:   ~/.config/mpv/
Windows: C:/Users/%username%/AppData/Roaming/mpv/
macOS:   ~/Library/Application Support/mpv/
```

- mpv 便捷版 `mpv.exe` 同目录下，新建目录：`portable_config`

```text
portable_config/
├── scripts/menu/
│            ├── main.lua
│            ├── dyn_menu.lua
|            ├── menu.lua
│            └── dialog.lua
├── script-opts/
│   └── menu_style.conf   # 自定义样式/Style
└── input.conf
```

- 参考来源：[mpv-menu-plugin](https://github.com/tsl0922/mpv-menu-plugin)，由于 mpv-menu-plugin 不支持其他平台，因此新做了这个项目
  > [!IMPORTANT] > **不要再同时加载旧版 `menu.dll` 后端，否则可能出现两个菜单后端同时工作的情况。**

## 配置文档（Configuration）

- [文档：Documentation](https://github.com/mpvConfig/mpv-menu-plugin-next/blob/main/doc/README.md)
- [示例 input.conf example](https://github.com/mpvConfig/mpv-menu-plugin-next/tree/main/example)
- [已使用本插件的 mpv 配置-example for mpv config](https://github.com/akFace/mpv.config)

## 预览（preview）

### 默认风格 Default style

🎨 [menu_style.conf](https://github.com/mpvConfig/mpv-menu-plugin-next/tree/main/src/script-opts/menu_style.conf)

![image](./doc/images/Snipaste_2026-08-14_21-31-47.jpg)
🍵 [中文菜单预览图](https://github.com/akFace/mpv.config#%E5%8A%9F%E8%83%BD%E5%BC%BA%E5%A4%A7%E7%9A%84%E5%8F%B3%E9%94%AE%E8%8F%9C%E5%8D%95)

---

### 磨砂玻璃 menu_style-gray-glass

🎨 [menu_style-gray-glass.conf](https://github.com/mpvConfig/mpv-menu-plugin-next/tree/main/example/script-opts/menu_style-gray-glass.conf)

![image](./doc/images/Snipaste_2026-10-10_15-08-58.jpg)

---

### macos 白色风格 macos-white-style

🎨 [menu_style-macos-white.conf](https://github.com/mpvConfig/mpv-menu-plugin-next/tree/main/example/script-opts/menu_style-macos-white.conf)

![image](./doc/images/Snipaste_2026-08-16_01-04-38.jpg)

---

### macos 深色风格 macos-dark-style

🎨 [menu_style-macos-dark.conf](https://github.com/mpvConfig/mpv-menu-plugin-next/tree/main/example/script-opts/menu_style-macos-dark.conf)

![image](./doc/images/Snipaste_2026-08-16_01-10-14.jpg)

> 如果你有好看的风格配置文件`menu_style.conf`，也可以在 issue 中分享（If you have an attractive style `menu_style.conf`, you can also share it in the issue.）
