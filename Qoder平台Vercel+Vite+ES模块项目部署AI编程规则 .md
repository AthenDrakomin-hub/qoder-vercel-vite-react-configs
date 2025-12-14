# Qoder平台Vercel+Vite+ES模块项目部署AI编程规则

# 一、核心原则

本规则旨在规范AI在Qoder平台处理Vercel托管的Vite+ES模块项目时的编程行为，确保AI准确理解技术栈特性、部署流程及兼容性要求，避免架构篡改、配置错误等问题，保障项目一次性部署成功。所有AI编程操作需严格遵循“先匹配技术栈、再按规范配置、分步验证、不擅自变更核心架构”的原则。

# 二、技术栈识别规范

## 2.1 技术栈判定标准

- 当项目包含`vite.config.js`/`vite.config.ts`时，直接判定为Vite构建项目；

- 当`package.json`中存在`"type": "module"`，或文件扩展名含`.mjs`/`.mts`，或代码中使用`import`而非`require`时，判定为ES模块项目；

- 当部署目标指定为Vercel时，自动匹配Vercel静态构建适配器（`@vercel/static-build`）及相关路由规则。

## 2.2 禁止误判场景

- 不得将Vite项目误判为Webpack、Rollup等其他构建工具项目；

- 不得将ES模块项目按CommonJS规范处理（如添加`require`语句、删除`"type": "module"`配置）；

- 不得忽略Vercel平台对Node.js版本的要求（必须选用Node.js 20及以上版本，禁止选用已弃用的Node.js 18及以下版本）。

# 三、核心配置编程规范

## 3.1 package.json配置要求

- 必须添加`"type": "module"`字段，确保ES模块正常加载；

- 必须定义标准构建脚本：`"scripts": { "build": "vite build", "preview": "vite preview" }`，不得遗漏或修改为非标准命令；

- 依赖安装需明确指定版本范围，避免因依赖兼容问题导致部署失败，优先选用支持ES模块的依赖包。

## 3.2 vite.config.js配置要求

- 必须通过`defineConfig`函数配置，核心参数不可缺失：`plugins`（匹配项目框架，如React用`@vitejs/plugin-react`、Vue用`@vitejs/plugin-vue`）、`build.outDir`（固定为`"dist"`）；

- 若项目部署在子路径下，必须配置`base`参数（如部署路径为`/app/`，则设置`base: "/app/"`）；

- 路径别名配置需规范：使用`resolve.alias`时，需通过`path.resolve`函数拼接绝对路径（如`alias: { '@': resolve(__dirname, './src') }`）；

- 不得擅自修改构建输出格式，Vite默认ES模块输出格式需保留，无需额外指定`formats`参数（库模式除外）。

## 3.3 vercel.json配置要求

- 必须指定配置版本：`"version": 2`，禁止使用旧版本配置；

- builds配置需固定：`"builds": [{"src": "package.json", "use": "@vercel/static-build"}]`，不得选用其他不匹配的构建适配器；

- 路由配置需适配SPA场景：`"routes": [{"src": "/(.*)", "dest": "/index.html"}]`，确保所有请求重定向到index.html；

- 静态资源目录需明确：添加`"public": "public"`字段，确保Vercel正确处理静态资源。

## 3.4 环境变量配置要求

- 访问Vercel系统环境变量时，必须添加`VITE_`前缀（如`VITE_VERCEL_ENV`），并通过`vite.config.js`的`define`字段注入（如`define: { __APP_ENV__: process.env.VITE_VERCEL_ENV }`）；

- 自定义环境变量需通过Vercel平台项目设置添加，不得硬编码在代码中；敏感信息（如API密钥）需使用加密环境变量，禁止明文暴露；

- 使用`dotenv`加载本地环境变量时，需在`vite.config.js`中先执行`config()`函数，再进行配置。

# 四、构建与部署流程规范

## 4.1 流程步骤顺序

1. 依赖安装：执行`npm install`或`yarn install`，确保所有依赖安装完成，无缺失依赖；

2. 本地构建验证：执行`npm run build`，检查构建过程无错误，`dist`目录生成正常（包含`index.html`和`assets`目录）；

3. 本地预览验证：执行`npm run preview`，验证功能正常、路由跳转无误、静态资源加载成功；

4. 部署配置检查：确认`vercel.json`配置正确，环境变量已添加；

5. 执行部署：通过Vercel CLI（`vercel --prod`）或Qoder平台集成功能发起部署。

## 4.2 流程约束

- 不得跳过本地构建和预览步骤，直接发起部署；

- 构建过程中若出现依赖兼容性错误（如`ERR_REQUIRE_ESM`），需优先解决依赖问题（如升级依赖版本、替换为支持ES模块的依赖），不得强行修改模块系统；

- 部署后需检查Vercel构建日志，若出现红色错误信息（含`Error`关键字），需先修复错误再重新部署，不得忽略错误继续部署。

# 五、AI编程特殊约束

## 5.1 架构保护约束

- 不得擅自修改项目核心架构（如删除Vite配置文件、替换构建工具、更改模块系统类型）；

- 不得随意添加无关依赖或删除必要依赖，所有依赖变更需有明确的兼容性验证依据；

- 修改配置文件时，需保留原有正确配置，仅针对问题部分进行精准修改，不得大面积改写。

## 5.2 错误处理约束

- 遇到部署错误时，需按“配置检查→依赖检查→构建日志分析→分步调试”的逻辑排查，不得盲目尝试修改；

- 常见错误处理方案需严格遵循：ES模块加载失败（如`ERR_REQUIRE_ESM`）：优先检查`package.json`的`"type": "module"`配置，或文件扩展名是否正确，禁止添加`require`语句；

- 404错误：检查`vite.config.js`的`base`参数和`vercel.json`的路由配置，确保静态资源路径正确；

- 500错误：检查环境变量配置、API路由语法及依赖兼容性，需添加错误捕获机制定位问题。

## 5.3 验证与回滚约束

- 所有编程修改后，必须输出本地验证步骤和验证要点，指导用户完成本地测试；

- 需告知用户版本控制和回滚方法：使用Git管理版本，部署失败时可通过Vercel仪表板“Instant Rollback”功能回滚到上一个成功版本；

- 修改核心配置文件（如`vite.config.js`、`vercel.json`）前，需自动备份原有文件（如生成`vite.config.js.bak`），便于回滚恢复。

# 六、规则执行监督

- AI完成编程后，需自动生成“规则符合性检查清单”，逐一核对是否满足本规则所有要求；

- 若存在规则违反项，需明确指出问题所在及修正方案，不得输出违反规则的代码或配置；

- 用户反馈部署问题时，AI需先对照本规则排查是否存在规则未执行情况，再进行问题定位和解决。

# 七、Qoder平台配套配置文件模板

## 7.1 package.json模板（ES模块+Vite基础配置）

```json

{
  "name": "qoder-vercel-vite-react-project",
  "version": "1.0.0",
  "type": "module", // 强制ES模块模式，不可删除（React+ES模块必选）
  "scripts": {
    "dev": "vite", // 开发环境启动命令
    "build": "vite build", // 标准构建命令，不可修改（适配Vercel构建流程）
    "preview": "vite preview", // 本地预览命令，不可修改（部署前验证必备）
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0" // React项目推荐新增代码检查命令
  },
  "dependencies": {
    // React项目核心依赖，指定明确版本范围，确保ES模块兼容性
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.0.0", // Vite核心依赖，版本需≥4.0.0（适配React 18+）
    "@vitejs/plugin-react": "^4.2.0", // React专属Vite插件，不可替换为其他框架插件
    "eslint": "^8.55.0", // 代码检查工具（React项目推荐）
    "eslint-plugin-react": "^7.33.2", // React语法检查插件
    "eslint-plugin-react-hooks": "^4.6.0", // React Hooks规则检查插件
    "eslint-plugin-react-refresh": "^0.4.5" // React热更新适配插件
  }
}
```

## 7.2 vite.config.js模板（适配Vercel部署）

```javascript

import { defineConfig } from 'vite';
import path from 'path';
// React专属Vite插件，固定引入不可修改
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()], // 必须加载React插件，确保JSX语法解析、热更新等功能正常
  build: {
    outDir: 'dist', // 输出目录固定为dist，与Vercel识别规则一致
    assetsDir: 'assets', // 静态资源目录，默认配置可保留
    rollupOptions: {
      // React项目可选配置：优化打包体积，排除不必要的依赖
      external: [],
      output: {
        manualChunks: {
          // 拆分React相关依赖，提升加载性能
          reactVendor: ['react', 'react-dom']
        }
      }
    }
  },
  resolve: {
    alias: {
      // 标准路径别名配置，React项目常用@指代src目录，避免路径错误
      '@': path.resolve(__dirname, './src')
    },
    extensions: ['.js', '.jsx', '.json'] // 自动解析React相关文件扩展名
  },
  server: {
    port: 3000, // 开发服务器端口，可根据需求调整
    open: true, // 启动开发服务器时自动打开浏览器
    proxy: {
      // React项目常用配置：解决本地开发跨域问题（根据实际接口地址调整）
      '/api': {
        target: 'https://your-api-domain.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  // 若部署在子路径（如https://xxx.vercel.app/react-app/），需取消下一行注释并修改路径
  // base: '/react-app/'
});
```

## 7.3 vercel.json模板（Qoder平台-Vercel部署专用）

```json

{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "version": 2, // 固定版本2，不可使用旧版本
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build" // 固定静态构建适配器，不可替换
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html" // SPA路由重定向，确保前端路由正常
    }
  ],
  "public": "public", // 静态资源目录声明，不可修改
  "env": {
    // 环境变量注入配置，Vercel系统变量需加VITE_前缀
    "VITE_VERCEL_ENV": "${VERCEL_ENV}"
  }
}
```

## 7.4 .eslintrc.js模板（React项目专用）

```javascript

module.exports = {
  env: {
    browser: true, // 适配浏览器环境
    es2021: true, // 支持ES2021语法
    node: true // 适配Node.js环境（Vercel构建时需要）
  },
  extends: [
    'eslint:recommended', // 启用ESLint核心推荐规则
    'plugin:react/recommended', // 启用React基础推荐规则
    'plugin:react/jsx-runtime', // 适配React 17+ JSX自动导入模式
    'plugin:react-hooks/recommended' // 启用React Hooks严格规则
  ],
  parser: '@babel/eslint-parser', // 解析React JSX语法及ES模块语法
  parserOptions: {
    ecmaFeatures: {
      jsx: true // 允许解析JSX
    },
    ecmaVersion: 'latest', // 支持最新ECMAScript语法
    sourceType: 'module' // 明确指定为ES模块模式，不可改为script
  },
  plugins: [
    'react',
    'react-hooks',
    'react-refresh' // 配合热更新插件的语法检查
  ],
  rules: {
    // React核心规则配置
    'react/react-in-jsx-scope': 'off', // React 17+无需手动导入React，关闭该规则
    'react/prop-types': 'off', // 若使用TypeScript可关闭，纯JS项目可改为warn/enable
    'react/jsx-no-target-blank': 'warn', // 禁止a标签target="_blank"缺失rel="noopener noreferrer"
    // React Hooks规则配置（强制遵循Hooks使用规范）
    'react-hooks/rules-of-hooks': 'error', // 强制Hooks只能在函数组件/自定义Hooks中使用
    'react-hooks/exhaustive-deps': 'warn', // 检查依赖数组完整性，避免闭包问题
    // 通用JS规则配置
    'no-undef': 'error', // 禁止使用未定义变量
    'no-unused-vars': 'warn', // 未使用变量警告（开发阶段友好）
    'semi': ['error', 'always'], // 强制使用分号
    'quotes': ['error', 'single'] // 强制使用单引号
  },
  settings: {
    react: {
      version: 'detect' // 自动检测React版本，适配不同版本特性
    }
  }
}
```

## 7.6 配置文件使用说明

- 所有模板中的注释可保留作为AI编程指引，实际部署前删除不影响功能；

- React项目需固定使用`@vitejs/plugin-react`插件，严禁混用其他框架插件；

- 依赖版本可更新，但需确保兼容性：Vite≥4.0.0、React≥18.0.0、Node.js≥20.0.0；

- 部署子路径时，仅修改vite.config.js的`base`参数即可，其他配置无需调整；

- 本地开发跨域代理（server.proxy）仅作用于开发环境，部署到Vercel后需通过Vite环境变量或边缘函数处理跨域；

- 执行`npm run lint`前，需确保.eslintrc.js和.eslintignore文件均已配置完成，避免检查无关文件或遗漏规范检查；

- 若项目使用TypeScript，需额外安装对应依赖并调整配置，具体见下方TypeScript专属模板；

- 必须在项目根目录添加`.gitignore`文件（模板见7.6.1节），避免node_modules等依赖目录、构建产物被推送到GitHub仓库。

## 7.6.1 .gitignore标准模板（通用JS/TS项目）

```Plain Text

# 依赖目录（核心忽略项，避免推送node_modules）
node_modules/
# Vite构建产物目录
dist/
# 环境变量文件（含敏感信息）
.env
.env.*
!.env.example
# 编辑器配置目录
.vscode/
.idea/
.sublime-*
# 日志文件
logs/
*.log
# 备份文件
*.bak
# 操作系统临时文件
.DS_Store
Thumbs.db
# Vite缓存目录
.vite/
node_modules/.vite-cache/
# TypeScript编译临时文件（若有）
*.tsbuildinfo
# ESLint缓存文件
.eslintcache
# 打包压缩包
*.zip
*.tar.gz

```

说明：该模板适配本项目技术栈，可直接复用，无需修改；核心作用是阻止依赖文件（node_modules）、构建产物、敏感信息被Git追踪并推送到GitHub。

## 7.7 tsconfig.json模板（React+TypeScript+Vite适配）

```json

{
  "compilerOptions": {
    "target": "ES2020", // 适配ES2020及以上，兼容现代浏览器与ES模块
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"], // 引入必要的类型库
    "module": "ESNext", // 模块系统使用ESNext，兼容Vite的ES模块构建
    "skipLibCheck": true,
    "moduleResolution": "bundler", // 采用打包工具的模块解析方式，适配Vite
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true, // 由Vite负责构建，TypeScript仅做类型检查
    "jsx": "react-jsx", // 适配React 17+ JSX转换，无需额外导入React
    "strict": true, // 开启严格类型检查，提升代码质量
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".", // 基础路径，配合路径别名使用
    "paths": {
      "@/*": ["src/*"] // 匹配vite.config.js中的路径别名，确保类型提示正常
    }
  },
  "include": ["src"], // 类型检查范围
  "references": [{ "path": "./tsconfig.node.json" }] // 关联Node.js相关配置
}
```

```json

{
  // tsconfig.node.json模板（Vite配置文件的TypeScript支持）
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

## 7.8 .eslintrc.js模板（React+TypeScript专属）

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
    'plugin:react-hooks/recommended',
    'plugin:@typescript-eslint/recommended', // 启用TypeScript核心推荐规则
    'plugin:@typescript-eslint/recommended-requiring-type-checking' // 启用严格类型检查相关规则
  ],
  parser: '@typescript-eslint/parser', // 替换为TypeScript专属解析器
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: ['./tsconfig.json'], // 关联tsconfig.json，启用类型检查
    tsconfigRootDir: __dirname
  },
  plugins: [
    'react',
    'react-hooks',
    'react-refresh',
    '@typescript-eslint' // 添加TypeScript专属插件
  ],
  rules: {
    // 继承React相关规则，保持一致性
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off',
    'react/jsx-no-target-blank': 'warn',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    // TypeScript专属规则调整
    '@typescript-eslint/no-unused-vars': ['warn', { argsIgnorePattern: '^_' }], // 忽略下划线开头的未使用参数
    '@typescript-eslint/no-explicit-any': 'warn', // 限制any类型的使用
    '@typescript-eslint/explicit-module-boundary-types': 'off', // 关闭显式模块边界类型要求（开发友好）
    // 通用规则保持不变
    'no-undef': 'error',
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

## 7.9 TypeScript项目额外配置说明

- 依赖安装：需额外执行`npm install typescript @types/react @types/react-dom @types/node @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev`；

- vite.config.ts调整：将vite.config.js重命名为vite.config.ts，保持配置逻辑不变，确保引入路径与类型正确；

- 文件扩展名：React组件文件需从.jsx改为.tsx，普通脚本文件从.js改为.ts；

- .eslintignore无需额外修改，已兼容TypeScript文件的忽略逻辑；

- 部署流程与JavaScript项目一致，无需额外调整vercel.json配置，Vite会自动处理TypeScript的编译构建。

## 7.10 配置文件压缩包使用说明

### 7.10.1 压缩包文件清单

压缩包内含两套配置方案（JavaScript版、TypeScript版），文件按以下结构组织，可根据项目技术选型直接取用对应目录下的文件：

```Plain Text


qoder-vercel-vite-react-config/
├── javascript-version/  # JavaScript项目专用配置
│   ├── package.json
│   ├── vite.config.js
│   ├── vercel.json
│   ├── .eslintrc.js
│   ├── .eslintignore
│   └── .gitignore  # 新增：Git忽略配置，避免依赖推送
└── typescript-version/  # TypeScript项目专用配置
    ├── package.json
    ├── vite.config.ts
    ├── vercel.json
    ├── .eslintrc.js
    ├── .eslintignore
    ├── tsconfig.json
    ├── tsconfig.node.json
    └── .gitignore  # 新增：Git忽略配置，避免依赖推送

```

### 7.10.2 解压与使用步骤

- 解压压缩包，根据项目类型（JS/TS）选择对应版本的目录；

- 将所选目录下的所有文件复制到项目根目录（若项目已有同名文件，建议先备份原有文件，再覆盖）；

- 打开终端，在项目根目录执行`npm install`（TypeScript项目需额外执行7.9.1节的依赖安装命令）；

- 根据项目实际需求微调配置：如部署子路径则修改vite.config.js/ts的`base`参数、调整跨域代理的`target`地址等；

- 执行`npm run build`验证构建是否正常，再通过`npm run preview`本地预览，确认无误后即可部署到Vercel。

### 7.10.3 验证与问题排查

- 配置应用后，优先执行`npm run lint`检查代码规范，修复所有error级错误；

- 若构建失败，查看终端错误信息，优先检查依赖安装是否完整、配置文件中的路径/语法是否正确；

- 部署失败时，查看Vercel构建日志，对照本规则“五、AI编程特殊约束”中的错误处理方案排查。

# 七、配置文件纯文本代码块合集（直接复制创建）

以下代码块按压缩包目录结构整理，每个代码块对应一个配置文件，复制对应代码到项目根目录的同名文件中即可使用。

## 7.1 JavaScript版本配置文件（对应 javascript-version/ 目录）

### 7.1.1 javascript-version/package.json

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

### 7.1.2 javascript-version/vite.config.js

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

### 7.1.3 javascript-version/vercel.json

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

### 7.1.4 javascript-version/.eslintrc.js

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

### 7.1.5 javascript-version/.eslintignore

### 7.1.5 javascript-version/.eslintignore

```Plain Text


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

### 7.1.6 javascript-version/.gitignore

```Plain Text

# 依赖目录（核心忽略项，避免推送node_modules）
node_modules/
# Vite构建产物目录
dist/
# 环境变量文件（含敏感信息）
.env
.env.*
!.env.example
# 编辑器配置目录
.vscode/
.idea/
.sublime-*
# 日志文件
logs/
*.log
# 备份文件
*.bak
# 操作系统临时文件
.DS_Store
Thumbs.db
# Vite缓存目录
.vite/
node_modules/.vite-cache/
# ESLint缓存文件
.eslintcache
# 打包压缩包
*.zip
*.tar.gz

```

## 7.2 TypeScript版本配置文件（对应 typescript-version/ 目录）

### 7.2.1 typescript-version/package.json

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

### 7.2.2 typescript-version/vite.config.ts

```typescript

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
    extensions: ['.ts', '.tsx', '.js', '.json']
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

### 7.2.3 typescript-version/vercel.json

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

### 7.2.4 typescript-version/.eslintrc.js

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
    'plugin:react-hooks/recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking'
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: ['./tsconfig.json'],
    tsconfigRootDir: __dirname
  },
  plugins: [
    'react',
    'react-hooks',
    'react-refresh',
    '@typescript-eslint'
  ],
  rules: {
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off',
    'react/jsx-no-target-blank': 'warn',
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',
    '@typescript-eslint/no-unused-vars': ['warn', { argsIgnorePattern: '^_' }],
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'no-undef': 'error',
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

### 7.2.5 typescript-version/.eslintignore

### 7.2.5 typescript-version/.eslintignore

```Plain Text


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

### 7.2.8 typescript-version/.gitignore

```Plain Text

# 依赖目录（核心忽略项，避免推送node_modules）
node_modules/
# Vite构建产物目录
dist/
# 环境变量文件（含敏感信息）
.env
.env.*
!.env.example
# 编辑器配置目录
.vscode/
.idea/
.sublime-*
# 日志文件
logs/
*.log
# 备份文件
*.bak
# 操作系统临时文件
.DS_Store
Thumbs.db
# Vite缓存目录
.vite/
node_modules/.vite-cache/
# TypeScript编译临时文件
*.tsbuildinfo
# ESLint缓存文件
.eslintcache
# 打包压缩包
*.zip
*.tar.gz

```

### 7.2.6 typescript-version/tsconfig.json

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

### 7.2.7 typescript-version/tsconfig.node.json

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

## 7.3 合集使用提示

- 复制代码时，确保完整复制代码块内所有内容，包括换行和标点；

- 创建文件时，文件名需与标注的路径完全一致（含前缀“.”，如.eslintrc.js、.gitignore）；

- TypeScript项目创建完成后，需先执行专属依赖安装命令（见7.9.1节）再进行后续操作；

- 创建.gitignore文件后，需先执行`git rm -r --cached node_modules`（若已误提交依赖），再提交.gitignore，避免历史依赖文件残留。

## 7.4 Qoder平台直接应用配置的操作步骤

以下步骤适配Qoder平台在线编程特性，可直接关联Vercel部署，无需本地手动创建文件，按顺序执行即可完成配置与部署准备：

### 7.4.1 步骤1：创建Qoder项目并选择技术栈

- 登录Qoder平台，点击「新建项目」，选择「前端项目」分类；

- 技术栈选择「React」，构建工具选择「Vite」，模块系统默认「ES Module」（若平台有选项需手动确认）；

- 项目名称建议与配置文件中`name`字段一致（如`qoder-vercel-vite-react-project`），点击「创建项目」。

### 7.4.2 步骤2：快速导入配置文件（三种方式可选）

#### 方式1：直接复制代码生成文件（推荐，无需本地文件）

- 在Qoder项目根目录，点击「新建文件」，依次输入7.1/7.2节对应版本（JS/TS）的文件名（含前缀“.”，如`.eslintrc.js`、`.gitignore`）；

- 每个文件创建后，打开对应文件，复制文档中“配置文件纯文本代码块合集”里对应文件的完整代码，粘贴到文件编辑区，点击「保存」；

- 重复操作，直至对应版本的所有配置文件（如JS版6个文件、TS版8个文件，含新增的.gitignore）全部创建完成。

#### 方式2：上传配置文件（适合已下载压缩包）

- 在Qoder项目根目录，点击「新建配置文件」打开项目相关文件管理界面；

- 找到「上传配置文件」按钮，选择本地解压后的对应版本配置文件（可批量选择同目录下所有文件）；

- 上传完成后，检查根目录文件列表，确保无遗漏、无文件名错误（如避免`eslintrc.js`缺失前缀“.”）。

#### 方式3：输入配置文件URL导入（适合配置文件已托管在线）

- 在Qoder平台点击「新建配置文件」打开项目，进入文件导入界面；

- 找到「输入配置文件URL」输入框，粘贴对应配置文件的在线URL（需确保URL可公开访问、文件格式正确，如JS/TS/JSON等）；

- 若需导入多个配置文件，可点击「添加URL」按钮批量输入，或按压缩包在线URL批量导入（若平台支持）；

- 点击「确认导入」，等待平台下载并生成文件；导入完成后，检查项目根目录文件，确认所有配置文件完整且无格式错误。

#### 方式1：直接复制代码生成文件（推荐，无需本地文件）

- 在Qoder项目根目录，点击「新建文件」，依次输入7.1/7.2节对应版本（JS/TS）的文件名（含前缀“.”，如`.eslintrc.js`）；

- 每个文件创建后，打开对应文件，复制文档中“配置文件纯文本代码块合集”里对应文件的完整代码，粘贴到文件编辑区，点击「保存」；

- 重复操作，直至对应版本的所有配置文件（如JS版5个文件、TS版7个文件）全部创建完成。

#### 方式2：上传配置文件（适合已下载压缩包）

- 在Qoder项目根目录，点击「上传文件」，选择本地解压后的对应版本配置文件（可批量选择同目录下所有文件）；

- 上传完成后，检查根目录文件列表，确保无遗漏、无文件名错误（如避免`eslintrc.js`缺失前缀“.”）。

### 7.4.3 步骤3：在Qoder平台安装依赖

- 打开Qoder平台的「终端」工具（通常在项目底部）；

- 若为JavaScript项目，直接执行命令：`npm install`；

- 若为TypeScript项目，执行命令：`npm install typescript @types/react @types/react-dom @types/node @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev`（完整命令见7.9.1节）；

- 等待依赖安装完成，终端显示“added xxx packages”即成功（若有依赖冲突，可执行`npm install --force`强制安装，需确保依赖版本兼容）。

### 7.4.4 步骤4：配置微调与本地预览验证

- 根据项目实际需求微调配置：如部署在子路径，打开`vite.config.js/ts`，取消`base`参数注释并修改路径（如`base: '/react-app/'`）；如需跨域，修改`server.proxy.target`为实际API地址；

- 点击Qoder平台的「预览」按钮（通常在顶部），选择「本地预览」，平台会自动执行`npm run dev`，验证项目是否正常启动（无报错、页面可正常渲染）；

- 额外验证：在终端执行`npm run lint`检查代码规范，执行`npm run build`验证构建是否正常，确保无error级错误。

### 7.4.5 步骤5：关联Vercel并触发部署

- 在Qoder平台找到「部署」模块，选择「Vercel部署」（若未关联，需先输入Vercel的API密钥完成授权，密钥可在Vercel个人设置中获取）；

- 授权完成后，平台会自动识别`vercel.json`配置，无需额外设置，点击「触发部署」；

- 部署过程中可查看实时日志，若部署失败，对照文档“5.2 错误处理约束”或“7.10.3 验证与问题排查”章节排查问题，修复后重新触发部署。

### 7.4.6 关键注意事项

- Qoder平台若默认Node.js版本低于20.0.0，需在「项目设置」-「环境配置」中手动切换为Node.js 20+版本；

- 环境变量配置：敏感信息（如API密钥）需在Qoder平台「项目设置」-「环境变量」中添加，变量名需加`VITE_`前缀，部署时会自动同步到Vercel；

- 部署成功后，可通过Vercel生成的域名访问项目，若需自定义域名，在Vercel后台配置，Qoder平台无需额外操作。

### 7.4.7 补充：常见部署错误快速排查表

针对Qoder+Vercel部署场景，整理高频错误及解决方案，快速定位问题：

|错误现象|常见原因|解决方案|
|---|---|---|
|Vercel构建日志显示「ERR_REQUIRE_ESM」|ES模块配置缺失，或依赖为CommonJS类型|1. 检查package.json是否有`"type": "module"`；2. 替换不支持ES模块的依赖为兼容版本；3. 确保文件扩展名为.js/.jsx（JS项目）、.ts/.tsx（TS项目）|
|部署后访问404（前端路由失效）|vercel.json路由配置错误，未适配SPA|确认vercel.json中routes配置为`{"src": "/(.*)", "dest": "/index.html"}`，重新部署|
|Qoder预览时静态资源加载失败（404）|vite.config.js的base参数配置错误，或路径别名错误|1. 未部署子路径时注释base参数；2. 部署子路径时确保base值以“/”开头和结尾（如`base: "/react-app/"`）；3. 检查alias配置中`@`是否指向正确的src目录|
|TypeScript项目构建时类型错误|tsconfig.json配置不完整，或依赖类型未安装|1. 确认tsconfig.json包含`"references": [{ "path": "./tsconfig.node.json" }]`；2. 执行TS专属依赖安装命令（见7.9.1节）；3. 修复代码中的类型错误后重新构建|
|ESLint检查报错「parserOptions.project」缺失|TS项目的.eslintrc.js未关联tsconfig.json|检查.eslintrc.js的parserOptions中是否有`project: ["./tsconfig.json"]`，缺失则补充后保存|
### 7.4.8 补充：Qoder平台进阶优化配置

若项目需要提升构建速度或优化部署性能，可添加以下配置，直接在Qoder平台修改对应文件即可：

- 优化Vite构建速度：在vite.config.js/ts中添加`cacheDir: './node_modules/.vite-cache'`（与plugins同级），开启构建缓存，减少重复构建时间；

- 配置Vercel构建缓存：在vercel.json中添加`"installCommand": "npm install --cache .npm"`（与builds同级），缓存依赖安装文件，提升部署构建速度；

- Qoder平台Git集成（版本控制）：点击Qoder项目的「Git」模块，关联GitHub/GitLab仓库，提交配置文件到仓库，后续部署可直接关联仓库分支，实现自动部署；

- 环境变量区分开发/生产：在Qoder平台「项目设置」-「环境配置」中，为不同环境（开发/生产）添加对应的环境变量（如VITE_API_URL），部署时Vercel会自动匹配对应环境的变量。

### 7.4.9 Qoder平台新增特性适配说明

Qoder平台的代码库索引、记忆功能等特性可进一步提升开发与部署效率，以下为适配本项目的具体配置与使用方法：

- 代码库索引配置：进入「项目设置」-「代码库索引」，开启「自动创建与更新索引」功能，系统会自动为文件数＜10000的代码库维护索引；建议添加「排除目录/文件」：`node_modules/`、`dist/`、`.git/`，减少索引冗余，提升检索速度。

- 全局记忆功能启用：在「项目设置」-「记忆功能」中开启「全局记忆应用」，平台会自动记录项目配置习惯（如路径别名、依赖版本、部署参数）、常用操作流程等；记忆数据随项目迭代自动更新，后续新建同类项目可直接复用配置，无需重复设置。

- 自定义规则添加：通过「项目设置」-「规则管理」-「新增规则」，可导入本文档的AI编程规则（复制规则文本粘贴到规则内容区），或手动创建个性化规则（如代码规范校验、部署前置检查、依赖版本限制等）；规则创建后可设置「自动执行时机」（如代码提交前、部署前），自动规避配置错误。

- MCP服务集成：进入「项目设置」-「服务集成」，找到「MCP服务」并开启；无需修改任何配置文件，平台会自动适配Vite+Vercel部署流程，优化模块间通信效率、智能调度部署资源，降低部署超时风险。

- 智能会话使用：点击平台顶部「智能会话」入口，可直接咨询部署相关问题（如“配置文件导入失败如何解决”“路由404怎么排查”）；支持上传配置文件片段（如vite.config.js、vercel.json），AI会结合本文档规则给出精准解决方案，快速定位问题。
> （注：文档部分内容可能由 AI 生成）