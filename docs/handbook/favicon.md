# 更换网站图标（Favicon）

Chirpy 的默认图标存放在 `assets/img/favicons/`。以下是生成并替换为自己的图标的步骤。

## 1. 准备图片

准备一张正方形图片（PNG、JPG 或 SVG），尺寸至少 **512×512** 像素。建议使用你的头像或博客 Logo。

## 2. 生成图标

访问在线工具 [Real Favicon Generator](https://realfavicongenerator.net/)：

1. 点击 **Pick your favicon image** 按钮，上传你的图片。
2. 下一页会展示图标在各种场景中的预览。保持默认选项即可。
3. 滚动到页面底部，点击 **Next →** 按钮，下载生成的图标包。

## 3. 替换文件

下载并解压图标包，删除以下文件：

- `site.webmanifest`（Chirpy 已经自带）

然后将剩下的图片文件（`.PNG`、`.ICO`、`.SVG`）复制到博客仓库的 `assets/img/favicons/` 目录，覆盖原有文件。

| 文件 | 来自在线工具 | 来自 Chirpy 默认 |
|------|:------------:|:----------------:|
| `*.PNG` | ✓ 保留 | ✗ 删除 |
| `*.ICO` | ✓ 保留 | ✗ 删除 |
| `*.SVG` | ✓ 保留 | ✗ 删除 |

> ✓ 表示保留，✗ 表示删除。

## 4. 验证

本地运行 `bundle exec jekyll serve`，或直接部署后，打开浏览器检查浏览器标签页上的图标是否已更新。

> **提示：** 由于浏览器缓存，可能需要强制刷新（Ctrl+Shift+R / Cmd+Shift+R）或使用隐私模式查看。
