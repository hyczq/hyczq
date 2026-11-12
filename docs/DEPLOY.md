# GitHub 首页贪吃蛇 — 部署说明

目标：让 `github.com/hyczq` 的个人主页底部出现一条吃掉贡献图的贪吃蛇，并且每天自动更新。

## 生效的三个条件

| 条件 | 说明 |
|---|---|
| 仓库名必须是 `hyczq/hyczq` | 只有「仓库名 == 用户名」的仓库，README 才会显示在个人主页 |
| 仓库必须是 public | 私有仓库的图片无法被 README 引用 |
| 仓库里要有 README.md | 就是本目录的 `README.md`，在 `main` 分支根目录 |

## 会生成什么

`.github/workflows/snake.yml` 每 12 小时跑一次（UTC 00:17 / 12:17）：

1. `Platane/snk` 抓取 `hyczq` 的贡献日历，渲染成 SVG；
2. `crazy-max/ghaction-github-pages` 把 SVG 推送到 `output` 分支根目录；
3. README 里的 `raw.githubusercontent.com/hyczq/hyczq/output/...` 指向它，浏览器按深浅色自动切换。

产出两个文件：

- `output/github-snake.svg` — 浅色主题
- `output/github-snake-dark.svg` — 深色主题（`palette=github-dark`）

## 部署步骤

```bash
# 1. 在 GitHub 上新建空仓库 hyczq/hyczq（public，不要勾选 README/.gitignore/License）

# 2. 推送本目录
cd projects/github-profile-snake
git init -b main
git add -A
git commit -m "feat: contribution snake animation"
git remote add origin git@github.com:hyczq/hyczq.git
git push -u origin main
```

3. 打开仓库 `Settings → Actions → General → Workflow permissions`，选 **Read and write permissions** 并保存
   （工作流里已声明 `permissions: contents: write`，但仓库级别的默认设置不能是只读）。
4. 打开 `Actions → 🐍 Generate Contribution Snake → Run workflow` 手动跑一次。
5. 跑完后仓库会出现 `output` 分支，里面有 `github-snake.svg` 和 `github-snake-dark.svg`。
6. 访问 `https://github.com/hyczq` 看效果（README 有缓存，等 1–2 分钟或强制刷新）。

## 常见问题

| 现象 | 原因 / 处理 |
|---|---|
| 图片 404 | 工作流还没跑完，或 `output` 分支没生成；去 Actions 看日志 |
| Actions 页面没有这个工作流 | 文件不在默认分支 `main` 上，或仓库没启用 Actions |
| 报 `Resource not accessible by integration` | 第 3 步的 Workflow permissions 还是只读，改成 Read and write |
| 跑成功了但图片没变 | 浏览器/Camo 缓存，等几分钟；或换一个 `?v=2` 查询串强制刷新 |
| 60 天后定时任务停了 | GitHub 对长期无提交的仓库会暂停 schedule；点一次 Run workflow 即可恢复（正常情况下提交本身会续命） |
| 主页不显示 README | 仓库名不是 `hyczq/hyczq`，或分支不是默认分支 |
| 想改配色 | 在 `snake.yml` 的 outputs 里加参数，如 `?palette=github-dark&color_snake=#00ff88` |
| 想改成每小时 | 把 cron 改成 `17 * * * *`（注意 Actions 最小间隔 5 分钟，且免费额度有限） |
