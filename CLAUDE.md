# BridgeLi 博客项目规范

Hugo 博客，主题 PaperMod，部署到 GitHub Pages。

## 内容目录约定

### 新文章（Page Bundle）
每篇文章一个目录，放在 `content/posts/` 下，带日期前缀方便文件管理器中识别：

```
content/posts/2026-05-01-slug-name/       # 日期前缀 + 英文 slug
  index.md                                 # 文章内容，frontmatter 含 date 字段
  some-image.png                           # 配图，直接同目录引用
```

markdown 中引用图片：`![desc](some-image.png)`，不用写路径前缀，Hugo Page Bundle 自动处理。

日期前缀和 frontmatter 的 `date` 字段保持一致，方便在文件管理器中一眼看出文章时间。

### 旧文章
旧文章是单文件模式（`日期-标题.md`），不动，Hugo 兼容两种模式共存。

### 命名规则
- 目录名：日期前缀 + 英文 slug，格式 `YYYY-MM-DD-slug-name`
- 图片：英文命名，小写，单词间用 `-` 连接
- frontmatter 中 `date` 字段提供完整日期时间

## 图片规范
- 图片只放对应文章的 Page Bundle 目录内，不放 `static/`
- `static/` 只放站点级资源（favicon 等）

## Hugo 配置要点
- 构建命令：`hugo`
- 本地预览：`hugo server`
- 发布：推送到 main 分支，GitHub Actions 自动构建部署
- baseURL: `https://bridgeli.cn/`，本地预览用 `hugo server` 自动覆盖为 localhost

## 不做的事
- 不迁移旧文章到 Page Bundle
- 不用年/月/日嵌套目录（会导致 URL 层级过深）
- 不把图片放 `static/` 目录