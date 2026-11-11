# CHANGELOG

## 0.1.23 · 2026-11-08

**设置窗中英文统一使用整理后的布局（用户 m02744）**

- `__ecStyleSettingsWindow` 去掉语言分支：中文 / 英文都使用同一套整理后的设置页——
  左对齐、行距 6、半长暗灰分隔线、常用脚本目录行上下留白、确定按钮居中。
- 原 0.1.21 的「中文回原版紧凑布局」行为撤销；避免用户误以为中英设置页不一致。
- 验收：无头探针确认设置窗 root `alignChildren=left/top`、`spacing=6`、目录行 `margins=[10,8,10,8]`、
  onDraw 生成 3 条 #383840 1px 半长描边；构建 `node --check` 通过。
- 构建/安装一致：2,674,816 字节，SHA256 `37EE156B0BD6FD5A6F0C96572BD30B06A1913A2B122D1E0A586E7C5A9CBB4595`。

## 0.1.22 · 2026-11-08

**按语言重置控件尺寸（中文紧凑 / 英文宽松）+ 缩窄单字母复选框**

- 修复：`__ecPolishLayout` / `__ecPolishPass` 只增不减，英文撑大后切回中文，功能区仍保持英文尺寸。
  - 抛光撑大前把原 `preferredSize` 存到 `node.__ecPolishSaved`；新增 `__ecPolishReset(root)` 递归还原并清标记。
  - `__ecI18nApplyWin` 在翻译前先 `__ecPolishReset(win)`，再按当前语言重新抛光。
    这样中英切换等价于两套布局（中文紧凑、英文按英文文本放宽），不需要复制两份面板。
- 修复：`__ecTextPad("checkbox"/"radiobutton")` 64 → 26，避免「A/P/S/R/T」这类单字母复选框
  被撑到约 72px（重置 tab 顶部一排被拉得很开、连带工作区变宽的问题）。
- 验收：无头探针确认 checkbox「A」仅 31→34px、英文标签按需 115px、`__ecPolishReset` 能完整还原；
  构建 `node --check` 通过；安装文件与 src 一致 2,675,605 字节，SHA256 `BDAB93660E3F7FCADB14FAAB0D11F09103E1D66B6D90AC27CB415BE657A43E72`。

## 0.1.21 · 2026-11-08

**设置窗按语言切换排版（用户 m02538）**

- 中文：恢复原版紧凑设置窗（内容居中、行距 0、无分隔线），信息密度更高。
- 英文：保留当前排版（左对齐、行距 6、半长暗灰分隔线、「常用脚本目录」行上下留白），
  英文变长时不至于挤在一起。
- `__ecStyleSettingsWindow` 先读 `__ecLangGet()` 再决定套哪套样式；打开设置窗时即时生效。
- 验收：中文设置窗为居中紧凑原版样式；切 English 后设置窗为左对齐+分隔线；
  语言可来回切换，`script_settings.json` 最终记录 `lang:"zh"`。

## 0.1.20 · 2026-11-08

**启动语言修复**

- `__ecI18nApplyWin` 应用前先调 `__ecLangEnsure()`：启动时语言值尚未读取，导致
  `__EC_LANG` 为空、首次 `__ecApplyLang(win)` 不翻译 tab1；修复后「语言=en 重开面板」
  导航与 tab1 内容全部英文。
- `showTab` 在切入 tab 后再补一次 `__ecApplyLang(tabs[index])`，覆盖启动时同步构建的 tab1。
- 验收：英文启动 → 重开面板，tab1 显示 Add Keyframe / Center Anchor Point 等英文；
  切回中文后导航与内容即时恢复；构建安装一致（2,673,137 字节，SHA256 `A50D1C17…`）。

## 0.1.19 · 2026-11-07

**设置窗细节 + 全脚本中英双语（用户 m01806）**

- 设置窗分隔线再调暗（`[0.22,0.22,0.25,1]`），长度缩为可用宽度的一半，并在「常用脚本目录」行
  上下加留白，避免线和文字贴太近。
- 新增运行时中英切换：设置窗加「界面语言 / English」勾选；语言存 `app.settings`
  （`MyScript_Settings / lang`），确定后立即生效。
- 新增 `src\i18n.js`（翻译控件 text / helpTip / listbox、dropdown 项 / 窗口标题，保留中文原文
  可来回切）与 `src\i18n-dict.json`（1126 条）；`tools\build.mjs` 把字典注入构建产物，
  `tools\lazy-tabs.mjs` 让懒建 tab 也应用语言，`src\ec-scriptbar.js` 导航/表头文案走 `__ecT()`。
- 英文比中文长，翻译后补一次 `__ecPolishLayout()`，避免设置窗标签/按钮被截断。
- 版本号标签是拼接字符串，单独改成 `__ecT("当前版本：") + "v" + ...` 参与翻译。
- 验收：构建 `node --check` 通过并安装；重开面板后设置窗英文版整体排版正常（无截断），
  取消 English 后导航与设置窗即时恢复中文。

## 0.1.18 · 2026-11-07

**设置窗口左对齐 + 功能块之间的细灰线（用户 m01660）**

- 设置窗口不再是居中排版：兼容层新增 `__ecStyleSettingsWindow()`，在 `popUpWindow.show()`
  前识别设置窗（按「[属性功能]添加关键帧相隔」文案，免责声明弹窗不受影响），把根容器改为
  左上对齐、行内左对齐，并加大行距。
- 根容器绘制三条 1px 灰色分隔线（复选框组 / 关键帧组 / 脚本目录组 / 版本组之间），
  最后与「确定」按钮之间不画线。
- 验收：构建 `node --check` 通过并安装；重开面板后设置窗内容左对齐、三条细灰线正常、无多余分隔线。

## 0.1.17 · 2026-11-07

**合成工具页去掉灰色标题 + 左侧导航卡顿优化（用户 m01535）**

- 合成工具页的「预合成」灰色标题框：原版用 `panel` 承载，EffectCraft 会额外画标题和边框；
  构建时改成 `group`（`tools\patches.mjs`），内容布局不变，标题框消失。
- 左侧导航卡顿：移除了脚本栏条目、加号和导航按钮的 `mouseover/mouseout` 悬停监听及悬停填充。
  EC 只要窗口内任一控件注册鼠标监听，就会在 `mouse_events` 里每帧追踪指针，并在悬停控件变化时
  触发整窗口重新快照；右侧功能面板的原生按钮悬停由 egui 内部处理，所以左侧更卡。现在只在
  表头按下（瞬时高亮）和条目右键时注册 mousedown，鼠标移动不再触发脚本侧快照。
- 验收：构建 `node --check` 通过并安装；重开面板后合成工具页无「预合成」灰框，导航切换/展开正常。

## 0.1.16 · 2026-11-07

**「脚本」栏细节修正 + 脚本信息页署名（用户 m01202）**

- 展开态不再常亮：表头只在鼠标按下瞬间填充蓝色，松开/展开后恢复圆角描边胶囊，避免遮住
  外层容器边框造成「底边与边框之间 1px 缝隙」的观感。
- 外层容器描边改为内缩 0.5px 的 1px 描边，与填充同坐标对齐，消除展开态底部的 1px 缝。
- 折叠箭头由两条线组成的 V 形改成**等边三角形**（外接圆重心定位，展开/收起方向翻转）。
- 「脚本信息」页新增「EC版作者：KOGGUA」与「EC版 GitHub 仓库」按钮，指向
  `https://github.com/KOGGUA-EDIT/gongjuxiangdashi/tree/ec-port`；EC 不允许脚本启动浏览器，
  点击后弹出完整地址供复制（兼容层新增 `__ecOpenUrl`）。
- 设置窗「常用脚本目录」行保留；脚本栏数据仍存 `ToolBoxMaster_ScriptBar`。
- 验收：构建 `node --check` 通过；无头绘制探针确认展开态无蓝色常亮、按下才填蓝、三角边长为 9
  的等边三角形、描边内缩 0.5px；GUI 重开面板后实测展开态、等边三角、脚本信息页署名与链接提示正常。

## 0.1.15 · 2026-11-07

**左侧导航加「脚本」展开栏（用户 m00005）**

- 导航从 listbox 改成自绘 `group` 列（`src\ec-scriptbar.js` 的 `__ecBuildNav`）：选中项蓝底圆角、
  未选中透明、悬停深蓝；「锚点工具 / 常用工具 / 合成工具 / 重置 / 脚本信息」5 项不变，
  在「重置」与「脚本信息」之间插入脚本栏。
- 脚本栏：收起时是圆角蓝描边胶囊「脚本 ∨」；展开后表头变蓝色实心「脚本 ∧」，下方列出已保存脚本
  和「＋」胶囊。点「＋」扫描设置的脚本目录里的 `*.jsx`，可改显示名后加入；条目右键弹出
  「打开所在文件夹 / 重命名 / 删除 / 取消」；左键点击按 `$.evalFile` 运行，脚本自己的窗口在 EC
  里显示为悬浮窗（实测 KCurve 2.0.2 正常浮出）。
- 设置窗口新增「常用脚本目录」行：默认 `%APPDATA%\EffectCraft\Scripts\ScriptUI Panels`
  （EC 的 ScriptUI Panels 目录），可改；「默认」按钮回填。脚本列表存
  `app.settings` 的 `ToolBoxMaster_ScriptBar`（`items` JSON + `dir`）。
- 兼容层：`__ecFitContentWide` 识别新的 `ecNavColumn` 导航列，`__ecDensifyLayout` 跳过
  `__ecFreeLayout` 节点；`tools/build.mjs` 追加注入 `src\ec-scriptbar.js`。
- EC 限制：脚本不能替你启动资源管理器，「打开所在文件夹」改为弹出完整目录/文件路径的提示窗。

**修「设置窗口点确定后关不掉」**

- 根因：设置窗 onClick 里还有一段给 `Button3.text/enabled` 赋值的代码，而 `Button3` 的
  `var` 定义已在删掉的 B01 区间（原版 44332-44406）里，点击时抛 `ReferenceError`，
  执行不到后面的 `popUpWindow.close()`。`tools/patches.mjs` 新增 `D07` 删除原版 2424-2431
  这段死代码后，确定按钮恢复保存 + 关闭。

**验收**：构建 2,489,893 字符，`node --check` 通过；在运行中的 EffectCraft 里重开面板逐项实测：
收起/展开、＋添加 KCurve、改名、右键删除、点击在悬浮窗运行、设置窗目录回填、点确定关闭并写入
`script_settings.json`。

## 0.1.14 · 2026-11-05

- 按用户要求删掉「脚本信息」页的 **AI工具模块设置** 块：`格式:MP4 [Lossless]`、
  `格式:mp3 [Waveform]` 两个只读输入框，以及 `LosslessText`/`WaveformText` 常量和
  四个防止手改的 onChanging/onChange 回调（`_src\工具箱大师-原版.jsx:44249-44286`，共 38 行）。
  这两个值在原版里是给「AI 工具模块」说明输出模板名用的，移植版没有对应功能。

**验收**：新构建 2,469,325 字符；另开干净 EffectCraft 实例（控制口 9878）装载后，
脚本信息页只剩「作者：小庄掌门 / 参与优化脚本 / 灵感来源 / Bilibili / TEXT / mikan / lookae」
与重置等按钮，AI 工具模块设置整块消失，控制台无报错。
## 0.1.13 · 2026-11-05

**修「重置页：全部重置会把图层删掉」「图层样式 on/off 报 cannot convert null to object」**

- 「全部」按钮走 `All_attributes()` → `selectLayerStyles()`，原版在那里先
  `layer.property("ADBE Layer Styles").selected = true` 再 `app.executeCommand(21)`
  （AE 的 Edit ▸ Clear）。EffectCraft 里 21 号命令被映射成 `edit.clear`，而 EC 又不能选中
  「图层样式」这种属性组 → 就退化成**删掉整个图层**，所以点一次「全部」图层全没了。
  改法：`tools/patches.mjs` 把那段整体换成 `__ecClearLayerStyles(layer)`，
  由 `src/compat.js` 直接遍历 `ADBE Layer Styles` 组里的属性逐个 remove；EC 没有这个组时安静跳过。
- 「图层样式 (on/off)」：EC 的图层没有 `layerStyle` 属性（实测 `typeof L.layerStyle === "undefined"`，
  `property("ADBE Layer Styles")` 返回 null），原来直接读 `curL.layerStyle.canSetEnabled` 就抛
  `TypeError: cannot convert 'null' or 'undefined' to object`。现在改成
  `if (curL.layerStyle && curL.layerStyle.canSetEnabled)` + `try/catch`，
  并新增 `__ecToggleLayerStyle(layer)` 兜底：**EC 不支持图层样式，按钮变成安全的空操作**。

**实测（新开的干净 EffectCraft 实例 + 桥）**

| 操作 | 结果 |
| --- | --- |
| 选中纯色层点「全部」 | 图层仍在（`layers=1`，名字未变），变换被重置 |
| 先加 Drop Shadow + 蒙版，点「效果」/「蒙版」 | `fx 1→0`、`masks 1→0`，图层未被删 |
| 缩放 50%、不透明度 40% 后点「重置属性」 | `scale 50→100`、`position→960,540`；不透明度没动（T 勾选框默认未勾，与原版一致） |
| 点「图层样式 (on/off)」 | 无报错（EC 无该 API，安全空转） |
## 0.1.12 · 2026-11-04

**修「点纯色/单帧 报 click handler: TypeError: not a callable function」**

EffectCraft 里没有 `$.colorPicker`（实测 `typeof $.colorPicker === "undefined"`），而原版有 6 处
直接调用它（`_src\工具箱大师-原版.jsx` L10561 / L10624 / L10853 / L10934 / L12547 / L12640，
分别是「纯色/单帧」「批量纯色」等按钮），点下去就抛 not a callable function。

- `src\compat.js` 新增 `$.colorPicker` 替代实现：返回**记忆色**（首次白色），颜色存在
  `app.settings` 的 `工具箱大师EC / lastSolidColor`，重启后仍然记得；按住 **Alt 再点**会弹出
  一个 18 色的小调色板（`__ecShowColorChooser()`），选完记住下次直接用。
  控制台会打印「EC 无系统取色器，使用记忆色 #FFFFFF（Alt+点击可改）」。
- 实测：选中 `Solid 1`(0–2.2022) 点「纯色/单帧」→ 新建 `纯色 4`，入出点 = 0–2.2022，
  控制台无异常；测试层已删。
## 0.1.11 · 2026-11-04

**修「创建空对象报错 + 不按选中图层时间裁剪」**

- 根因：原版多处 
ullLayer.source.name = …，但 ddNull() 出来的空对象 .source 就是 **null**，
  EffectCraft（AE 同理）会抛 TypeError: cannot convert 'null' or 'undefined' to object。
  这个赋值排在 inPoint / outPoint 之前，异常一抛，后面的裁剪就没执行——所以用户看到的
  「报错 + 没裁剪」是同一个 bug。
- 修法：构建期把 9 处 .source.name = X; 换成 __ecSetSourceName(layer, X);，兼容层里带
  if (layer && layer.source) 保护（空对象直接跳过，命名仍由 layer.name 承担）。
- 实测（GUI 桥，Comp 1 里选中 Solid 1 0–2.2022 后点「空对象」）：新建的空对象
  in=0 out=2.2022，跟选中图层时间一致，控制台无报错。
## 0.1.10 · 2026-11-04

- **删掉「填充/描边/两者 + 矩形 + 形状图层 + 标 + 蒙版色 + 转移」整块**（原版 8218–8282，
  65 行）；unction sortLayers() 保留。控件删掉后三处 onClick 赋值（xingzhuangButton、
  markButton4、maskcoloButton5）改用 if (0) … 短路，避免重载时报 undefined。
  File.decode( 期望数 37→35。
- **内容左对齐**：EC 的列容器默认把子控件居中，原版 AE 是左上对齐。这一版只把
  depth>=3 的容器改成 lignChildren = ["left","top"]（行容器 ["left","center"]），
  上层（窗口 / 导航行 / 内容外壳）仍保持 fill——上一轮全局左对齐会把整列压到 180px。
  实测「常用工具 → 创建」页内容现在从左边 ~10px 起排，「时间重映射」那一行贴到边。

产物 2,466,446 字符；5 个页 check-layout.ps1 全 OK。
## 0.1.9 · 2026-11-04

按用户第 9 轮要求：

- **banner 去掉按钮底与选中态**：顶部 logo 原来是 iconbutton（EC 会画按钮底和聚焦高亮框），
  改成 image 控件，只显示透明 PNG；原来点它有「常用效果」窗口，现在这个入口没了。
- **缓动图标加回来**：D02 删除区间由 7376–7657 收回 7376–7517，KEYGroup 的 9 个
  缓动按钮（缓入/线性/缓出/定格入点/线性定格入点/定格/线性定格出点/定格出点/圆滑）
  连同 onClick 重新保留，只去掉时间/帧速率那条。
- **「创建」页去掉对不上的外框**：layerGroup 原本是 panel，只包住前 5 行；后两行
  （300/动态拼贴/JAWS、普通/等比）在原版靠绝对坐标落在框内，EC 的流式布局必然排到框外，
  所以把这个 panel 改成 group，去掉那条包不住内容的细边。
- 试验过把容器统一改左对齐（lignChildren=["left","top"]），会把整列压窄、效果更差，已回退。

**实测**：产物 2,474,488 字符；5 个页 check-layout 全 OK；banner 截图确认无底色阴影。
## 0.1.8 · 2026-11-04

**修掉「分段/统一」「父子级/自定义命名」被压成竖排**

根因是 EffectCraft 的 checkbox / radiobutton 渲染缺陷：标签区域只按勾选框本身的宽度算，
preferredSize / size 给到 100~300px 都一样，中文会被拆成一列一个字
（实测探针：父/子/级/对/象、对/象、ABCDEFGH 同样折行）。这不是布局参数能救的，
所以在构建期把这些控件拆成「空控件 + 独立静态文字」：

- 新增 	ools\labelify.mjs：X = P.add("checkbox"|"radiobutton", <bounds>, "文字")
  → X = __ecLabeled(P, "checkbox", "文字")，本版共转换 **24 处**（15 勾选框 + 9 单选框）；
  EC_TOOLBOX_NO_LABELIFY=1 可关。
- src\compat.js 新增 __ecLabeled(parent, kind, text)：建一个 row 组，勾选框不带文字，
  旁边放 statictext 标签，.value / .enabled 用法完全不变。
- 顺带把容器（group/panel/tab）不再写死 preferredSize（原版手调的整流宽度会把中文挤行），
  让容器按内容自适应。

**实测**：合成工具页 ☑ 父子级对象  ☐ 自定义命名 单行显示；常用工具页
● 分段 ● 统一、● 填充 ● 描边 ● 两者 单行显示；设置窗口标签与「是/否」不再重叠；
产物 2,454,499 字符；5 个页 check-layout.ps1 全 OK。
## 0.1.7 · 2026-11-04

按用户第 7 轮截图/文字要求做的五项改动：

- **设置窗口不再压字**：根因是 	ools\flowify.mjs 的词法器不认正则字面量——原版
  _src\工具箱大师-原版.jsx:1159 的 obj.replace(/"/g, '\\"') 让扫描器把 / 后面的
  " 当成字符串开头，从那行起整片代码被误判成字符串，后面 100+ 处
  .add("控件", [l,t,r,b], …) 全部没被转换（全文件只转了 174 处）。给扫描器加了
  __looksLikeRegex() / __skipRegex() 后转换数 174→185，设置窗口的静态文字与复选框
  不再上下重叠。
- **删掉「模糊过渡」页**：nav 6→5，__ecTabOrder 改为 [0,1,2,7,12]。
- **常用工具只保留「创建」**：删掉子页导航（✔创建/其它/小工具）与「其它」「小工具」两个
  子页本体（约 3400 行），subTabs/showSubTab 一并不再需要。
- **锚点工具删掉整条底栏**：分/秒/帧输入、选定图层时间、time、帧速率，以及那排 9 个
  缓动图标按钮（缓入/线性/缓出/定格…，原 KEYGroup）；后面的 unction Smooth() 保留。
- **顶部 banner 去黑底**：新增 	ools\banner.mjs（xtract 从原版字面量还原 PNG、
  ncode 把处理后的 PNG 编回 \xNN 字面量 → 	ools\banner-transparent.json）；
  透明化规则 lpha = clamp((max(r,g,b) - 6) * 1.08)，保留描边光晕；构建器第 3.4 步
  自动替换，EC_TOOLBOX_NO_BANNER=1 可关。
- 修 pp.scheduleTask 垫片：EC 的 scheduleTask 是宿主对象，往上挂 __ecOriginal
  自定义属性在面板作用域会丢，导致每次布局抛光都抛
  scheduleTask 失败：$.global.__ecPolishAgain();；改为模块级变量保存原函数。

**实测**：产物 2,454,039 字符；面板 47 个节点（原 57）；导航 5 项；
check-layout.ps1 -AllTabs 5 页全部 OK；设置窗口标签/输入框分行不重叠。
## 0.1.6 · 2026-11-04

- 左侧导航列表宽度 110→94：把 20px 让给右侧内容列。常用工具页的三列小按钮在 417px 的
  停靠列里正好卡在「差一点点放不下」的边缘，窄一点导航后按钮可以按文字宽度排开。
- 	ools\check-layout.ps1 -AllTabs 复查：6 个页仍然 0 重叠。

**仍不理想的**：「常用工具」页的「创建 / 小工具」区块、「合成工具」页的「父子级 / 自定义命名」、
「模糊过渡」页的若干标签，原版是按绝对坐标拼版的（标签在控件上方、局部还有互相叠压），
EC 的流式布局只能按顺序排，做不到像素级还原。要彻底干净只能把这几块按行重写成规整网格。
## 0.1.5 · 2026-11-04

**移除中转站残留 UI + 修「标题/文字压住控件」**

- 按用户要求把中转站相关 UI 整个摘掉（EC 起不了外部程序，先删、以后修好再加回来）：
  设置窗口里的「中转站路径」整块（含选择路径、校验、删除默认目录等 4 个私有函数，
  共 490 行）删除；启动时的 checkToolBoxMasterPath() 不再弹路径窗口；
  确定按钮里的保存路径分支加了 pathInput 空值保护。
- 带标题的 panel/group 统一加 20px 上边距：EC 把标题画在容器内部左上角却不给布局留位置，
  「小工具」「预合成」「普通模糊」这些标题原本会压在按钮/复选框上。
- 按钮/勾选框/单选钮最小高度抬到 24px（原版有 15px 高的「应用」，文字会被裁）；
  复选框/单选钮按「方框 + 文字」加宽（+46px），不再让文字压到方框上。
- 设置弹窗也是运行期临时建的，popUpWindow.show() 前补一次布局抛光。
## 0.1.4 · 2026-11-04

**按需求裁掉一整块界面 + 7 个页**

用户要求：删掉面板底部两行按钮，并删掉「叠层 / 添加预设 / 常用 SFX / GPT改图 /
超分抽补帧 / 抠图等程序 / SHAKE 工具」这 7 个页。

- 底部两行（caijiangroup 的 8 个 iconbutton + 其 15 个裁剪辅助函数、Attributesgroup
  的 A P R S T U 隐藏 与 sendPP）整块删除，共 329 行；purge1（重置页在用）
  与 syncWorkAreaToComp（7 处在用）保留。
- 左侧导航只剩 6 项：锚点工具 / 常用工具 / 合成工具 / 模糊过渡 / 重置 / 脚本信息；
  
av.onChange 经 __ecTabOrder = [0,1,2,6,7,12] 映射到原 tab 索引。
- 被删的 7 个页仍由懒构建包成 builder，只是永远不会被调用（不可达，无运行时开销）；
  没有物理删除它们的 2 万多行代码，避免影响共享函数与上提变量。
- 面板拿掉了底部两行后 root children 由 4 → 2，首帧节点数 92；6 个页全建后 327 节点、
  树 JSON 744 KB（原来 13 页 1,385 KB）。

**版式再调**：中文宽度估算 12.5→13.6px、按钮内边距 20→24px（时间重映射 这类
4~5 字按钮不再裁字）；check-layout.ps1 -AllTabs 改为按导航实际项数扫描，6 个页全部 0 重叠。

**已知**：常用工具页仍是密集小格子（原版就是 46 个 20~60px 的小按钮拼贴），
在 EC 的流式布局下只能做到「不重叠、不裁字」，做不到 AE 那种像素级拼版。
## 0.1.3 · 2026-11-04

**布局：把 AE 的绝对坐标改成 EffectCraft 的流式布局**

EffectCraft 的布局引擎不把子控件的绝对坐标算进容器尺寸，固定坐标的控件也不推进行光标
（`crates/engine/src/scriptui.rs` 的 `preferred()` / `place()`），所以原版那 197 处
`add("控件", [l,t,r,b], …)` 直接搬过来会互相压字、被裁切。这一版做三件事：

- 新增 `tools\flowify.mjs`：构建期把 `.add("控件", [l,t,r,b], …)` 改写成 `undefined`，
  并在语句后补 `NAME.preferredSize = [w,h]`（跨行调用也能识别，字符串/注释里的内容跳过；
  共转换 174 处，跳过 3 处无可赋值目标）。`EC_TOOLBOX_NO_FLOW=1` 可关掉。
- `.size = […]` 一律改写为 `.preferredSize = […]`（6 处）。EC 的 `size` setter 写的是绝对
  坐标 `__ub`，会把控件全堆到父容器左上角——这正是 SHAKE 页三个下拉框重叠 900px² 的原因。
- 面板建好后 `__ecPolishLayout` / `__ecDensifyLayout` 做三件收尾：
  ① 中文按钮按文字宽度撑开（EC 估宽度是「字符数 × 7px」，中文会短 40% 以上，导致逐字折行）；
  ② 容器 spacing 10→3、margins 收到 4；
  ③ 顶部容器 `alignChildren = ["fill","fill"]`，并按实际宽度把左侧导航行右侧的
  内容容器撑满（否则三列按钮各只有 60px，`分解文本` 会被折成两行）。
  懒构建的 tab 在 `showTab` 里再补一轮同样的抛光。

**实测**：`tools\check-layout.ps1 -AllTabs` 扫描可见控件的兄弟矩形相交，tab1–tab13 除
tab5 的 `overlayGroup`（`orientation="stack"`，本来就是叠加显示）外全部 0 重叠；
此前 tab3 有 7 处、tab2 有满屏压字。

## 0.1.2 · 2026-11-04

**性能：13 个 tab 改为懒构建**

EffectCraft 每帧都会 clone + layout 整棵 ScriptUI 控件树（实测 ≈0.4 ms/控件/帧，隐藏的 tab
同样计价）。原版在面板加载时把 13 个 tab 的 ~800 个控件一次建完，单次布局 300 ms 量级。
现在面板加载只建导航 + tab1，tab2…tab13 在首次切换过去时才创建。

- 新增 `tools\lazy-tabs.mjs`，由 `tools\build.mjs` 在 flowify（流式布局）之后、注入兼容层之前调用：
  把每个 tab 块包成 `__ecTabBuilders[N] = function () { if (tabN) return tabN; … }`，
  首次切换时由 `showTab` 调用并缓存到 `tabs[index]`。
- 作用域保持与原版一致：块内顶层 `var` 集中声明（373 个名字），块内顶层函数声明
  提升到 `DockableUI` 作用域（328 个），块内去掉 `var`/`function` 关键字；
  字符串、注释、正则里的内容不会被误伤（词法扫描，不是裸正则）。
- tab1 保持加载即建（默认可见，且其块内内联大量公共函数）；tab2…tab13 懒建。
- `EC_TOOLBOX_NO_LAZY=1` 时整步跳过，产物退化为全量构建（对照测试用）。
- `EC_TOOLBOX_DUMP=<路径前缀>` 可在变换前落盘拼装文本，便于排查。

**实测（同一份 compat / flowify，EffectCraft 0.21.0，控制端口 9877）**

| 指标 | 全量构建 | 懒构建 |
| --- | --- | --- |
| 面板加载后控件节点 | 512 | **91**（−82%） |
| `scriptui.get` JSON | 1,385,160 字符 | **233,959 字符**（−83%） |
| 13 个 tab 首轮切换 CPU（净） | 2.900 s | **1.838 s**（−36.6%） |
| 13 个 tab 第二轮切换 CPU（净） | 2.627 s | 2.530 s（已全建，持平） |

- 验收：两轮 26 次切换全部 `ok`，第二轮各 tab 容器 id 与第一轮完全相同（13/13，不重复构建）；
  13 个 tab 的第一个按钮逐个真点一遍，无超时/报错，面板存活；`window.restoreScriptPanels`
  的 `failed` 为空。
- 风险扫描（跨 tab 变量引用）0 命中；各 tab 首次构建后控件总数与全量构建完全一致（512）。

## 0.1.1 · 2026-11-04

- 原版 7 处「`app.findMenuCommandId(name)` → `app.executeCommand(id)`」拿到的是 EC 自己的命令编号，
  之前被命令映射层当未知 AE ID 忽略；现在未收录的数字会透传给 EC 原始实现，中文菜单名走名称兜底表。
- `app.executeCommand(0)`（findMenuCommandId 未命中时的返回值）显式忽略，不再落到引擎下标 0。

## 0.1.0 · 2026-11-04

首个可装载版本（EffectCraft 0.21.0）。

**移植基础设施**
- `tools/build.mjs`：以原版 45,373 行脚本为输入，按补丁表删除/替换后注入兼容层，输出单文件面板脚本；
  带 `node --check` 语法检查、命中数断言、悬空标识符断言（被删函数的引用必须为 0）。
- `tools/patches.mjs`：19 个删除区间 + 14 条正则替换，原版的全部改动都记录在这里。
- `tools/install.ps1`：一键构建并安装到 `%APPDATA%\EffectCraft\Scripts\ScriptUI Panels`。
- `tools/ec-run.ps1`：通过 EffectCraft 控制端口在运行中的 GUI 里执行 jsx（自动化调试用）。

**兼容层 `src/compat.js`**
- `File.decode(x)` → 直传二进制图像串（EC 的 `decode` 是 `decodeURIComponent`，会抛 `UriError`）。
- `$.fileName` → `__ecScriptPath()`（EC 只返回文件名）。
- `Folder.userData` 赋值、`Folder.temp/desktop/myDocuments` 替换（EC 里是不可重定义的 null getter）。
- `alert` → 非阻塞小窗 + 写控制台；`confirm` → 恒真；`prompt` → 返回默认值。
- `app.scheduleTask` 包装：把回调函数挂到 `$.global` 再调度（EC 的定时任务在独立作用域求值）。
- `system.callSystem` → 外部命令降级层（EC 禁止启动进程）。
- `$.sleep` 忙等实现（EC 的实现不睡）。

**命令映射 `src/ec-cmdmap.js`**
- 26 个 AE 数字命令 ID 全部改走 EffectCraft 引擎命令，EC 没有对应命令的用脚本等价实现。

**外部工具 `src/ec-launch.js`**
- 中转站的 8 个外部工具在 EC 下无法拉起（`system.callSystem` 被禁、`File.execute()` 只走浏览器
  open_url 不创建进程），降级为「命令记入 `%APPDATA%\EffectCraft\工具箱大师\待运行命令.txt`
  + 弹窗提示手动运行 + 中断原版轮询」，不再让面板卡在等待状态。
- 删除类命令（`cmd /c rd|del`）改为脚本内直接删；`explorer /select` 改为提示文件所在目录；
  URL 类改为提示在浏览器打开。
- 另外原版还有 22 处直接 `new File(x).execute()`，同样不会创建进程，已把
  `File.prototype.execute` 接到同一降级流程（实测 EC 的 `File.prototype` 可写）。

**启动弹窗**
- 修复「中转站路径设置」窗口关不掉的问题：原版 `onClose` 会 `scheduleTask` 重新拉起自己，
  而 EC 的定时任务在独立作用域求值（`ReferenceError`）导致循环。兼容层把回调搬到 `$.global`
  调度，并在已有有效 `ToolBoxMasterPath.txt` 时短路掉这次重弹。

**裁剪**
- 按用户确认删除语音转文字 / 深度分析（含新版）/ 保留人声三块功能，共 2,896 行；
  产物中被删函数引用为 0。

**验证**
- 面板在 EffectCraft 里可停靠打开，13 个标签页全部可切换，55 个内嵌按钮图正常渲染；
  锚点类按钮（锚点居中 / 整体居中 / 适合画面）点击不报错。
