---
title: 如何用Fuwari+Cloudflare搭建出一个属于自己的博客
published: 2026-01-10
description: 本教程将介绍如何使用 Fuwari 和 Cloudflare Pages 服务搭建一个静态博客。整个过程需要一定的技术基础，但按照步骤操作即可完成
image: ""
tags:
  - 教程
category: 教程
draft: false
lang: ""
---
## 方案特点

这套方案的主要特点：

- **免费托管**：Cloudflare Pages 提供免费的静态网站托管服务
- **CDN 加速**：利用 Cloudflare 的全球节点提供访问加速
- **自动部署**：通过 Git 推送实现自动构建和部署
- **Markdown 写作**：使用 Markdown 格式管理文章内容
- **响应式设计**：Fuwari 支持多设备适配和明暗主题

> [!TIP]
    如觉网络环境欠佳，建议后续操作全程开启魔法。
## 前置准备

### 需要的账号
- [GitHub](https://github.com) 账号：用于代码托管
- [Cloudflare](https://cloudflare.com) 账号：用于网站部署
### 本地环境要求

- **Git**：版本控制工具
  - Windows：从 https://git-scm.com 下载安装
  - macOS：通过 Homebrew 安装 `brew install git`
  - Linux：使用包管理器安装 `apt/yum install git`

- **Node.js**：JavaScript 运行环境（建议 18.0 或更高版本）
  - 下载地址：https://nodejs.org
  - 选择 LTS 版本即可

- **代码编辑器**：推荐 VS Code 或其他你熟悉的编辑器

## Step 1：创建项目

### Fork 仓库

1. 访问 https://github.com/saicaca/fuwari
2. 点击 Fork 按钮创建你自己的副本
3. 克隆到本地：

```bash
git clone https://github.com/你的用户名/fuwari.git
```

```bash
cd fuwari
```

**[图片位置：GitHub Fork 按钮位置]**

### 安装依赖

进入项目目录后，安装必要的依赖包：

```bash
# 安装 pnpm（如果未安装）
npm install -g pnpm
```

```bash
# 安装项目依赖
pnpm install
```

```bash
# 安装图片处理库(通常情况下应该自动安装好了)
pnpm add sharp
```

## Step 2：配置博客信息

### 主要配置文件

编辑 `src/config.ts` 文件，修改为你的个人信息：

```typescript
export const siteConfig: SiteConfig = {
	title: "Fuwari", //博客主标题
	subtitle: "Demo Site", //博客副标题。可选，在首页会显示为“主标题 - 副标题”
	lang: "en", // Language code, e.g. 'en', 'zh_CN', 'ja', etc.
	//博客显示语言。注释已经列出了一些常用的值，如：en, zh_CN, zh_TW, ja, ko
	themeColor: {
		hue: 250, // Default hue for the theme color, from 0 to 360. e.g. red: 0, teal: 200, cyan: 250, pink: 345
		//hue值则是你的博客主题色，注释已经列出了一些常用的值，如红色0,青色250,粉色345
		fixed: false, // Hide the theme color picker for visitors
	},
	banner: {
		enable: false,
		src: "assets/images/demo-banner.png", // Relative to the /src directory. Relative to the /public directory if it starts with '/'
		//banner图片，支持http/https URL
		position: "center", // Equivalent to object-position, only supports 'top', 'center', 'bottom'. 'center' by default
		credit: {
			enable: false, // Display the credit text of the banner image
			text: "", // Credit text to be displayed
			url: "", // (Optional) URL link to the original artwork or artist's page
		},
	},
	toc: {
		enable: true, // Display the table of contents on the right side of the post
		depth: 2, // Maximum heading depth to show in the table, from 1 to 3
	},
	favicon: [
		// Leave this array empty to use the default favicon
		// {
		//   src: '/favicon/icon.png',    // Path of the favicon, relative to the /public directory
		//   theme: 'light',              // (Optional) Either 'light' or 'dark', set only if you have different favicons for light and dark mode
		//   sizes: '32x32',              // (Optional) Size of the favicon, set only if you have favicons of different sizes
		// }
		//网站图标，支持http/https URL
	],
};

export const navBarConfig: NavBarConfig = {
	links: [
		LinkPreset.Home,
		LinkPreset.Archive,
		LinkPreset.About,
		{
			name: "GitHub",
			url: "https://github.com/saicaca/fuwari", // Internal links should not include the base path, as it is automatically added
			//友情链接，这些链接在导航栏上
			external: true, // Show an external link icon and will open in a new tab
		},
	],
};

export const profileConfig: ProfileConfig = {
	avatar: "assets/images/demo-avatar.png", // Relative to the /src directory. Relative to the /public directory if it starts with '/'
	//你的头像，不建议用png之外的后缀
	name: "Lorem Ipsum", //你的名字
	bio: "Lorem ipsum dolor sit amet, consectetur adipiscing elit.", //个性签名，会显示在头像和名字下面
	links: [
		{
			name: "Twitter",
			icon: "fa6-brands:twitter", // Visit https://icones.js.org/ for icon codes
			// You will need to install the corresponding icon set if it's not already included
			// `pnpm add @iconify-json/<icon-set-name>`
			url: "https://twitter.com",
		},
		{
			name: "Steam",
			icon: "fa6-brands:steam",
			url: "https://store.steampowered.com",
		},
		{
			name: "GitHub",
			icon: "fa6-brands:github",
			url: "https://github.com/saicaca/fuwari",
		},
	],
};
//我想我不用讲你也知道是干什么的
```

### 站点配置

修改 `astro.config.mjs` 中的站点 URL：

```javascript
// https://astro.build/config
export default defineConfig({
	site: "https://fuwari.vercel.app/", //这里修改成自己的url，比如这里就应该写成https://blog.sikon.top/
	base: "/",
	trailingSlash: "always",
	integrations: [
		tailwind({
			nesting: true,
		}),
		swup({
			theme: false,
			animationClass: "transition-swup-", // see https://swup.js.org/options/#animationselector
			// the default value `transition-` cause transition delay
			// when the Tailwind class `transition-all` is used
			containers: ["main", "#toc"],
			smoothScrolling: true,
			cache: true,
			preload: true,
			accessibility: true,
			updateHead: true,
			updateBodyClass: false,
			globalInstance: true,
		}),
})
```

### 添加头像

1. 准备一张正方形图片（建议 60x60 像素）
> [!TIP]
    如果没有60x60像素大小图片的话可以去[IloveIMG](https://www.iloveimg.com/zh-cn)调整一下原图像大小

2. 命名为 `avatar.jpg`
3. 放入 `src/assets/favicon/` 目录

## Step 3：创建文章

### 使用命令创建文章

运行以下命令创建新文章：

```bash
pnpm new-post [文章名称]
```

这会在 `src/content/posts/` 目录下生成一个 Markdown 文件。

### 文章格式

文章采用 Markdown 格式，文件开头需要包含 Front Matter 信息：

```markdown
---
title: [文章名称]
published: 2024-01-09
description: '文章简介'
image: './cover.jpg'  # 封面图（可选）
tags: [标签1, 标签2]
category: '分类'
draft: false  # 是否为草稿
---

## 正文开始

这里写文章内容...

### 子标题

支持所有 Markdown 语法：
- 列表项
- **粗体**
- *斜体*
- `行内代码`
- [链接](https://example.com)

代码块示例：
​```javascript
const greeting = "Hello World";
console.log(greeting);
​```

```


> [!TIP]
    > Front Matter 信息一般会在创建新文章时自动生成

### 图片处理

文章中的图片有两种处理方式：

1. **本地存储**：将图片放在文章同目录，使用相对路径引用
2. **外部图床**：使用图床服务，直接引用图片 URL

## Step 4：本地预览

在部署之前，建议先在本地预览效果：

```bash
pnpm dev
```

打开浏览器访问 http://localhost:4321 查看博客效果。


## Step 5：部署到 Cloudflare Pages

### 推送到 GitHub

首先将代码推送到 GitHub 仓库：

```bash
# 如果是新建的项目，需要初始化 Git
git init
git add .
git commit -m "初始提交"

# 在 GitHub 创建新仓库后，添加远程地址
git remote add origin https://github.com/你的用户名/my-blog.git
git push -u origin main
```

### 配置 Cloudflare Pages

1. **登录 Cloudflare 控制台**
   - 访问 https://dash.cloudflare.com
   - 进入 Pages 部分

2. **创建项目**
   - 点击 "Create a project"
   - 选择 "Connect to Git"
   - 授权访问你的 GitHub 账号
   - 选择刚创建的博客仓库

3. **设置构建配置**
   
   填写以下信息：
   - 项目名称：`my-blog`（将影响默认域名）
   - 生产分支：`main`
   - 构建命令：`pnpm build`
   - 构建输出目录：`dist`

4. **开始部署**
   
   点击 "Save and Deploy"，等待构建完成（通常需要 1-2 分钟）。

### 访问网站

部署成功后，你会获得一个默认域名：
- 格式：`项目名.pages.dev`
- 例如：`my-blog.pages.dev

## Step 6：日常维护

### 发布新文章流程

1. 创建文章：
   ```bash
   pnpm new-post [文章名称]
   ```

2. 编辑内容并本地预览：
   ```bash
   pnpm dev
   ```

3. 提交并推送：
   ```bash
   git add .
   git commit -m "添加新文章"
   git push
   ```

4. Cloudflare Pages 会自动检测更新并重新部署

### 更新配置

修改任何配置文件后，同样通过 Git 推送即可触发自动部署。

## 可选配置

### 绑定自定义域名

如果你有自己的域名：

1. 在 Pages 项目设置中选择 "Custom domains"
2. 添加你的域名
3. 根据提示配置 DNS 记录
4. 等待 SSL 证书生成(自动完成)

## 相关资源

- [Fuwari 项目地址及文档](https://github.com/saicaca/fuwari)
- [Cloudflare Pages 文档](https://developers.cloudflare.com/pages/)
- [Astro 文档](https://docs.astro.build/)
- [Markdown 语法参考](https://www.markdownguide.org/)

## 总结

通过以上步骤，你应该已经成功搭建了一个基于 Fuwari 和 Cloudflare Pages 的个人博客。这个方案的主要优势在于免费、稳定且易于维护。后续你只需要专注于内容创作，技术层面的事情都已经自动化处理了。