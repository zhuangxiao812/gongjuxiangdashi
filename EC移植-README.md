# 工具箱大师 · EffectCraft 移植版

基于原版 `工具箱大师.jsx` v1.0.123（AE ScriptUI 面板）移植到 **EffectCraft / ArtCraft**（0.21.x）的版本。
不是一个重写，而是**同一份脚本 + 兼容层 + 若干裁剪**，所以界面、按钮图、交互习惯尽量与原版一致。

当前版本：**0.1.15**（原版 v1.0.123 派生）

## 安装

1. 把 `工具箱大师-EC移植.jsx` 复制到：
   ```
   %APPDATA%\EffectCraft\Scripts\ScriptUI Panels\
   ```
2. 打开 EffectCraft，在 **窗口** 菜单里打开「工具箱大师」。
   - 面板是 dockable 的，可以拖进任意分栏。
   - 改了脚本要**关掉面板再重新打开**，EC 不会热重载面板脚本。

## 与原版的差异（都会在面板里直接看到）

| 项目 | 原版（AE） | EC 移植版 |
| --- | --- | --- |
| 启动 | 检查 `ToolBoxMasterPath.txt`，没有就弹路径设置窗口 | 不检查、不弹窗（中转站工具已移除） |
| 顶部 banner | 按钮样式（带底和选中高亮） | 纯 PNG（透明底，无按钮状态） |
| 左侧导航 | 13 个页 | **5 个页**：锚点工具、常用工具（只剩「创建」）、合成工具、重置、脚本信息 |
| 锚点工具页 | 时间/帧速率输入 + 走带按钮 + 缓动图标 | 只保留缓动图标那排（时间/帧速率/走带已删） |
| 常用工具页 | 创建 / 其它 / 小工具 三个子页 | 只保留「创建」；「填充/描边/两者 + 形状图层」那一块已删 |
| 模糊过渡页 | 有 | 已删 |
| 中转站工具（补帧、超分、抠图、GPT改图、SFX、场景裁剪、查找相似、yt-dlp） | 调用外部 exe | **已移除**（EC 脚本无法启动外部程序，见下） |
| 外部程序 | `system.callSystem(...)` | 被禁用；调用点改为「记录命令 + 弹窗提示」 |
| 取色 | 系统取色器 `$.colorPicker` | 记忆色 + Alt+点击打开 18 色调色板 |
| 逐帧渲染 / 输出模块 | `renderQueue` 模板 `Lossless` / `PNG Sequence` | EC 只有 `H.264 / ProRes / PNG Sequence / …`，相关导出功能受限 |

## 兼容层做了什么

原版脚本里大量 AE 专属 API 在 EC 上形态不同，移植版在脚本最前面注入三层垫片：

- `compat.js`：`File.decode`（EC 是 `decodeURIComponent`，会把 PNG 字节里的 `%` 当转义）、`$.fileName`（EC 只给文件名，改成自定位）、`Folder.temp/desktop/myDocuments/userData`（EC 里是 null getter / 未定义）、`alert/confirm/prompt`（EC 里不可见 / 恒真 / 返回默认值）、`app.scheduleTask`（EC 的定时任务在独立全局作用域求值）、`$.colorPicker`、`__ecSetSourceName()`（空对象没有 `.source`）、`__ecClearLayerStyles()/__ecToggleLayerStyle()`（EC 没有图层样式 API）、布局抛光（把按文字宽度过窄的按钮撑开、给带标题的面板留出标题高度）。
- `ec-cmdmap.js`：`app.executeCommand()` 的 26 个 AE 数字 ID → EC 命令（`findMenuCommandId("Easy Ease")` → `keys.easyEase` 等），未收录的 ID 透传给 EC 自己的命令编号。
- `ec-launch.js`：原版所有 `system.callSystem()` 的出口。删除类命令同步执行；启动外部程序类命令只记录到
  `%APPDATA%\EffectCraft\工具箱大师\待运行命令.txt` 并弹提示。

另外构建期做了两件事：

- **绝对坐标 → 流式布局**：AE 的 `add("按钮", [l,t,r,b])` 在 EC 里会互相压字（EC 的 `preferred()` 不算子控件绝对坐标、`place()` 对固定位置子控件不推进行光标），构建时统一换成 `undefined` + `preferredSize`。
- **tab 懒构建**：面板加载时只建导航 + 第一个页，其余页首次切入时才建（EC 每帧重排整棵控件树，13 页全建会明显拖慢）。

## 已知限制（EC 0.21.x）

1. **脚本无法启动外部程序**：`system.callSystem` 在 EC 里直接抛错，`File.execute()` 走的是
   `open_url`，`file://` 打不开（会变成浏览器下载）。所以中转站那 8 个工具包功能全部移除。
   等 EC 开放进程启动能力后可以接回来（预留了 `ec-launch.js`）。
2. **没有系统文件选择器**：`File.openDialog` / `Folder.selectDialog` 恒返回 `null`，凡是需要选目录的地方改为手填路径。
3. **没有真实取色器**：`$.colorPicker` 未实现，用记忆色 + 内置调色板代替。
4. **图层样式 API 缺失**：`layer.layerStyle` 是 `undefined`，相关按钮安全空转并打印日志。
5. **`app.settings` 可用但无弹窗**：参数保存在 EC 的脚本设置里，行为与 AE 一致。
6. **版式**：EC 的布局器与 AE 不同，密集页面（原版用绝对坐标拼的）只能做到「不重叠、不裁字」，
   做不到像素级还原；停靠列窄于 ~430px 时会更挤。
7. **渲染导出**：EC 的输出模块模板与原版 AE 不同，逐帧/深度图/序列帧类导出需要按 EC 模板改写，部分功能不可用。

## 修复过的原版遗留问题

- 新建空对象时会抛 `TypeError: cannot convert 'null' or 'undefined' to object` 并且不裁剪 →
  空对象没有 `.source`，改名赋值包了保护，现在会按选中图层的时间裁剪。
- 「纯色/单帧」抛 `TypeError: not a callable function` → `$.colorPicker` 缺失。
- 重置页「全部重置」会直接把图层删掉 → AE 的 `executeCommand(21)` 在 EC 里是 `edit.clear`，
  改为遍历图层样式组删除。
- 图层样式 on/off 抛错 → EC 无该 API。

## 从源码构建（可选）

移植版自带一套构建管线（Node.js ≥ 18）：

```
tools/build.mjs     原版 jsx → 删除区间 → 文本替换 → 流式布局 → 懒构建 → 注入兼容层
tools/flowify.mjs   绝对坐标 → preferredSize
tools/labelify.mjs  checkbox/radiobutton 文字拆成独立 statictext（EC 的勾选框标签会竖排）
tools/banner.mjs    顶部 banner 去黑底（透明 PNG）
tools/install.ps1   构建并复制到 EC 的 ScriptUI Panels 目录
```

## 许可

移植代码（兼容层与工具脚本）随原脚本；原脚本版权归原作者（@小庄掌门）。
