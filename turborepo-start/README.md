# 引入 Turborepo

# 初始化项目

1. 初始化 npm 项目

```bash
npm init -y
```

2. 添加 pnpm-workspace.yaml 文件

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

3. 创建共享组件库 packages/ui-components

4. 创建应用 apps/vite-app1

```bash
npm create vite
```

5. vite-app1 引入 ui-components 组件库，并使用 Button 组件
   package.json 添加依赖

```json
"dependencies": {
  "ui-components": "workspace:*"
}
```

6. 安装依赖

```bash
pnpm install
```

# 引入 Turborepo

1. 安装 Turborepo

```bash
pnpm add turbo --D --w
```

2. 添加 turbo.json 配置文件

```json
{
  // 给 编辑器用的 schema，用于代码提示和校验
  "$schema": "https://turborepo.dev/schema.json",
  // 定义调度任务，任务名对应 package.json 里的脚本名
  "tasks": {
    "dev": {
      // 告知 Turborepo 该任务是一个持续运行的任务
      "persistent": true,
      // 禁用缓存
      "cache": false
    },
    "build": {
      // 指定构建输出目录，作为缓存依据
      "outputs": ["dist/**"]
    },
    "check-types": {
      // 在执行本任务前，先执行依赖包的同名任务
      "dependsOn": ["^check-types"]
    }
  }
}
```

3. 根目录 package.json 添加 turborepo 运行脚本一件启动多个应用

```json
"scripts": {
  "dev": "turbo run dev",
  "dev:app1": "pnpm --filter vite-app1 dev",
  "dev:app2": "pnpm --filter vite-app2 dev"
}
```

# 用 eslint+prettier 规范代码

- eslint 负责代码质量，即影响代码正确性的规则
  - plugin 主要提供规则
  - parser 主要负责代码解析
- prettier 负责代码格式化，即影响代码美观性的规则

### 1. 安装依赖

- **eslint**: JavaScript 和 TypeScript 的代码质量和风格检查工具
- **prettier**: 代码格式化工具
- **@typescript-eslint/eslint-plugin**: 提供 TypeScript 相关的 ESLint 规则
- **@typescript-eslint/parser**: 允许 ESLint 解析 TypeScript 代码
- **eslint-config-prettier**: 关闭所有与 Prettier 冲突的 ESLint 规则
- **eslint-plugin-prettier**: 将 Prettier 作为 ESLint 规则运行，让 ESLint 插件能够报告 Prettier 格式化问题
- **eslint-plugin-vue**: 提供 Vue.js 相关的 ESLint 规则
- **vue-eslint-parser**: 允许 ESLint 解析 Vue 单文件组件（SFC）

```bash
pnpm add -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser eslint-config-prettier eslint-plugin-prettier prettier eslint-plugin-vue
```

### 2. 配置 `eslint.config.js` + `.prettierrc`

- `eslint.config.js`

```javascript
const vue = require('eslint-plugin-vue');
const tseslint = require('@typescript-eslint/eslint-plugin');
const tsparser = require('@typescript-eslint/parser');
const vueParser = require('vue-eslint-parser');
const prettier = require('eslint-plugin-prettier');

module.exports = [
  {
    files: ['**/*.vue'],
    languageOptions: {
      parser: vueParser,
      parserOptions: {
        parser: tsparser,
        ecmaVersion: 2021,
        sourceType: 'module',
      },
    },
    plugins: {
      vue,
      '@typescript-eslint': tseslint,
      prettier,
    },
    rules: {
      ...vue.configs.recommended.rules,
      ...tseslint.configs.recommended.rules,
      // 让 ESLint 能够报告 Prettier 格式化问题
      'prettier/prettier': 'warn',
      // 自定义规则
      '@typescript-eslint/no-unused-vars': 'error',
      '@typescript-eslint/no-explicit-any': 'warn',
    },
  },
  {
    files: ['**/*.{js,ts}'],
    languageOptions: {
      parser: tsparser,
      ecmaVersion: 2021,
      sourceType: 'module',
    },
    plugins: {
      '@typescript-eslint': tseslint,
    },
    rules: {
      ...tseslint.configs.recommended.rules,
      'prettier/prettier': 'warn',
    },
  },
];
```

- `.prettierrc`

```json
{
  "singleQuote": true,
  "printWidth": 100
}
```

### 3. 配置 VSCode 自动格式化

`.vscode/settings.json`
注意：.vscode必须在工作区根目录下才能被 VSCode 识别到

```json
{
  // 启用 ESLint 插件对 JavaScript、TypeScript 和 Vue 文件进行校验
  "eslint.validate": ["javascript", "typescript", "vue"],
  // 针对eslint配置文件不在根目录的情况
  "eslint.workingDirectories": [{ "mode": "auto" }],
  "editor.formatOnSave": true,
  // 优先使用 Prettier 进行格式化，Prettier 处理格式化问题更快快、覆盖面广、体验好（非必须，因为 ESLint 也可以格式化Prettier规则）
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  // 保存自动修复 ESLint 问题
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  }
}
```
