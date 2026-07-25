# AI 学习手记

> 把每一次理解，写在边页。

这是一个由 **Astro 7** 构建并部署在 GitHub Pages 上的静态博客。它不是作品展示页，而是一份持续更新的学习记录：从 AI 开发、智能体与 Web 安全的探索，到 Java 周测后的复盘、代码练习和真实项目中的踩坑与验证。

在线访问：[allen-david.github.io/ai-learning-blog](https://allen-david.github.io/ai-learning-blog/)

## 这个博客记录什么

- **AI 开发与智能体实践**：把工具、想法和实现过程沉淀为可回看的笔记；
- **基础学习复盘**：记录 Java、集合、Stream、线程等知识点的理解与纠错；
- **动手过程**：不把“看懂文档”当作结束，而是通过亲手敲代码、编译、运行和记录，把问题落实下来；
- **个人与项目经历**：保留关于学习方向、信息安全兴趣与项目实践的阶段性自述。

目前已有“从零开始学习 AI 开发”“第一次周考复盘”“关于我”等文章；后续内容直接以 Markdown 的形式加入即可。

## 技术方案

- **Astro 7**：生成静态页面，默认不向访客发送多余的前端框架运行时代码；
- **Astro Content Collections**：使用 Zod 为每篇文章约束 `title`、`description`、`pubDate` 与 `tags`；
- **Markdown 写作**：文章存放在 `src/content/blog/`，文件名会成为文章链接的一部分；
- **原生 CSS 与少量浏览器脚本**：实现纸张质感、阅读进度、渐入效果和减少动态效果偏好；
- **GitHub Actions + GitHub Pages**：每次推送到 `main`，都会自动构建并部署 `dist/`。

## 页面与内容结构

```text
.
├─ src/
│  ├─ content/blog/              # Markdown 文章
│  ├─ content.config.ts          # 文章元数据 schema
│  ├─ layouts/BaseLayout.astro   # 全站布局、导航与阅读交互
│  └─ pages/
│     ├─ index.astro             # 首页：最新文章列表
│     └─ blog/[...slug].astro    # 文章详情页
├─ .github/workflows/
│  └─ deploy-pages.yml           # GitHub Pages 自动部署工作流
├─ astro.config.mjs              # 站点域名与仓库子路径配置
└─ public/                       # 可直接引用的静态资源
```

## 写一篇新文章

在 `src/content/blog/` 下创建一个 `.md` 文件。例如：

```md
---
title: "文章标题"
description: "用一两句话说明这篇文章解决了什么问题。"
pubDate: 2026-07-25
tags:
  - AI 开发
  - 学习记录
---

从一个真实问题开始写：看到了什么、亲手做了什么、结果如何、下一步准备怎么验证。
```

保存后，首页会自动按日期排序显示文章；文章地址为 `/blog/文件名/`。构建时如果缺少必填字段或日期格式不正确，Astro 会报错提示。

## 本地开发（WSL）

项目位于 WSL 的 Linux 文件系统中：

```bash
cd ~/study/blog
npm install
npm run dev
```

终端会显示本地访问地址，通常是 `http://localhost:4321`。如果该端口已被其他项目占用，Astro 会自动使用下一个可用端口，以终端输出为准。

构建并预览生产版本：

```bash
npm run build
npm run preview
```

`npm run build` 会把静态站点生成到 `dist/`；提交前建议至少运行一次，确认文章、类型检查和页面生成都正常。

## GitHub Pages 部署

部署由 `.github/workflows/deploy-pages.yml` 负责。工作流会在每次推送 `main` 后执行：

1. 使用 Node.js 22 安装依赖；
2. 运行 `npm run build`；
3. 上传 `dist/`；
4. 发布到 GitHub Pages。

首次使用时，请在仓库 **Settings → Pages** 中将 Source 设为 **GitHub Actions**。之后将经过验证的改动提交并推送到 `main` 即可：

```bash
git add <需要提交的文件>
git commit -m "Describe the change"
git push origin main
```

仓库配置了 `base: "/ai-learning-blog"`，因此站内链接和静态资源会自动适配 GitHub Pages 的仓库子路径；不要在文章或页面中随意去掉这一前缀。

## Windows 与 WSL 协作

日常开发、安装依赖与运行命令建议在 WSL 的 `~/study/blog` 内完成。若需要通过 Windows 下载 GitHub 模板或资料，可以先放入 `F:\situ\bridge`，再从 WSL 的 `/mnt/f/situ/bridge` 复制到项目需要的位置。

```text
Windows / GitHub → F:\situ\bridge → /mnt/f/situ/bridge → ~/study/blog
```

这样既能利用 Windows 的网络环境，也能保持 Node 依赖与文件监听在 WSL 中稳定运行。

## 写作原则

这里希望留下的不只是结论，而是可追溯的学习过程：

> 看到问题 → 拆开问题 → 亲手验证 → 记录结果 → 再次复盘

AI 可以帮助整理思路、生成初稿和排查错误；最终的理解仍要回到代码、命令、测试和实际运行结果中确认。
