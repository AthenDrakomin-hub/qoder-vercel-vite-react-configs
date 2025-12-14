# Qoder平台Vercel+Vite项目部署工作区

# 一、工作区概述

本工作区整合Qoder平台部署Vercel+Vite+ES模块项目的全流程资源，包含配置文件模板、操作步骤、特性适配指南及问题排查方案，可直接关联GitHub仓库使用，实现配置-开发-部署一体化管理。

# 二、核心组成模块

## 2.1 配置文件模板区（直接复用）

### 2.1.1 按技术版本分类

- JavaScript版本（5个核心文件）：package.json、vite.config.js、vercel.json、.eslintrc.js、.eslintignore

- TypeScript版本（7个核心文件）：在JS版本基础上增加tsconfig.json、tsconfig.node.json

### 2.1.2 快速获取方式

- 复制本工作区“配置文件纯文本合集”章节的代码，直接在Qoder创建对应文件

- 通过GitHub仓库关联，直接拉取仓库中config-templates目录下的对应版本文件

- 使用Qoder“输入配置文件URL”功能，导入在线托管的配置文件链接

## 2.2 操作指南区（分步执行）

### 2.2.1 项目初始化流程

1. Qoder平台新建项目：选择「前端项目」→ 技术栈「React」→ 构建工具「Vite」→ 模块系统「ES Module」

2. 导入配置文件：选择工作区2.1.2中的任意一种方式，完成对应版本配置文件导入

3. 安装依赖：打开Qoder终端，JS项目执行`npm install`，TS项目执行专属依赖安装命令

### 2.2.2 部署准备与执行

1. 配置微调：根据项目需求修改vite.config.js/ts的base参数、proxy目标地址等

2. 本地验证：执行`npm run lint`（代码规范检查）、`npm run build`（构建验证）、`npm run preview`（预览验证）

3. 关联Vercel：Qoder「部署」模块选择「Vercel部署」，输入API密钥授权，触发部署

### 2.2.3 GitHub仓库关联流程

1. GitHub创建仓库，按规范上传文件：将工作区配置文件按`config-templates/javascript-version/`、`config-templates/typescript-version/`目录组织，规则报告放在`docs/`目录

2. Qoder关联仓库：进入Qoder「Git」模块，选择「关联已有仓库」，输入GitHub仓库地址完成绑定

3. 同步更新：后续配置或规则更新后，推送至GitHub仓库，Qoder项目拉取最新代码即可同步

## 2.3 Qoder特性适配区（优化效率）

- 代码库索引：「项目设置」-「代码库索引」开启自动索引，排除node_modules/、dist/等目录

- 全局记忆功能：「项目设置」-「记忆功能」开启全局应用，自动复用配置习惯和操作流程

- 自定义规则：「项目设置」-「规则管理」导入工作区关联的AI编程规则，设置部署前自动执行

- MCP服务：「项目设置」-「服务集成」开启MCP服务，优化模块通信和部署资源调度

- 智能会话：平台顶部「智能会话」入口，咨询部署问题或上传配置片段获取解决方案

## 2.4 问题排查区（快速定位）

|错误现象|解决方案|
|---|---|
|Vercel构建日志「ERR_REQUIRE_ESM」|1. 检查package.json是否有`"type": "module"`；2. 替换不支持ES模块的依赖；3. 确认文件扩展名为.js/.jsx（JS）、.ts/.tsx（TS）|
|部署后访问404（前端路由失效）|确认vercel.json路由配置为`{"src": "/(.*)", "dest": "/index.html"}`，重新部署|
|Qoder预览静态资源加载失败|1. 未部署子路径时注释vite.config的base参数；2. 部署子路径确保base值以“/”开头结尾；3. 检查@别名是否指向src目录|
|TS项目构建类型错误|1. 确认tsconfig.json包含`"references": [{"path": "./tsconfig.node.json"}]`；2. 安装TS专属依赖；3. 修复代码类型错误|
# 三、配置文件纯文本合集（直接复制）

## 3.1 JavaScript版本配置

### 3.1.1 package.json

```json

{
  "name": "qoder-vercel-vite-react-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "eslint": "^8.55.0",
    "eslint-plugin-react": "^7.33.2",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.5"
  }
}
```

### 3.1.2 vite.config.js

```javascript

import { defineConfig } from 'vite';
import path from 'path';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    rollupOptions: {
      external: [],
      output: {
        manualChunks: {
          reactVendor: ['react', 'react-dom']
        }
      }
    }
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    },
    extensions: ['.js', '.jsx', '.json']
  },
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'https://your-api-domain.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
  // base: '/react-app/' // 部署子路径时取消注释并修改
}
```

### 3.1.3 vercel.json

```json

{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ],
  "public": "public",
  "env": {
    "VITE_VERCEL_ENV": "${VERCEL_ENV}"
  }
}
```

### 3.1.4 .eslintrc.js

```javascript

module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended'
  ],
  parser: '@babel/eslint-parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: [
    'react',
    'react-hooks',
    'react-refresh'
  ],
  rules: {
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off',
    'react/jsx-no-target-blank': 'warn',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    'no-undef': 'error',
    'no-unused-vars': 'warn',
    'semi': ['error', 'always'],
    'quotes': ['error', 'single']
  },
  settings: {
    react: {
      version: 'detect'
    }
  }
}
```

### 3.1.5 .eslintignore

```plaintext

# 依赖目录，无需ESLint检查
node_modules/
# 构建产物目录，无需检查
dist/
# 版本控制目录
.git/
# 编辑器配置目录
.vscode/
# 环境变量文件
.env
.env.*
!.env.example
# 其他静态资源目录（若有）
public/
# 备份文件
*.bak
# 日志文件
logs/
```

## 3.2 TypeScript版本配置（额外补充文件）

### 3.2.1 package.json

```json

{
  "name": "qoder-vercel-vite-react-ts-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "@vitejs/plugin-react": "^4.2.0",
    "eslint": "^8.55.0",
    "eslint-plugin-react": "^7.33.2",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.5",
    "typescript": "^5.2.2",
    "@types/react": "^18.2.37",
    "@types/react-dom": "^18.2.15",
    "@types/node": "^20.9.2",
    "@typescript-eslint/parser": "^6.11.0",
    "@typescript-eslint/eslint-plugin": "^6.11.0"
  }
}
```

### 3.2.2 tsconfig.json

```json

{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 3.2.3 tsconfig.node.json

```json

{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

# 四、工作区使用说明

- 配置复用：直接复制“配置文件纯文本合集”中的代码，在Qoder新建对应文件粘贴即可

- 流程执行：按“2.2 操作指南区”的步骤顺序执行，每个步骤完成后可通过工作区“问题排查区”验证是否正常

- 仓库关联：按“2.2.3  GitHub仓库关联流程”组织仓库文件，关联后可实现工作区与仓库的同步更新

- 特性优化：部署前务必开启“2.3 Qoder特性适配区”的相关功能，提升开发和部署效率