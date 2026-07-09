# GitHub Pages 部署问题排查与解决方案

## 项目概述

本项目是一个**纯静态个人网站**，包含：
- `index.html` - 主页面（个人介绍网站）
- `styles.css` - 样式文件
- `script.js` - 脚本文件
- `public/` - 旧的 Hexo 构建产物（已不再使用）

## 问题现象与原因分析

### 问题一：GitHub Actions 默认构建失败（2026-07-09）

**错误信息：**
```
Internal server error. Correlation ID: b92e780c-bdc1-484e-b44f-f7d78bac5794
The job was not acquired by Runner of type hosted even after multiple attempts
```

**原因分析：**
这是 GitHub 官方 Pages 构建服务的**临时服务器故障**。当配置 Pages 从 `master` 分支的 `/` 目录构建时，GitHub 会自动触发 `pages-build-deployment` 工作流。

- ✅ Build 阶段成功（8秒）
- ✅ Report-build-status 成功（3秒）
- ❌ Deploy 阶段超时（15分钟）

**根因：** GitHub 服务器端的 Runner 资源不足或内部错误，导致部署任务无法被分配执行器。

**解决方案：** 等待一段时间后自动恢复，或手动重新触发构建。

### 问题二：自定义 GitHub Action 权限拒绝（2026-07-09）

**错误信息：**
```
remote: Permission to HuiFeiYa/HuiFei.github.io.git denied to github-actions[bot].
The requested URL returned error: 403
```

**原因分析：**
`username.github.io` 类型的仓库是特殊的**用户主页仓库**，默认从 `master` 分支直接部署。

当尝试使用自定义 Action（如 `peaceiris/actions-gh-pages@v4`）将构建产物推送到 `gh-pages` 分支时：
1. 仓库没有 `gh-pages` 分支
2. `github-actions[bot]` 没有权限创建新分支或推送到受保护的分支
3. 用户主页仓库的 Pages 配置默认指向 `master` 分支

**解决方案：** 删除自定义 Action，使用直接推送方式。

## 推荐部署方式

### 方式一：直接推送（推荐）

**原理：** 这是一个纯静态网站，直接将根目录文件推送到 GitHub `master` 分支即可。

**步骤：**
```bash
git add .
git commit -m "Update website"
git push origin master
```

### 方式二：使用 Hexo 部署工具（已验证可用）

虽然这不是 Hexo 项目，但项目中安装了 `hexo-deployer-git` 插件，可以利用它来部署。

**部署命令：**
```bash
npm run deploy
```

**执行结果（2026-07-09 成功）：**
```
To https://github.com/HuiFeiYa/HuiFei.github.io.git
 + a973682...69f05d5 HEAD -> master (forced update)
INFO  Deploy done: git
```

**注意：** 此方式会将 `public/` 目录的内容推送到 GitHub，而非根目录文件。

## 推荐的标准部署流程

### 方法：直接 git 推送（最简单可靠）

每次修改后运行：
```bash
git add .
git commit -m "更新内容描述"
git push origin master
```

### GitHub Pages 配置

在 GitHub 仓库的 `Settings > Pages` 中：
- **Source**：选择 `Deploy from a branch`
- **Branch**：选择 `master`，目录选择 `/ (root)`

## 常见问题

### Q1: 部署后网站没有更新？
**原因：** GitHub Pages 有缓存机制

**解决：**
- 等待几分钟，缓存会自动刷新
- 手动触发重新部署：修改任意文件并重新推送

### Q2: 部署报错 "Permission denied"？
**原因：** Git 凭证问题

**解决：**
- 检查 Git 配置的用户名和邮箱
- 在 Windows 上可以使用 Git Credential Manager 管理凭证

### Q3: 网站样式丢失？
**原因：** 文件路径问题或缓存

**解决：**
- 检查 `index.html` 中引用的 `styles.css` 和图片路径
- 强制刷新浏览器缓存（Ctrl + Shift + R）

## 项目文件说明

| 文件 | 作用 |
|------|------|
| `index.html` | 主页面（个人介绍网站） |
| `styles.css` | 样式文件 |
| `script.js` | 脚本文件 |
| `public/` | 旧的 Hexo 构建产物（已不再使用） |
| `package.json` | Node.js 项目配置（包含部署脚本） |

## 总结

对于 `username.github.io` 类型的用户主页仓库，**最简单可靠的部署方式是直接使用 `git push`**，不需要配置 GitHub Action。这种方式避免了 GitHub 服务器端的 Runner 资源问题，部署速度快且稳定。

如果之前部署成功过，那是因为：
1. GitHub 服务器状态正常
2. 配置正确指向了 `master` 分支的根目录