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
  - `vendor/fonts/`（DM Sans、DM Mono 拉丁字符子集 woff2，从 Google Fonts 下载后本地引用，约 54KB）—— 中文字符走系统字体（PingFang SC / Microsoft YaHei），未引入中文字体包（Noto Sans SC 体积太大，不划算）
- **水印渲染**：Canvas 2D API，`applyWatermark()` 是核心函数，文字水印和图片水印共用同一套平铺/单点定位逻辑
  - 平铺（tile）模式用**砖块式错位排列**（奇数行整体偏移半格），避免旋转后出现大片对角空隙
  - 平铺密度滑块的 min/max 会跟字体大小动态联动（`updateDensityRange()`），不是固定区间
- **PDF 导出**：把水印画到离屏 canvas → 转 PNG → 用 pdf-lib 逐页 `embedPng` + `drawImage` 叠加到原 PDF 页面上，不是往 PDF 里画矢量文字
- **持久化**：仅用 `localStorage` 存两类东西——上次使用的参数（`localmark_last_settings_v1`）和命名预设（`localmark_presets_v1`）。**不存储任何文件内容**，图片/PDF 只存在于内存里，刷新页面就没了

## 视觉设计规范

黑白工业风，参考工程图纸美学。**改动 UI 前先看这一节，不要自作主张换风格。**

- 不用渐变、阴影、圆角卡片这类"互联网范"装饰，`border-radius` 一律为 0
- 字体：英文/数字用 `DM Sans`（正文标题）/ `DM Mono`（数值、标签、按钮文字），中文走系统默认无衬线
- 唯一强调色是红色 `#b23a2e`（CSS 变量 `--red`），只用在"批量导出 ZIP"主 CTA 按钮和少量点缀（文件列表激活态左边框、进度条等），不要引入其他彩色
- 头部有一个仿工程图纸 title block 的信息条（PROCESS / NETWORK / STATUS）
- 文件列表用 `IMG` / `PDF` 文字标签代替 emoji 图标；页面里没有 emoji 装饰

## 文件结构

```
index.html          单文件应用，全部逻辑在这一个文件里
vendor/
  pdf-lib.min.js
  jszip.min.js
  fonts/
    dmsans-latin.woff2
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
- PDF 水印是"画图叠加"不是"矢量文字叠加"——导出的 PDF 里水印是一张 PNG 图片，不是可选中的文字层。目前是有意这样做的（实现简单、效果稳定），如果以后要做"可编辑/可选中的水印文字层"需要重新设计导出逻辑

## 待办事项

以下是 PRD 中标注为 V2、目前未实现的方向，供后续参考：

- [ ] 快捷键支持（如 Ctrl+Z 撤销参数调整）
- [ ] 预设的导出/导入 JSON 配置文件（方便团队共享水印方案）
- [ ] 水印位置支持自由拖拽定位，不只是九宫格
