---
title: "关于我：让好奇心与 AI 一起工作"
description: "信息安全专业学生，主线是 AI 学习与智能体实践；安全是课程需要，也是持续延展的兴趣。"
pubDate: 2026-07-24
tags:
  - 关于我
  - 信息安全
  - AI 学习
  - 智能体
---

> **好奇地拆解系统，认真地验证结论；让 AI 成为学习与构建的放大器。**

## 🎥 两分钟认识我

<video controls preload="metadata" poster="/ai-learning-blog/images/about/avatar-intro-wave.png" width="100%" style="display: block; border-radius: 14px; box-shadow: 0 18px 45px rgba(20, 28, 46, 0.2); background: #111827;">
  <source src="/ai-learning-blog/images/about/about-avatar-intro.mp4" type="video/mp4" />
  你的浏览器暂不支持视频播放，可直接下载 <a href="/ai-learning-blog/images/about/about-avatar-intro.mp4">这段数字人自我介绍</a>。
</video>

*由极客风虚拟形象出镜：有一点手势、有一点表情，剩下的是我对 AI 的好奇心。*

## 👋 关于我

你好，我是 **徐恺晨**，一名信息安全专业学生，也是一名仍在持续升级的技术探索者。

我不是一个很聪明的人，但我相信勤能补拙。学习上，我更喜欢后程发力：先把基础和问题耐心啃下来，再在持续练习里把理解一点点拉高。

安全最初是我的专业课需要，而不是一条预先规划好的职业标签。学着学着，我开始对系统的边界、输入如何改变行为、一个小小配置如何放大风险产生了好奇。于是，课程之外也会继续了解 Web 安全的基本问题，并把所学放进**授权靶场、课程实验和自建演示项目**中验证。

现在对我而言，主线更明确：**学习 AI，理解智能体，并用它提高解决问题的效率。** 安全是这条主线的兴趣延展。无论是读代码、整理抓包线索，还是把项目从“能跑”推进到“能展示”，我都在体会 AI 的价值：它能提出思路、组织信息、加快试错；而我仍要负责理解、验证与取舍。

```text
身份标签：Information Security Student / AI Learner / Curious Builder
当前关键词：AI · Agent · Vibe Coding · Web Security · Project Delivery
```

## 🛠️ 我在学什么、折腾什么

### Web 安全：从课程知识到可验证的实验

我目前处在打基础、重理解的阶段。学习与练习主要围绕常见 Web 风险，以及“为什么会发生、如何验证、怎么修复”这条链路：

- **输入与查询安全**：SQL 注入、XSS、CSRF；
- **访问控制与文件边界**：越权、文件上传、路径处理与基础认证/会话问题；
- **工具与观察**：在课程或授权实验中使用 Burp Suite 进行弱口令验证，用 Wireshark 阅读抓包，也尝试让 AI 帮我归纳流量与排查线索；
- **最重要的习惯**：不把“能跑通”当成结论，保留证据、控制影响范围，并回到修复建议与复盘。

### AI 与智能体：把“会用”变成“会判断”

我接触 AI 的起点很实际：最开始只是想更高效地完成课程项目与日常开发；真正接触到智能体（Agent）后，却被它的能力彻底打动，顺手掉进了氛围式编程（Vibe Coding）的快乐里。Claude Code、Codex 等工具轮番上阵，让我直观感受到：在需求拆分、代码生成、查错、文档整理和界面打磨上，AI 已经是很有力量的协作者。

有一段很真实的“血泪史”：6 月 25 日，绑定 Google 的 Claude 账号被封，我一度气到宣布“再也不买 Pro”。不过吐槽归吐槽，工具始终只是工具；ChatGPT 的两次 Plus 使用体验一直很稳，也让我更愿意把注意力放回问题本身：如何把 AI 用好、用准、用得可验证。

- 我关心的不是“让 AI 替我完成一切”，而是怎样提出更好的问题、把结果接回真实工作流；
- 我会用智能体辅助读代码、梳理排错路径、撰写文档和快速搭建原型；
- 目前正在系统学习 AI 应用开发、智能体编排与大模型 API 的工程化落地；
- 我仍会保留人工验证：**AI 可以提出假设，结论要由代码、日志、测试和实际运行结果确认。**

## 🚀 折腾与实践

### AI 代码审计平台：从规则结果到可追问的解释

这是一次围绕代码安全检测的完整实践：上传演示代码、创建审计会话、输出风险等级、定位问题行、给出修复建议，并支持基于已上传代码的继续追问与 PDF 报告生成。平台接入了 DeepSeek API；大模型负责解释与问答，最终结论仍需要结合代码和运行结果判断。

<figure class="project-shot">
  <img src="/ai-learning-blog/images/about/scap-audit-findings.png" alt="AI 代码审计平台展示 SQL 注入、反射型 XSS、路径遍历等审计发现" loading="lazy" />
  <figcaption>代码审计结果：漏洞等级、代码定位、影响说明与修复建议会在同一视图中呈现。</figcaption>
</figure>

<figure class="project-shot">
  <img src="/ai-learning-blog/images/about/scap-ai-analysis.png" alt="AI 代码审计平台中智能体对漏洞进行结构化解释" loading="lazy" />
  <figcaption>智能体问答：将代码上下文交给模型，换取可读的风险解释与改进方向。</figcaption>
</figure>

这个项目让我看到 AI 更适合放在“协作环节”：先帮助整理问题、生成可读的解释，再由人完成验证、修复与交付。

### VeilLink 竞赛任务：为 API 调用设计前端入口

在学校比赛的小组项目 VeilLink 中，我负责的任务是 **API 调用前端**。我根据项目方案，完成了面向管理侧的 API Key 管理界面和相关调用入口：把原本偏底层的服务能力，转化为可以查看、创建和管理的前端交互。

<figure class="project-shot">
  <img src="/ai-learning-blog/images/about/veillink-api-console.png" alt="VeilLink 管理控制台中的 API Key 管理页面" loading="lazy" />
  <figcaption>VeilLink 控制台：在小组分工中，我聚焦 API 调用前端与管理操作的可用性。</figcaption>
</figure>

这段协作也提醒我：项目交付不仅是把功能堆出来，更要能把方案读懂、把自己的模块接好，并让演示者或用户一眼知道如何使用。

### AI 辅助靶机学习：让线索更快收敛

在**授权靶机与学习环境**中，我也尝试让 AI 协助整理信息收集、权限边界分析与提权思路。它最有价值的地方，不是直接给出“答案”，而是帮助我把零散现象写成待验证的假设：

```text
现象 → 假设 → 验证命令 / 证据 → 调整思路 → 复盘
```

这段经历让我更重视基本原则：明确授权边界、保留验证证据、理解每一个动作背后的系统逻辑，而不是机械地复制命令。

## 🏆 比赛经历

- **个人赛事**：参加过蓝桥杯和全国大学生数学竞赛，均获得省级奖项。它们让我持续练习在有限时间内拆解问题、保持耐心，并把解题过程写得更清楚。
- **团队赛事**：参与大学生服务外包创新创业大赛，团队获得国家级奖项。对我来说，这段经历的收获不只是一份结果，更是理解分工协作、方案表达和项目落地的一次实践。

## 🏃 运动，也是一种长期主义

写代码和训练看起来相距很远，但它们都需要把注意力放在当下的一步。参加学校运动会和越野赛时，我更能体会到：节奏、耐心和持续投入，往往比一开始冲得多快更重要。

<div class="sports-gallery">
  <figure class="wide">
    <img src="/ai-learning-blog/images/about/sports/sports-group-college.jpg" alt="计算机科学与工程学院同学参加院级越野赛后的合影" loading="lazy" />
    <figcaption>院级越野赛结束后，与计算机科学与工程学院同学的合影。</figcaption>
  </figure>
  <figure>
    <img src="/ai-learning-blog/images/about/sports/cross-country-running.jpg" alt="校级运动会赛道上正在跑步的徐恺晨" loading="lazy" />
    <figcaption>校级运动会途中：先找到自己的呼吸，再找到自己的节奏。</figcaption>
  </figure>
  <figure>
    <img src="/ai-learning-blog/images/about/sports/cross-country-finish.jpg" alt="校级越野赛接近终点时的徐恺晨" loading="lazy" />
    <figcaption>校级越野赛接近终点时，仍然把每一步跑稳。</figcaption>
  </figure>
  <figure class="wide">
    <img src="/ai-learning-blog/images/about/sports/sports-group-team.jpg" alt="同学参加院级越野赛后的合影" loading="lazy" />
    <figcaption>院级越野赛之后的集体留影。一起出发、一起完成，也是很珍贵的经历。</figcaption>
  </figure>
</div>

这种经历也会反过来影响我的学习方式：不把某一次卡住当成停下的理由，先做一点、记录一点，再从反馈里调整下一步。

## 🌍 好奇心也在技术之外

我喜欢探索还不了解的东西。除了课程和项目，我也会浏览不同地区的产品、技术社区与公共讨论：Instagram、X、WhatsApp、Telegram、Facebook、Discord 等平台对我更像观察窗口——标准的“互联网深海潜水员”，只阅读内容、只做观察、不抢麦发言。

日常还会“养” Hermes，让它在一些小实验里多燃一点 token；看着 token 哗哗流走，多少有点痛，但也确实快乐。也会不断试新工具、看新工作流。对我而言，这些并不等于追逐热点，而是希望在快速变化的 AI 时代里，保留观察、提问和亲手验证的习惯。

## 📬 结语

这个博客是我的技术实验记录本，也是给未来自己的可检索备忘。

我会在这里持续沉淀：AI 学习笔记、智能体辅助开发的真实经验、信息安全课程中的理解、项目踩坑与修复过程，以及那些尚未成熟但值得继续追问的想法。希望每篇文章都不只是结论，而是一次可以被复现、被讨论的探索。

如果你也在学习 AI、探索智能体、关注 Web 安全，或只是想交流一个有趣的技术问题，欢迎通过博客评论区或 [GitHub](https://github.com/Allen-David) 与我联系。

> Keep curious, keep verifying, keep building. 🤖
