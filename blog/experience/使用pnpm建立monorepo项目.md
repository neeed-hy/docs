# 使用 pnpm 建立 monorepo 项目

## 初始化

```shell
# 全局安装pnpm
npm install -g pnpm
# 初始化package.json
pnpm init
# 建立pnpm-workspace.yaml
touch pnpm-workspace.yaml
```

最基本的 pnpm-workspace.yaml 文件内容：

```yaml
packages:
  # all packages in direct subdirs of packages/
  - 'packages/*'
  # exclude packages that are inside test directories
  - '!**/test/**'
```

## 参考资料

- [工作空间（Workspace）](https://pnpm.io/zh/workspaces)
- [pnpm-workspace.yaml](https://pnpm.io/zh/pnpm-workspace_yaml)
