---
title: '从零开始搭建个人博客完全指南'
description: '一个适合零基础的建站全流程教程，使用 Astro Koi 模板和 Vercel 搭建一个高性能、完全免费的个人博客网站'
pubDate: 'Oct 17 2025'
heroImage: '/blog-placeholder-3.jpg'
---

## 前言

新手也可以学会。本教程假设你有基本的 Git/GitHub 知识，不了解前端技术栈也没关系——跟着步骤走就能成功！

**你将学会**：
- ✅ 选择并配置博客模板
- ✅ 在本地运行和预览网站
- ✅ 使用 Vercel 免费部署到云端
- ✅ 解决常见的部署问题

**成果展示**：一个像 [我的博客](https://pcblog-ashy.vercel.app) 这样的个人网站！

---

## Part 1: 准备工作

### 1.1 安装 Node.js

现代前端项目都需要 Node.js 环境。

**Windows 用户**：
1. 访问 [Node.js 官网](https://nodejs.org/)
2. 下载 LTS 版本（长期支持版）
3. 双击安装包，一路"下一步"即可

**验证安装**：
打开命令提示符（PowerShell 或 CMD），输入：
```bash
node -v
npm -v
```
能看到版本号说明安装成功！

**安装 pnpm**（推荐的包管理器）：
```bash
npm install -g pnpm
```

### 1.2 选择博客模板

我选择的是 [Koi 模板](https://github.com/tcdw/koi)，原因：
- 🎨 简洁美观，支持亮/暗主题
- ⚡ 基于 Astro，性能极佳
- 📝 Markdown 写作，上手容易
- 🛠️ 配置简单，开箱即用

**Fork 模板到你的 GitHub**：
1. 访问 [Koi 仓库](https://github.com/tcdw/koi)
2. 点击右上角 **Fork** 按钮
3. Fork 到你自己的账号下

---

## Part 2: 本地搭建

### 2.1 克隆项目到本地

在你想要存放项目的目录下，打开终端：
```bash
# 替换成你自己的 GitHub 用户名
git clone https://github.com/你的用户名/koi.git
cd koi
```

### 2.2 安装依赖
```bash
pnpm install
```

这个过程会下载项目所需的所有依赖包，可能需要几分钟。

### 2.3 启动开发服务器
```bash
pnpm dev
```

看到类似这样的提示：
```
🚀 astro v4.x.x started in XXXms
┃ Local    http://localhost:4321/
┃ Network  use --host to expose
```

在浏览器打开 `http://localhost:4321`，你应该能看到网站了！🎉

**神奇的热更新**：
修改任何文件（比如博客内容），保存后浏览器会**自动刷新**展示变化，非常方便！

---

## Part 3: 个性化配置

### 3.1 修改站点信息

编辑 `src/consts.ts`：
```typescript
export const SITE_TITLE = '你的博客名称';
export const SITE_DESCRIPTION = '你的博客简介';
export const SITE_AUTHOR_NAME = '你的昵称';
// ... 其他配置
```

### 3.2 修改 Astro 配置（重要！）

打开 `astro.config.js`，修改这两个配置：
```javascript
export default defineConfig({
  // 改为你的域名（Vercel 会提供）
  site: 'https://你的项目名.vercel.app',
  
  // ⚠️ 关键：删除或注释掉这一行
  // base: process.env.NODE_ENV === "production" ? "/koi/" : "",
  // 因为 Vercel 部署在根路径，不需要 base
  
  // 其他配置保持不变...
});
```

**为什么要改？**
- 原配置 `base: "/koi/"` 是为 GitHub Pages 子路径部署设计的
- Vercel 部署在根路径，如果不改会导致所有链接 404 错误！

### 3.3 写第一篇博客

在 `src/content/blog` 目录下创建 `hello-world.md`：
```markdown
---
title: '我的第一篇博客'
description: '记录建站过程'
pubDate: '2025-01-15'
---

## 欢迎！

这是我的第一篇博客文章，今天成功搭建了自己的网站！

### 为什么写博客？

- 记录学习过程
- 分享技术心得
- 建立个人ip
```

保存后，本地开发服务器会自动刷新，立即能看到新文章！

---

## Part 4: 部署到 Vercel

### 4.1 推送代码到 GitHub
```bash
git add .
git commit -m "feat: 初始化个人博客"
git push
```

### 4.2 导入项目到 Vercel

1. 访问 [Vercel 官网](https://vercel.com)
2. 使用 GitHub 账号登录
3. 点击 **Add New... → Project**
4. 选择你的 `koi` 仓库，点击 **Import**

### 4.3 配置构建设置

**关键步骤**：确认以下设置（Vercel 通常会自动识别）

- **Framework Preset**: `Astro`
- **Build Command**: `pnpm build` 或 `npm run build`
- **Output Directory**: `dist`

**如果自动识别失败，手动设置这些参数！**

### 4.4 部署！

点击 **Deploy** 按钮，等待 1-3 分钟，看到庆祝动画后，你的网站就上线了！🎊

Vercel 会提供一个免费域名：`https://你的项目名.vercel.app`

---

## Part 5: 常见问题与解决方案

### ❌ 问题 1: 点击链接出现 404 错误

**症状**：首页能访问，但点击博客链接显示 `404: NOT_FOUND`

**原因**：`astro.config.js` 中的 `base` 配置没有修改

**解决方案**：
```javascript
// astro.config.js
export default defineConfig({
  site: 'https://你的域名.vercel.app',
  // 删除或注释掉这一行
  // base: '/koi/',
});
```

修改后重新 push，Vercel 会自动重新部署。

### ❌ 问题 2: Vercel 构建失败

**检查项**：
1. 查看 Vercel 的 Deployment Logs（部署日志）
2. 确认 `Framework Preset` 是否选择了 `Astro`
3. 本地是否能正常运行 `pnpm build`

**解决方案**：
在项目根目录创建 `vercel.json`：
```json
{
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "framework": "astro"
}
```

### ❌ 问题 3: 网站有时候访问不了

**可能原因**：
- Vercel 正在重新部署（1-3 分钟）
- 网络连接问题
- DNS 缓存问题

**解决方案**：
1. 检查 Vercel Dashboard 的部署状态
2. 等待几分钟后重试
3. 清除浏览器缓存（`Ctrl + F5`）

---

## Part 6: 后续优化

### 绑定自定义域名（可选）

如果你有自己的域名，可以在 Vercel 项目设置中绑定，让网站更专业！

### 自动化部署

现在已经实现了自动化：
- 本地修改代码 → `git push`
- Vercel 自动检测 → 重新构建部署
- 几分钟后网站更新完成

---

## 总结

恭喜你完成了从零到一的建站过程！整个流程其实很简单：

1. 📦 选择模板并 Fork
2. 💻 本地配置和预览
3. ⚙️ 修改关键配置（`astro.config.js`）
4. 🚀 推送到 GitHub，Vercel 自动部署

**核心经验**：
- ✅ 记得修改 `base` 配置，避免 404 错误
- ✅ 使用 Vercel 比 GitHub Pages 更简单
- ✅ 热更新让本地开发体验极佳
- ✅ 自动化部署节省大量时间

现在，开始享受写作吧！每次 `git push`，你的内容就会自动发布到全世界。📝✨

---

## 模板

恭喜你完成了从零到一的建站过程！整个流程其实很简单：

1. [Novel Tex](https://github.com/soryu-ryouji/novel-tex-theme)
2. [Fuwari](https://github.com/saicaca/fuwari)
3. [樱花庄的白猫博客主题](https://github.com/mashirozx/sakura?tab=readme-ov-file)
4. [astro-theme-pure](https://github.com/cworld1/astro-theme-pure)

---

## 参考资源

- [Astro 官方文档](https://docs.astro.build/)
- [Koi 模板仓库](https://github.com/tcdw/koi)
- [Vercel 文档](https://vercel.com/docs)
- [原教程参考](https://axi404.top/blog/website-vercel)

**如果这篇教程帮到了你，欢迎留言交流！**