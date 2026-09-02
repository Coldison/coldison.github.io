# Coldison Blog

这是 `https://coldison.github.io` 的 Hexo 源码项目，使用 Fluid 主题。

## 环境

- Node.js 20
- npm
- Git

首次使用或依赖发生变化时安装依赖：

```powershell
npm ci
```

`node_modules` 不需要备份或提交到 Git。若 OneDrive 中运行 Hexo 很慢，可将源码复制到普通本地目录后执行命令；文章和配置仍以本项目为准。

## 写作与预览

文章位于 `source/_posts`，草稿位于 `source/_drafts`。

```powershell
npm run server
```

浏览器访问 `http://localhost:4000`。停止服务时按 `Ctrl+C`。

## 发布前检查

```powershell
npm run check
```

该命令会清除旧缓存并重新生成完整站点。只有检查成功后才发布：

```powershell
npm run deploy
```

部署目标由 `_config.yml` 配置，目前是 `Coldison/coldison.github.io` 仓库的 `master` 分支。部署工具会强制更新该分支，因此不要在远端 `master` 分支直接编辑生成后的网页。

## 安全注意事项

- 不要把令牌、密码或 `.env` 文件提交到 Git。
- 项目中的旧 `gittoken.txt` 已被忽略；应在 GitHub 中撤销旧令牌，发布时使用 Git Credential Manager 登录。
- `public`、`db.json` 和 `.deploy_git` 都是可重新生成的文件，不应提交到源码仓库。
