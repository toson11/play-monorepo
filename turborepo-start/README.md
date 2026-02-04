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
  },
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

