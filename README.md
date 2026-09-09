# Ignis for LazyCat

Run Obsidian as a self-hosted web app. Not remote desktop, an actual web app.

Ignis 将 Obsidian 适配为浏览器中的 Web 应用，笔记保存在懒猫微服。上游：<https://github.com/Nystik-gh/ignis>。

## 安装与数据

设置向导的所有参数均为必填并提供默认值：PUID=1000、PGID=1000、PORT=8080、OBSIDIAN_VERSION=1.12.7。目录由应用固定管理，无需填写宿主机路径。

| 容器路径 | 懒猫路径 | 用途 |
|---|---|---|
| `/vaults` | `/lzcapp/var/vaults` | 笔记库，每个子目录一个 Vault |
| `/app/data` | `/lzcapp/var/data` | 服务插件设置、同步状态和令牌 |
| `/app/obsidian-app` | `/lzcapp/cache/obsidian-app` | 已下载解包的 Obsidian，可重建缓存 |

三个目录在重启和升级时保留；缓存清理后会重新下载 Obsidian。笔记位于应用数据中，卸载前请备份或导出。首次启动需要访问 GitHub 下载 Obsidian，并通过 npm 安装相关工具，耗时取决于网络。

Ignis 没有内置鉴权，入口使用懒猫身份认证，不开放 `public_path`。已授权访问应用的用户共享笔记库。已接入懒猫文件选择器拦截。

离线安装可由维护者将 Obsidian 包放入现有挂载目录，并配置 `OBSIDIAN_PACKAGE` 指向容器内 `.deb`、`.asar.gz` 或 `.asar` 文件。该可选项不作为必填向导字段，避免默认值导致启动失败。

## 构建与发布

```sh
lzc-cli project release -o dist/ignis.lpk
lzc-cli lpk info dist/ignis.lpk
actionlint
```

包名：`community.lazycat.app.ignis`；目标：`linux/amd64`。使用上游 `nobbe/ignis:latest` 的镜像加速地址，按架构摘要锁定内容。

`.github/workflows/lazycat.yml` 调用 `ca-x/lazycat-github-action/.github/workflows/lazycat.yml@v1`，定时或手动检测 latest 摘要。摘要改变后递增打包版本的 patch，摘要不变则不升级。打包版本初始为 0.8.10，后续并不等同于上游版本。镜像与上游摘要必须一致。

自动生成版本化 Release 资产 `<package>-v<version>.lpk`，验证 SHA256 后发布喵喵商店；不发布官方商店。组织或仓库需配置 `APPSTORE_URL`、`APPSTORE_TOKEN`；私有分组可使用 `PRIVATE_STORE_GROUP_CODES`。不在仓库保存凭据。

图标由用户提供。Ignis 上游采用 AGPL-3.0-or-later；Obsidian 是独立软件，由容器在首次运行时下载，使用遵循其自身条款。
