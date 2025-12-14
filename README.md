# Qoder平台Vercel+Vite项目部署配置

这个仓库包含了用于在Qoder平台上快速创建和部署Vite+React项目的配置模板。

## 目录结构

```
├── config-templates/
│   ├── javascript-version/     # JavaScript版本配置模板
│   └── typescript-version/     # TypeScript版本配置模板
└── docs/                       # 文档资料
```

## 配置模板说明

### JavaScript版本
位于 `config-templates/javascript-version/` 目录下，包含：
- `package.json` - 项目依赖和脚本配置
- `vite.config.js` - Vite构建配置
- `vercel.json` - Vercel部署配置
- `.eslintrc.js` - ESLint规则配置
- `.eslintignore` - ESLint忽略配置
- `.gitignore` - Git忽略配置

### TypeScript版本
位于 `config-templates/typescript-version/` 目录下，包含：
- `package.json` - 项目依赖和脚本配置
- `vite.config.ts` - Vite构建配置
- `vercel.json` - Vercel部署配置
- `tsconfig.json` - TypeScript编译配置
- `tsconfig.node.json` - Node.js环境TypeScript配置
- `.eslintrc.js` - ESLint规则配置
- `.eslintignore` - ESLint忽略配置
- `.gitignore` - Git忽略配置

## 使用方法

1. 选择适合的版本（JavaScript或TypeScript）
2. 将所选版本目录下的所有文件复制到您的项目根目录
3. 根据需要修改配置文件
4. 安装依赖：`npm install`
5. 启动开发服务器：`npm run dev`
6. 构建生产版本：`npm run build`
7. 部署到Vercel：使用Vercel CLI或连接GitHub仓库

## 部署到Vercel

所有配置均已针对Vercel平台优化：
- 使用 `@vitejs/plugin-react` 插件
- 正确配置了静态资源处理
- 设置了适当的代理规则
- 配置了环境变量支持

只需在Vercel上导入项目并选择正确的框架（Vite），即可自动完成部署。

## 注意事项

- 确保Node.js版本 >= 16
- 推荐使用npm作为包管理器
- 如需自定义部署路径，请修改 `vite.config.js/ts` 中的 `base` 配置