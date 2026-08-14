# bupt-major-switching-programming-questions

北京邮电大学 2025 年年初转专业机试部分编程题的题目、解答和解析。是转入计算机学院的机试题。

在线阅读： [链接](http://LS-Hower.github.io/bupt-major-switching-programming-questions/)

## 构建方式

使用 MkDocs 构建，在 GitHub Pages 中部署。

源码推送到 `main` 分支后 **不会** 自动部署。需要手动触发 GitHub Actions（ `.github/workflows/deploy.yml` ）才会重新构建并发布网页。

两种触发方式：

1. 命令行： `gh workflow run deploy.yml` 。需安装 [GitHub CLI](https://cli.github.com/) 并登录。
2. 进入仓库的 Actions 页面，选中“Deploy MkDocs”，点击“Run workflow”。
