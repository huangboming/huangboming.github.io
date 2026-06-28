# 设置社交预览图（Social Preview Image）

社交预览图是分享博客链接到 Twitter、Telegram 等平台时展示的卡片图（`og:image`）。设置后能提升分享时的点击率和专业感。

## 设计方向

社交预览图的核心设计约束：

- 尺寸 **1200×630**（比例 1.91:1），横向横幅
- 一眼看出博客主题
- 留有标题文字位置（如果图内不放标题，Chirpy 会在卡片上叠加标题）
- 与博客主题（「数字漫游」— 漫无目的地探索和折腾）契合

## 1. 生成源图（AI 生图）

使用 AI 生图工具生成一张横幅图。AI 生图工具通常不保证精确尺寸，所以生成接近比例的大图即可（如 16:9），后续用 ImageMagick 精确裁剪。

以下是本博客使用的 prompt 模板，设计概念为「0/1 码流组成的蜿蜒小径延伸至远方」：

```
A cinematic wide banner, 1200x630, dark digital landscape. A winding glowing path made of scattered binary digits (`0` and `1`) and tiny luminous particles snakes from the bottom center towards a bright horizon point in the upper third. The path resembles both a data stream and a lone trail through a vast abstract digital wilderness. Deep navy (#0f172a) background with subtle circuit-board grid lines fading into the distance. A soft warm gradient glow at the horizon (cyan to amber), suggesting dawn and discovery. Above the horizon, large empty negative space for text overlay. Matte painting meets cyber-minimalism — atmospheric, clean, not cluttered. No human figures, no text, no UI elements. 16:9 cinematic aspect ratio, photorealistic yet abstract.
```

## 2. 精确裁剪（ImageMagick）

源图保存为 `social_preview_image.png`，裁剪到精确的 1200×630：

```shell
magick social_preview_image.png -resize "1200x630^" -gravity center -extent 1200x630 assets/images/social-preview.png
```

- `-resize "1200x630^"` — 等比缩放到填满 1200×630 的最小尺寸（超出部分居中裁剪）
- `-gravity center` — 居中定位
- `-extent 1200x630` — 裁剪到精确尺寸

## 3. 配置

在 `_config.yml` 中设置路径：

```yaml
# The URL of the site-wide social preview image used in SEO `og:image` meta tag.
social_preview_image: assets/images/social-preview.png
```

> **说明**：文章可以在 Front Matter 中单独设置 `image:` 来覆盖全站默认图。

## 4. 验证

部署后使用以下工具验证效果：

- [opengraph.xyz](https://opengraph.xyz) — 预览各种平台的卡片效果
- [Twitter Card Validator](https://cards-dev.twitter.com/validator) — 验证 Twitter 卡片
