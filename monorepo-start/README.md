# Monorepo 项目搭建
创建目录结构：
```monorepo/
├── apps/
│   └── main/
├── packages/
│   └── ui-components/
├── pnpm-workspace.yaml
└── package.json
```
配置 `pnpm-workspace.yaml`：
```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```
配置 根目录 `package.json`运行脚本：
**--filter**：只对指定的包运行命令
```json
{
  "scripts": {
    "start:main": "pnpm --filter main start",
    "start:ui": "pnpm --filter ui-components start",
  }
}
```
配置 `apps/main/package.json` 依赖 `ui-components`：
```json
{
  "dependencies": {
    "ui-components": "workspace:*"
  }
}
```