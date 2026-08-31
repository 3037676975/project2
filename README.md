# Project2 · AI 自动化开发部署工作流

> 把想法交给 AI，把部署交给系统。

Project2 是一个用于验证 **ChatGPT → GitHub → Webhook → 宝塔 → Linux** 自动化开发与部署链路的实验项目。

它不仅是一个静态展示页面，更重要的是用来验证一套可以重复复用的开发基础设施：

- ChatGPT 负责页面设计、代码生成与修改
- GitHub 负责源码保存与版本管理
- GitHub Webhook 负责在 Push 后通知服务器
- 宝塔面板负责自动执行 Git 更新
- Linux / Nginx 负责运行并对外提供网站

---

## 项目目标

传统的小型网站开发经常需要：

```text
修改代码
↓
手动上传服务器
↓
覆盖文件
↓
刷新页面
↓
发现问题后重新上传
```

Project2 希望把这个过程改造成：

```text
ChatGPT 修改代码
↓
提交 GitHub main
↓
GitHub Webhook 触发
↓
宝塔自动 Git Pull
↓
Linux 网站目录更新
↓
新版本自动上线
```

最终目标是让 **GitHub 成为唯一代码源**，服务器只负责运行最新代码。

---

## 当前页面

Project2 当前首页包含以下内容：

- 首页 Hero 主视觉
- 项目介绍
- 技术栈
- 自动化部署流程
- 部署指南
- 常见问题 FAQ
- GitHub 项目入口
- 移动端响应式布局

前端整体采用深色科技 / AI / Developer Tool 风格。

---

## 技术栈

| 模块 | 技术 / 服务 | 作用 |
| --- | --- | --- |
| AI 开发 | ChatGPT | 页面设计、代码生成、代码修改 |
| 代码托管 | GitHub | Git 仓库、Commit、版本管理 |
| 自动触发 | GitHub Webhook | Push 后通知宝塔 |
| 服务器管理 | 宝塔面板 | Git 管理、网站管理、部署日志 |
| 操作系统 | Linux | 运行网站与服务 |
| Web Server | Nginx | 对外提供 HTTP 服务 |
| 前端 | HTML + CSS + JavaScript | 当前 Project2 页面 |
| 版本控制 | Git | Clone、Pull、分支管理 |

---

## 项目结构

```text
project2/
├── index.html
├── README.md
├── coverage_inventory.json
└── green_source_prompt.md
```

说明：

- `index.html`：当前正式首页
- `README.md`：项目说明与部署文档
- `coverage_inventory.json`：Image-First 工作流中的页面覆盖清单
- `green_source_prompt.md`：Green Source 视觉资产生成提示

---

## 自动部署架构

```text
ChatGPT
   │
   │ 修改代码
   ▼
GitHub Repository
   │
   │ Commit / Push to main
   ▼
GitHub Webhook
   │
   │ HTTP 通知
   ▼
宝塔 Git 管理
   │
   │ git pull
   ▼
/www/wwwroot/project2
   │
   ▼
Nginx / Linux
   │
   ▼
网站上线
```

---

## 当前部署配置

```text
Repository:
https://github.com/3037676975/project2.git

Branch:
main

Server Path:
/www/wwwroot/project2

Web Port:
28438
```

> 如果迁移到新的服务器或域名，只需要替换服务器目录、网站域名/端口以及对应 Webhook 即可。

---

## 从零部署

### 1. 创建 GitHub 仓库

创建仓库后，必须至少存在一次 Commit。

如果仓库完全为空，宝塔可能无法正确获取 `main` 分支。

可以先创建：

```text
README.md
```

或：

```text
index.html
```

---

### 2. 宝塔创建 Git 网站

进入：

```text
宝塔
→ 网站
→ 添加站点
→ Git 部署
```

填写：

```text
仓库：
https://github.com/3037676975/project2.git

分支：
main

网站目录：
/www/wwwroot/project2
```

添加新项目时请为每个项目使用独立目录。

公开仓库优先使用 HTTPS Clone 地址，可以省去 SSH Deploy Key 的配置。

---

### 3. 配置 GitHub Webhook

在宝塔项目的 Git 管理页面复制 Webhook URL。

然后进入 GitHub：

```text
Repository
→ Settings
→ Webhooks
→ Add webhook
```

推荐设置：

```text
Payload URL:
宝塔生成的 Webhook URL

Content type:
application/json

Secret:
按实际配置填写；未启用时可留空

Events:
Just the push event

Active:
开启
```

之后每次 `main` 分支出现新的 Push，GitHub 就会通知宝塔执行部署。

---

## 重要：dubious ownership 报错

宝塔自动部署时可能出现：

```text
fatal: detected dubious ownership in repository at '/www/wwwroot/project2'
```

这是 Git 的目录安全检查。

在宝塔终端执行：

```bash
git config --system --add safe.directory /www/wwwroot/project2
```

然后检查：

```bash
git -C /www/wwwroot/project2 status
```

手动测试拉取：

```bash
git -C /www/wwwroot/project2 pull origin main
```

如果输出：

```text
Already up to date.
```

或正常出现 Fast-forward 更新，说明 Git 目录权限问题已经解决。

每创建一个新的宝塔 Git 项目，如果再次出现这个错误，都应该针对对应项目目录单独添加 `safe.directory`。

---

## 如何验证自动部署是否成功

最简单的测试方法：

1. 修改 `index.html`
2. Commit 到 `main`
3. 打开 GitHub Webhook Delivery 记录
4. 打开宝塔 Git 管理 → 部署记录
5. 确认服务器完成 `git pull`
6. 刷新网站

完整成功链路应该是：

```text
GitHub Push
✓
Webhook Delivery
✓
宝塔收到事件
✓
Git Pull 成功
✓
网站内容更新
```

---

## 常见问题

### 宝塔获取不到 main 分支

先确认仓库不是空仓库，并且已经存在第一次 Commit。

公开仓库建议使用：

```text
https://github.com/用户名/仓库.git
```

---

### Webhook 有记录，但网站没有更新

检查宝塔：

```text
网站
→ 项目设置
→ Git 管理
→ 部署记录
```

重点检查：

- Git 是否报错
- 当前分支是否为 `main`
- 是否存在 Git 冲突
- `safe.directory` 是否配置
- 网站目录是否正确

---

### 端口无法访问

依次检查：

- 宝塔安全设置
- Linux 防火墙
- 云服务器安全组
- Nginx 是否监听对应端口
- 网站状态是否正常

---

### 可以部署其他 GitHub 项目吗？

可以。当前这套链路可以作为通用部署底座，但不同技术栈在 `git pull` 后需要执行不同操作。

#### HTML / CSS / JavaScript

通常：

```bash
git pull origin main
```

即可。

#### PHP

通常 Git Pull 后即可运行，但还需要根据项目配置：

- PHP 版本
- Composer
- 数据库
- 伪静态
- `.env`

#### Node.js / Next.js

例如：

```bash
git pull origin main
npm install
npm run build
pm2 restart project-name
```

#### Python

一般需要：

```text
Git Pull
→ 安装 requirements
→ 配置虚拟环境
→ 重启 Gunicorn / Uvicorn / Supervisor 服务
```

#### Docker

例如：

```bash
git pull origin main
docker compose up -d --build
```

---

## 安全规范

以下内容不要直接提交到公开 GitHub：

```text
SSH 私钥
服务器密码
数据库密码
API Key
支付密钥
Cloudflare Token
Webhook Token
.env
```

推荐 `.gitignore`：

```gitignore
.env
.env.local
.env.production
node_modules/
vendor/
```

真正的生产密钥应该保存在服务器环境变量或服务器本地配置中。

---

## 开发原则

自动部署建立以后，尽量不要直接进入服务器网站目录手工修改源码。

推荐统一流程：

```text
修改 GitHub
↓
Commit
↓
Webhook
↓
自动部署
```

因为服务器上的临时手工修改，下一次 Git Pull 时可能被覆盖或产生冲突。

---

## 后续计划

Project2 可以继续扩展为：

- AI 产品 Demo
- AI Agent 展示站
- 自动化部署模板
- Node.js / Next.js 实验项目
- Docker 自动部署实验
- 私有仓库部署测试
- 域名 + HTTPS 正式环境
- 自动 Build / Restart 部署脚本
- 部署状态监控

---

## 项目仓库

```text
https://github.com/3037676975/project2
```

---

## 核心理念

> GitHub 负责保存代码，Webhook 负责触发，宝塔负责部署，Linux 负责运行，AI 负责把想法变成代码。

```text
ChatGPT → GitHub → Webhook → 宝塔 → Linux
```

**Project2 · Automation Lab**
