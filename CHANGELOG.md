# CHANGELOG

（此前的历史未记录，当前状态见 [CLAUDE.md](CLAUDE.md)）

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
