# LocalMark

[English](README.en.md) | 中文

本地图片/PDF 水印工具 — 纯浏览器处理，不上传任何文件，断网也能用。

🔗 在线使用：https://krstic-c.github.io/local-mark/

## 特点

- **零上传**：所有图片/PDF 处理都在浏览器本地内存里完成（Canvas API），不经过任何服务器或 API
- **断网可用**：双击 `index.html` 即可离线打开使用，字体、PDF 处理库等全部本地打包，不依赖 CDN
- **文字水印**：支持多行文字、颜色/字号/透明度/描边/混合模式，一键插入日期或日期+时间
- **图片/Logo 水印**：上传本地图片（如公司 Logo、机密图章）作为水印
- **两种布局**：平铺（可调角度、密度，错位排列避免留白）或九宫格单点定位
- **批量处理**：一次拖入多个文件，统一应用水印设置，打包导出 ZIP
- **PDF 水印**：逐页添加水印，支持导出加水印后的 PDF
- **预设模板**：命名保存常用水印方案，随时一键切换；自动记住上次使用的参数

## 使用方法

1. 打开 [在线版本](https://krstic-c.github.io/local-mark/)，或者克隆本仓库后直接双击 `index.html`
2. 左侧拖入图片（JPG/PNG）或 PDF 文件
3. 设置水印文字或上传水印图片，调整颜色、字号、透明度、角度、密度等参数，右侧实时预览
4. 点击「下载当前文件」导出单个文件，或点击「批量导出 ZIP」一次性导出所有文件
5. 常用的水印方案可以在「预设模板」里保存，下次直接加载

## 本地开发

这是一个纯静态单页应用，没有构建步骤。

```bash
git clone https://github.com/Krstic-c/local-mark.git
cd local-mark
python3 -m http.server 8000
```

然后在浏览器打开 `http://localhost:8000`。

也可以完全离线，直接双击 `index.html` 打开使用（部分浏览器对 `file://` 协议下的字体加载有限制，如遇样式异常可改用本地静态服务器打开）。

## 目录结构

```
index.html          单文件应用，包含全部 HTML/CSS/JS
vendor/
  pdf-lib.min.js     本地打包的 PDF 处理库
  jszip.min.js       本地打包的 ZIP 打包库
  fonts/             本地打包的 DM Sans / DM Mono 字体
```

## 隐私说明

本工具不会以任何形式上传、存储或传输你的图片/PDF 内容。仅使用浏览器 `localStorage` 保存水印参数预设（颜色、字号等配置），不涉及文件内容本身。
