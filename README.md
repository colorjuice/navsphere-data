# navsphere-data
🚀 NavSphere 项目部署与运维备忘手册
这份文档记录了本项目在 GitHub 和 Cloudflare Pages 上的所有关键配置、修改记录以及日后的维护指南，方便日后随时查阅。

1. 🏗️ 项目整体架构
组件	对应位置	作用描述
前端程序仓库	GitHub: colorjuice/NavSphere	Next.js 源码，负责渲染前端页面和后台管理界面
数据后端仓库	GitHub: colorjuice/navsphere-data	充当数据库，存放 JSON 数据文件和上传的图片
边缘计算托管	Cloudflare Pages	部署 Next.js 程序，绑定自定义域名，提供全局 CDN 加速
2. 🐙 GitHub 侧配置汇总
A. 源码仓库修改记录 (colorjuice/NavSphere)
我们对源代码进行了以下 4 处关键修复和定制：

wrangler.toml (指定 CF 编译目录)：
toml

name = "navsphere"
compatibility_date = "2024-11-11"
compatibility_flags = ["nodejs_compat"]
pages_build_output_dir = ".vercel/output/static"
src/app/page.tsx (修复首页动态数据读取 Bug)：
将原作者硬编码的静态 JSON 引用修改为从 getFileContent() 动态读取。
src/components/site-favicon.tsx (图片 CDN 自动转换)：
增加了自动转换逻辑：将自动获取/上传的 /assets/xxx.png 相对路径自动转换为 https://cdn.jsdelivr.net/... CDN 链接，无需手动粘贴 URL。
src/components/navigation-content.tsx (顶部图标精简)：
精简了右上角图标，仅保留 暗夜模式切换 (ModeToggle) 和 帮助问号 (HelpCircle)。
B. 数据库仓库结构 (colorjuice/navsphere-data)
数据仓库的目录结构如下：

text

colorjuice/navsphere-data/
└── src/
    └── navsphere/
        └── content/
            ├── navigation.json         # 网址导航的分类与书签数据
            ├── site.json               # 站点名称、描述、外观配置
            ├── videos.json             # 视频导航数据
            └── resource-metadata.json  # 图片上传元数据索引
└── public/
    └── assets/                         # 存放自动抓取或手动上传的图片/Icon
C. GitHub OAuth App 配置
访问 GitHub Developer Settings → OAuth Apps：

Application name: NavSphere
Homepage URL: https://你的自定义域名
Authorization callback URL: https://你的自定义域名/api/auth/callback/github
3. ☁️ Cloudflare Pages 侧配置汇总
A. 构建与部署设置 (Build Settings)
在 CF Pages 项目 → Settings → Build & deployments：

配置项	填写内容
构建命令 (Build command)	pnpm install && pnpm dlx @cloudflare/next-on-pages && rm -rf .next/cache .vercel/output/static/cache
构建输出目录 (Build output directory)	.vercel/output/static (由 wrangler.toml 锁定)
根目录 (Root directory)	/ (留空)
兼容性标记 (Compatibility flags)	添加 nodejs_compat
B. 环境变量与密钥 (Environment Variables & Secrets)
在 CF Pages 项目 → Settings → Variables and Secrets（使用“密钥”类型添加）：

变量名	填写内容	说明
NODE_VERSION	20	Node.js 版本
GITHUB_CLIENT_ID	OAuth App 的 Client ID	GitHub 登录
GITHUB_CLIENT_SECRET	OAuth App 的 Client Secret	GitHub 登录机密
GITHUB_OWNER	colorjuice	GitHub 账号名
GITHUB_REPO	navsphere-data	数据仓库名
GITHUB_BRANCH	main	数据仓库分支
AUTH_SECRET	随机生成字符串	NextAuth 加密密钥
NEXTAUTH_SECRET	随机生成字符串	与 AUTH_SECRET 保持一致
NEXTAUTH_URL	https://你的自定义域名	站点完整 URL
NEXT_PUBLIC_API_URL	https://你的自定义域名	API 地址
4. 🛠️ 日常维护与故障排查指南
Q1：如何在后台管理内容？
访问 https://你的自定义域名/admin，使用你的 GitHub 账号登录。
在后台可以自由添加/修改/拖拽排序网址分类和书签。保存后系统会自动提交写入 navsphere-data 仓库。
Q2：如果登录弹出 "Server error: There is a problem with the server configuration"？
检查 GitHub OAuth App 的 Callback URL 是否与当前域名完全匹配。
检查 CF Pages 里的 NEXTAUTH_URL 是否填了最新域名。
修改环境变量后，必须去 CF 点击 Retry deploy (重新部署) 才能对 Node 运行时生效。
Q3：如果前台内容没有更新？
强刷缓存：按 Ctrl + F5 强制刷新浏览器。
清除 CDN 缓存：去 CF 域名管理页面 → Caching → Configuration → Purge Everything。
Q4：更换域名时需要修改什么？
GitHub OAuth App：更新 Homepage URL 和 Callback URL。
CF Pages 环境变量：更新 NEXTAUTH_URL 和 NEXT_PUBLIC_API_URL。
重新部署 CF Pages 项目。
