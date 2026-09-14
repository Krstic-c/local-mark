# LocalMark 项目说明

本地图片/PDF 水印工具。纯前端单页应用，无后端、无构建步骤。

- 在线地址：https://krstic-c.github.io/local-mark/
- GitHub 仓库：https://github.com/Krstic-c/local-mark （Public）
- 用户文档：[README.md](README.md)（中文）/ [README.en.md](README.en.md)（英文）

## 产品定位

参考"装箱大师"式的单页工具形态。核心卖点是**零上传、可断网使用**——所有图片/PDF 处理都在浏览器本地内存里用 Canvas API 完成，不经过任何服务器/API。这个隐私特性是产品的核心差异化点，改动时不能破坏。

## 技术架构

- **纯静态单文件应用**：`index.html` 一个文件包含全部 HTML/CSS/JS（IIFE 包裹，无框架、无打包工具）
- **第三方库全部本地 vendor 化**，不依赖任何 CDN（保证断网可用）：
  - `vendor/pdf-lib.min.js`（pdf-lib 1.17.1）—— PDF 逐页加水印
  - `vendor/jszip.min.js`（JSZip 3.10.1）—— 批量导出打包 ZIP
  - `vendor/fonts/`（DM Mono 拉丁字符子集 woff2，约 17KB）—— 仅用于滑块数值等少量数据展示场景；正文/标题统一走系统字体栈（PingFang SC 优先）。2026-09-12 改版移除了 DM Sans（详见「视觉设计规范」）
- **水印渲染**：Canvas 2D API，`applyWatermark()` 是核心函数，文字水印和图片水印共用同一套平铺/单点定位逻辑
  - 平铺（tile）模式用**砖块式错位排列**（奇数行整体偏移半格），避免旋转后出现大片对角空隙
  - 平铺密度滑块的 min/max 会跟字体大小动态联动（`updateDensityRange()`），不是固定区间
  - `forEachTileCell()` 对实例总数做了软上限（`MAX_TILE_INSTANCES = 4000`），极端参数组合（大图 + 小水印 + 低密度）不会导致成千上万次绘制调用卡死页面
- **PDF 导出**：把水印画到离屏 canvas → 转 PNG → 用 pdf-lib 逐页 `embedPng` + `drawImage` 叠加到原 PDF 页面上，不是往 PDF 里画矢量文字
  - `exportPdfEntry()` 按页面尺寸缓存已生成的水印 PNG，同尺寸的页面只渲染/嵌入一次，多页 PDF（常见的同尺寸场景）导出明显更快
- **文件加载错误处理**：图片解码失败、PDF 解析失败、`FileReader` 读取失败都会在对应 `entry` 上设置 `error`/`errorMsg`，文件列表里对应项会标红 + 显示 ⚠，批量导出时会被自动跳过并在完成后用 toast 提示跳过了几个
- **离线安装（PWA）**：`manifest.json` + `sw.js`（Service Worker，cache-first 策略）让页面可以被"添加到主屏幕/安装"，首次访问后即使完全断网也能重新打开，与"零上传、可断网使用"的定位直接对应。`sw.js` 顶部的 `CACHE_NAME` 需要在 `index.html`/`vendor/*` 有实质性改动时手动升级版本号，否则回访用户会一直拿到旧缓存
- **持久化**：仅用 `localStorage` 存两类东西——上次使用的参数（`localmark_last_settings_v1`）和命名预设（`localmark_presets_v1`）。**不存储任何文件内容**，图片/PDF 只存在于内存里，刷新页面就没了
- **打码工具**（2026-09-14 新增）：与水印工具完全独立的第二个功能区块，页面上用分隔线隔开放在水印区块下方
  - 独立状态 `redactState`（regions/selectedId/defaultType 等）+ 独立 DOM id（`redact*` 前缀），不读写 `state`/`wm`，改水印代码不用担心影响这里，反之亦然
  - v1 范围：**只支持单张图片，不支持 PDF，不支持批量**——用户明确要求先做最小可用版本，这三项都在「待办事项」里
  - 支持三种效果：纯色块 / 像素化马赛克 / 高斯模糊，在预览图上拖框新建区域，选中后四角把手可缩放、拖内部可移动，`Delete`/`Backspace` 或列表 ✕ 删除
  - **关键设计**：选中某个已有区域时，"打码类型"面板编辑的是**这个区域自身**；没有选中任何区域时，编辑的是"下一个新框的默认值"（`redactState.defaultType/defaultColor/...`）——`syncRedactControlsFromActive()` 负责让面板显示跟这个规则保持一致，新增/清空/加载新图片这些会重置选中状态的地方都要记得调用它（`clearRedactFile()` 里漏调过一次导致面板显示脏状态，已修）
  - 所有效果（`applySolidRegion`/`applyPixelateRegion`/`applyBlurRegion`）都只从 `redactCleanCanvas`（加载图片时创建的一份未处理原图）取样，每次渲染都是"清空重画+按顺序叠加所有区域"，因此移动/缩放/切换类型/撤销都不会累积上一次的效果，不用担心脏状态
  - 像素化：用 canvas 内置缩放做降采样再最近邻放大（不用手写像素循环）
  - **模糊不要用 `ctx.filter = blur()`**（2026-09-14 踩过的坑）：最初 `applyBlurRegion` 用的是 `ctx.filter = blur(radius)` + `drawImage`（向四周多采样一圈再裁剪回原框，`pad` 就是干这个的），语法完全正确、也确实是 MDN 上的标准写法，但实测发现对**已解码的图片内容**（`drawImage(imgEl,...)` 画出来的 canvas）效果极弱——滑块拉到最大（167px）文字依旧清晰可辨，换成直接用 `fillText`/`fillRect` 画的合成内容测试却完全正常（会被完全洗白）。反复排查过是不是离屏 canvas 合成时机、`clip()`、坐标越界的问题，最后确认跟这些都无关，就是 `filter: blur()` 在这套环境下作为 `drawImage` 的滤镜、且源是"图片解码内容"时不可靠（原因未完全查清，怀疑是这套 Chromium 环境对图片来源画面的 filter 光栅化路径有 bug/限制）。**改为跟像素化一样的"缩小再放大+开启双线性平滑"技巧**（`applyBlurRegion` 内部：先把取样区域缩小到 `sw/factor × sh/factor`、`factor = radius/3`，`imageSmoothingEnabled=true` 放大回原尺寸），不依赖 CSS Filter Effects，实测从默认强度到滑块最大值都能正确、渐进地变强，直到完全洗白。**以后要做任何"模糊"效果，都优先用这个降采样技巧，不要指望 `ctx.filter = blur()` 对图片内容可靠**
  - 画布交互坐标：鼠标/触摸事件坐标是屏幕 CSS px，画布内部是原图分辨率 px，两者之间用 `getBoundingClientRect()` 算出的缩放比换算（`getRedactCanvasPos`/`redactDisplayScale`）——大图在预览区被等比缩小显示时这个换算是必须的，不能直接拿 `clientX/clientY` 当画布坐标用
  - 缩放只做了四个角的把手（拖角=对角锚点固定），没有做四条边中点的把手——四角已经能覆盖"改宽改高"的全部需求，边中点把手主要是锦上添花，为了减少代码量先跳过了

## 视觉设计规范

**2026-09-12 起改为"苹果极简风"**（参考 `~/Documents/GitHub/design-style` 仓库的 `references/apple-minimal.md`），替换了此前的"黑白工业风"。**改动 UI 前先看这一节，不要自作主张换风格**——如果又要换方向，先和用户确认清楚再动手（这次是用户主动要求参照苹果规范重做的）。

- 大留白、克制层级：卡片圆角分层（小控件 6-8px / 输入框按钮 10-12px / 面板容器 20px），配合柔和阴影 `--shadow-sm: 0 1px 2px rgba(0,0,0,.04), 0 10px 30px rgba(0,0,0,.06)`，不再是直角硬边框
- 中性色不用纯黑纯白：浅色模式文字 `#1d1d1f` / 次要文字 `#6e6e73`，背景 `#f5f5f7`（页面）/ `#ffffff`（卡片），分隔线用 `rgba(0,0,0,.08)` 极浅 hairline
- **必须支持深色模式**（`@media (prefers-color-scheme: dark)`，跟随系统，没有手动切换开关）：深色下阴影基本不可见，改用更亮的边框/表面色分层区分卡片
- 字体：正文/标题统一走系统字体栈 `"PingFang SC", -apple-system, BlinkMacSystemFont, "Helvetica Neue", Arial, sans-serif`（不再用 DM Sans）；`DM Mono` 只保留给滑块数值等少量"数据"场景，配合 `font-variant-numeric: tabular-nums`
- 唯一强调色仍是红色 `--red: #b23a2e`（深色模式下提亮为 `#ff6b57` 保持可读），延续品牌识别；开关/分段控件用真实的 iOS 风格 switch / segmented control，不用原生 checkbox/radio 直接摆
- 头部原来的工程图纸 title block（PROCESS/NETWORK/STATUS 三栏表格）已替换为一个圆角 `.status-pill` 小标签
- 交互态要是真的：输入框聚焦有边框变色 + `box-shadow` 光晕；按钮/开关/分段控件都有 hover/active/focus-visible
- 文件列表继续用 `IMG` / `PDF` 文字标签代替 emoji 图标；页面里没有 emoji 装饰（这条延续自旧规范，苹果风同样不建议乱用 emoji）
- 移动端断点 900px 切单列；触控目标（`.btn`）在移动端保证 `min-height: 44px`

## 文件结构

```
index.html          单文件应用，全部逻辑在这一个文件里
icon.svg             应用图标（favicon / manifest / apple-touch-icon 共用）
manifest.json         PWA 安装清单
sw.js                 Service Worker（离线缓存，cache-first）
vendor/
  pdf-lib.min.js
  jszip.min.js
  fonts/
    dmmono-latin-400.woff2
    dmmono-latin-500.woff2
README.md            中文使用说明（默认）
README.en.md         英文使用说明
CHANGELOG.md          工作日志
.gitignore
```

## 部署流程

- 部署目标：GitHub Pages，Source 设置为 `main` 分支根目录 `/`（已配置好，无需改动）
- 日常改动流程：本地改 `index.html` → 本地起 `python3 -m http.server` 用浏览器验证 → 展示 diff 给用户确认 → `git commit` → 用户明确说"push"才 `git push origin main`
- push 到 `main` 后 GitHub Pages 会在 1-2 分钟内自动重新构建，无需手动触发

## 环境变量

无。纯静态站点，不涉及任何环境变量或密钥。

## 已知的坑

- **测试环境的浏览器 pane 有偶发的截图/渲染卡顿**（跟本项目代码无关），表现为 `computer` 截图超时或返回陈旧画面；用 `javascript_exec` 确认页面其实还在正常响应，重开一个新 tab（`preview_start`）通常能解决，不要误判为代码死循环
- **同源 localStorage 会跨测试 tab 互相污染**：如果开多个 tab 测试同一个 `http://localhost:xxxx` 来源，`localStorage` 是共享的，上一次测试保存的"上次参数"会自动带到下一个新 tab。测试时如果要干净状态，记得先 `localStorage.clear()`
- **`file://` 协议下部分浏览器对本地字体加载有限制**：离线双击 `index.html` 打开一般没问题，但如果样式看起来不对（字体没生效），建议改用本地静态服务器（`python3 -m http.server`）打开来排查
- **Service Worker 在 `file://` 协议下不生效**：`registerServiceWorker()` 里已经判断了 `location.protocol === 'https:' || location.hostname === 'localhost'` 才会注册，双击打开 `index.html` 时会静默跳过，不用当成 bug
- **改完 `index.html`/`vendor/*` 记得手动升级 `sw.js` 里的 `CACHE_NAME`**（比如 `localmark-v1` → `v2`），否则已经安装过/访问过的用户会因为 cache-first 策略一直吃到旧版本，感知不到更新
- PDF 水印是"画图叠加"不是"矢量文字叠加"——导出的 PDF 里水印是一张 PNG 图片，不是可选中的文字层。目前是有意这样做的（实现简单、效果稳定），如果以后要做"可编辑/可选中的水印文字层"需要重新设计导出逻辑

## 待办事项

以下是 PRD 中标注为 V2、目前未实现的方向，供后续参考：

- [ ] 快捷键支持（如 Ctrl+Z 撤销参数调整）
- [ ] 预设的导出/导入 JSON 配置文件（方便团队共享水印方案）
- [ ] 水印位置支持自由拖拽定位，不只是九宫格
- [ ] 打码工具支持批量多图（v1 只做了单张图片，每张图的打码区域都不一样，要支持批量需要在文件列表间切换时分别维护每张图的 regions，数据结构和交互复杂度都会上一个台阶）
- [ ] 打码工具支持 PDF（需要先把每页转成图片预览再框选，再转回叠加到原 PDF 页面，实现和测试成本明显高于图片）
- [ ] 打码区域支持四条边中点的独立缩放把手（目前只有四角）
- [ ] OCR 自动识别敏感信息（银行卡号/身份证号等）辅助打码——讨论后决定暂不做：只能用 Tesseract.js 这类浏览器端方案（不能用云端 OCR，否则违反"零上传"定位），中文语言包体积大（几 MB 到十几 MB）且识别率对手机截图这类小字/低对比度场景不够可靠；即使做，输出也必须是"候选框 + 用户手动确认"而不能"识别到就自动打码直接导出"，否则漏检风险比没有这个功能更危险
