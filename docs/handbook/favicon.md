# 更换网站图标（Favicon）

Chirpy 的默认图标存放在 `assets/img/favicons/`。以下是生成并替换为自己的图标的完整流程。

## 设计方向

Favicon 的核心设计约束：

- 在小尺寸下（16×16 到 32×32）必须清晰可辨
- 极度简约，不要文字
- 与博客主题（「数字漫游」— 漫无目的地探索和折腾）契合

## 1. 生成源图（AI 生图）

使用 AI 生图工具（DALL·E、Midjourney、Ideogram 等）生成一张 **正方形**、**至少 1254×1254** 像素的源图。

以下是本博客使用的 prompt 模板，设计概念为「信号波形 + 探索小径」的蜿蜒曲线：

```
A minimalist flat icon, white on deep navy (#0f172a) background. A single continuous winding line that resembles both a signal waveform and a mountain trail, starting from bottom-left and ending at top-right. The line is composed of tiny glowing dots (like data packets or footsteps), with 4-5 small glowing nodes along the path. Clean vector illustration style, bold stroke weight, zero text, no shading, no gradient — flat design suitable for extreme downscaling to 16x16 pixels. The composition should be perfectly centered and symmetrical in weight. App icon style, duotone, sharp edges, pixel-crisp.
```

> **提示**：建议同时生成暗色背景和亮色背景两个版本，分别缩小到 16px 对比，选择更清晰的一方。

## 2. 生成图标文件（ImageMagick）

源图保存为 `favicon.png`，用 ImageMagick 一键生成所有需要的尺寸：

```shell
# 生成各尺寸 PNG
magick favicon.png -resize 96x96 assets/img/favicons/favicon-96x96.png
magick favicon.png -resize 180x180 assets/img/favicons/apple-touch-icon.png
magick favicon.png -resize 192x192 assets/img/favicons/web-app-manifest-192x192.png
magick favicon.png -resize 512x512 assets/img/favicons/web-app-manifest-512x512.png

# 生成多尺寸 .ico（16, 32, 48）
magick favicon.png -define icon:auto-resize=48,32,16 assets/img/favicons/favicon.ico

# 生成 SVG（内嵌 PNG）
echo "<svg xmlns=\"http://www.w3.org/2000/svg\" xmlns:xlink=\"http://www.w3.org/1999/xlink\" width=\"96\" height=\"96\" viewBox=\"0 0 96 96\">
  <image width=\"96\" height=\"96\" xlink:href=\"data:image/png;base64,$(base64 -i assets/img/favicons/favicon-96x96.png)\"/>
</svg>" > assets/img/favicons/favicon.svg

# 清理源文件
rm favicon.png
```

### 备选方案：在线工具

如果不方便使用 ImageMagick，也可以用 [Real Favicon Generator](https://realfavicongenerator.net/)：上传源图 → 保持默认选项 → 下载图标包 → 替换 `assets/img/favicons/` 下的文件（删除工具生成的 `site.webmanifest`，本仓库已自带）。

## 3. 清单

替换后的 `assets/img/favicons/` 应包含以下文件：

| 文件 | 用途 |
|------|------|
| `favicon.ico` | 经典浏览器图标 |
| `favicon-96x96.png` | 现代浏览器 PNG 图标 |
| `favicon.svg` | SVG 矢量图标 |
| `apple-touch-icon.png` | iOS 主屏幕 (180×180) |
| `web-app-manifest-192x192.png` | PWA 小图标 |
| `web-app-manifest-512x512.png` | PWA 大图标 |
| `site.webmanifest` | PWA 配置文件 |

## 4. 验证

本地运行 `bundle exec jekyll serve`，浏览器访问后检查标签页图标是否更新。

> **提示**：由于浏览器缓存，可能需要强制刷新（Cmd+Shift+R）或使用隐私模式查看。
