# Qoder平台Vercel+Vite项目部署规则报告

## 1. 项目初始化规则

### 1.1 技术栈选择
- 前端框架：React
- 构建工具：Vite
- 模块系统：ES Module

### 1.2 配置文件要求
- 必须包含package.json、vite.config.js/ts、vercel.json等核心配置文件
- 根据项目类型（JS/TS）选择对应的配置文件模板

## 2. 部署准备规则

### 2.1 本地验证步骤
1. 执行`npm run lint`进行代码规范检查
2. 执行`npm run build`进行构建验证
3. 执行`npm run preview`进行预览验证

### 2.2 配置微调要求
- 根据实际部署路径调整vite.config.js/ts中的base参数
- 配置正确的API代理目标地址

## 3. Vercel部署规则

### 3.1 部署前检查
- 确认vercel.json路由配置正确
- 确认环境变量配置完整

### 3.2 部署后验证
- 检查部署后的页面访问是否正常
- 验证API接口调用是否成功

## 4. GitHub仓库关联规则

### 4.1 文件组织规范
- 配置文件按版本分类存放在config-templates目录下
- 规则报告存放在docs目录下

### 4.2 同步更新要求
- 配置文件变更后及时推送到GitHub仓库
- Qoder项目定期拉取最新代码保持同步