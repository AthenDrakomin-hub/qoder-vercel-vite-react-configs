---
trigger: model_decision
description: 项目初始化阶段（选择 React+Vite+ES Module 技术栈时生效）
---
# 轻量智能体执行规则（关联Vercel+Vite部署工作区）
## 1. 核心技术栈约束
- 框架：React
- 构建工具：Vite 5.0.0+
- 模块系统：ES Module（package.json需含"type": "module"）
- 部署目标：Vercel（依赖vercel.json配置）
- 版本选择：优先JavaScript版本（5个核心配置文件），如需TS则额外加载tsconfig系列文件

## 2. 必执行流程（严格按工作区2.2章节顺序）
1. 项目初始化：新建前端项目→选React+Vite+ES Module
2. 配置导入：从config-templates目录拉取对应版本文件（或通过URL导入）
3. 依赖安装：JS项目执行npm install；TS项目执行专属安装命令
4. 配置微调：修改vite.config的proxy目标、base参数（如需）
5. 本地验证：依次执行npm run lint → npm run build → npm run preview
6. 部署触发：关联Vercel授权后执行部署

## 3. 关键校验标准
- 配置文件完整性：JS项目需5个文件，TS项目需7个文件，缺一不可
- 构建无错误：执行npm run build后无error级日志
- 路由有效性：部署后需满足SPA路由规则（vercel.json路由配置正确）
- 特性启用要求：必须开启代码库索引（排除node_modules/dist）、全局记忆功能

## 4. 错误处理准则（复用工作区4.4排查区）
- 遇ERR_REQUIRE_ESM：优先检查package.json的"type": "module"配置
- 部署404：验证vercel.json routes配置为{"src": "/(.*)", "dest": "/index.html"}
- 静态资源加载失败：检查vite.config的base参数和@别名配置
- TS类型错误：确认tsconfig.json包含references关联tsconfig.node.json