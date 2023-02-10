# 基于 pnpm 的 monorepo 项目实践

## 为什么

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
  - 'apps/*'
  - 'packages/*'
  - '!**/test/**'
```

## 参考资料

- [工作空间（Workspace）](https://pnpm.io/zh/workspaces)
- [pnpm-workspace.yaml](https://pnpm.io/zh/pnpm-workspace_yaml)
- [Monorepo 下的模块包设计实践](https://www.rustc.cloud/monorepo-pkg#be788358fa684f268bf6a2af81db1e17)
