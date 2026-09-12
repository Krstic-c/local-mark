# CHANGELOG

（此前的历史未记录，当前状态见 [CLAUDE.md](CLAUDE.md)）

## 2026-09-12

- **视觉风格由"黑白工业风"全面改为"苹果极简风"**（用户要求参照 `design-style` 仓库的 `apple-minimal.md` 规范重做）：圆角分层（6-24px）+ 柔和阴影替换直角硬边框；中性色改用 `#1d1d1f`/`#6e6e73`/`#f5f5f7` 而非纯黑白；正文/标题字体改为系统字体栈（PingFang SC 优先），移除 `DM Sans`（`vendor/fonts/dmsans-latin.woff2` 已删除），`DM Mono` 保留给滑块数值等数据展示场景；开关/分段控件改为真实的 iOS 风格组件；新增深色模式（跟随系统 `prefers-color-scheme`）；头部工程图纸 title block 换成圆角 status pill。红色强调色 `#b23a2e` 保留，延续品牌识别。
- **修复功能 Bug**：切换到"图片/Logo 水印"模式后，水印的缩放/透明度/混合模式滑块此前会跟着"文字样式"面板一起隐藏，导致用户完全无法调整图片水印的大小和透明度。现在拆分为"水印样式"面板，仅隐藏文字专属的颜色/描边控件，缩放（原字号滑块，label 动态改为"水印缩放"）/透明度/混合模式两种模式下都可见可调。
- **健壮性修复**：图片解码失败、PDF 解析失败、`FileReader` 读取失败现在会在文件列表里标红并显示 ⚠ 图标（此前静默失败，用户无感知）；批量导出时被跳过的文件数量会在完成后通过 toast 提示；预设重名保存前会二次确认，避免误覆盖；新增"清空全部"按钮清空文件列表；全部原生 `alert()` 替换为非阻塞的 toast 提示。
- **性能优化**：PDF 导出时按页面尺寸缓存已生成的水印 PNG，同尺寸页面只渲染/嵌入一次（多页 PDF 明显更快）；平铺水印的绘制实例数增加软上限（`MAX_TILE_INSTANCES = 4000`），避免极端参数组合（大图+小水印+低密度）卡死页面。
- **新增离线安装支持（PWA）**：新增 `manifest.json` + `sw.js`（Service Worker，cache-first 缓存策略）+ `icon.svg`，页面可被浏览器安装/添加到主屏幕，首次访问后断网也能重新打开，呼应产品"零上传、可断网使用"的定位。
- **无障碍/细节修复**：拖拽上传区、九宫格定位按钮、开关、分段控件补充键盘操作与 `aria-*` 属性；水印图片预览补充 `alt`；图片上传新增支持 WEBP 格式；新增页面 `favicon`/`<meta description>`。
- 影响范围：仅 `index.html` + 新增的 `icon.svg`/`manifest.json`/`sw.js`，`vendor/fonts/dmsans-latin.woff2` 已删除；未改动水印渲染算法本身（平铺/九宫格定位逻辑、PDF 叠加方式均不变）。

## 2026-08-31

- 页脚署名格式调整为 `Design by Krstic - 2026`（CSS `text-transform: uppercase` 会自动渲染成全大写）。影响范围：仅 `index.html` 页脚展示。
- 新增中英文 README（`README.md` 中文默认、`README.en.md` 英文），互相带语言切换链接。为什么：仓库已设为 Public，需要有面向使用者的文档。
- 页脚新增 "Design by Kristic" 署名（同日后续改为 "Krstic"，见上）。
- 新建 GitHub 仓库 `Krstic-c/local-mark`（Public），完成首次 push，并开启 GitHub Pages（Source: `main` 分支根目录），线上地址 https://krstic-c.github.io/local-mark/ 。为什么：需要一个可以直接分享给别人使用的网页链接，免费方案下 GitHub Pages 只支持 Public 仓库。
- 按用户提供的"黑白工业风"设计规范（DM Sans/DM Mono 字体、单一红色强调色 `#b23a2e`、无圆角无阴影、仿工程图纸 title block）重做整体视觉风格，替换原先的通用互联网风格配色。字体本地 vendor 化（下载 DM Sans/DM Mono 拉丁子集 woff2），保持断网可用不依赖 CDN。
- 修复平铺水印在小字号时出现大片对角空隙的问题：平铺间距的可选范围改为跟字体大小动态联动，并把平铺算法从规则网格改为砖块式错位排列。
- 头部锁图标由 emoji 换成线条风格 SVG。

## 2026-08-30

- 项目从零搭建：纯前端单页水印工具，`index.html` 单文件 + 本地 vendor 化的 pdf-lib（PDF 逐页水印）与 JSZip（批量导出）。
- 实现 PRD 中列出的全部 V1 功能：文字水印（多行、颜色/字号/透明度/描边/混合模式）、图片/Logo 水印、平铺与九宫格单点两种布局、批量拖拽上传与 ZIP 导出、PDF 逐页水印、预设模板保存/加载（`localStorage`）、实时预览。
- 本地 git 仓库初始化，完成首次 commit。
