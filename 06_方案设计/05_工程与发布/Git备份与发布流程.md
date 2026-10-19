# Git 备份与发布流程（GitHub: yan-mc）

> 适用：AMS 全部仓库（启动器/内核 fork/插件/服务/发布清单/文档）。
> 本机现状（2026-10-19 只读检查）：已安装 Git；**未安装 gh CLI**；`git config --global user.name/user.email` 为空；GitHub 登录态在凭据管理器中（用户已登录）。
> 本文是操作规范，不含产品代码。

## 1. 仓库规划

| 仓库 | 类型 | 可见性建议 | 说明 |
|---|---|---|---|
| `ams-launcher` | Tauri 启动器 | 先 Private，发布后 Public | 独立安装包/更新 |
| `ams-harness` | DSH fork | 先 Private（或 Public，随上游 MIT） | 保留 `upstream` remote；只放品牌/构建/裁剪 patch |
| `ams-plugins` | pnpm monorepo | Private 开发 → 发布后 Public | 每个插件独立版本、独立 tarball |
| `ams-services` | uv/Python monorepo | Private → Public | gateway + 各后端服务 |
| `ams-release` | 更新清单/脚本/公钥 | Public（更新 feed 需要） | Release 资产 + manifest + 签名 |
| `ams-docs` | 文档 | Public（可选） | 镜像 `06_方案设计` + 用户手册 |

命名统一 `ams-` 前缀；默认分支 `main`；所有仓库开启 Issues/Projects（可选）、Actions。

## 2. 提交身份与首次推送

1) 设置提交身份（一次即可，建议与 GitHub 账号一致）：
~~~powershell
git config --global user.name "yan-mc"
git config --global user.email "<你的GitHub邮箱>"
~~~
2) 在 GitHub `github.com/yan-mc` 网页新建空仓库（不要初始化 README），然后本地：
~~~powershell
git init
git add -A
git commit -m "chore: initial commit"
git branch -M main
git remote add origin https://github.com/yan-mc/<repo>.git
git push -u origin main
~~~
3) 凭据：本机 gh 未安装也可用 HTTPS + Git Credential Manager（首次 push 弹浏览器登录）；若安装 gh CLI，可 `gh auth login` 后用 `gh repo create`。
4) 建议开启 2FA、SSH 或 PAT（细粒度、仅 repo 权限）；**不要把 token 写进脚本/文档**。

## 3. 分支与提交规范

| 分支 | 用途 |
|---|---|
| `main` | 可发布、受保护（禁止强推，必须 PR/检查通过） |
| `dev` | 集成分支（可选，小团队可省） |
| `feat/<scope>` | 功能；`fix/<scope>` 修复；`chore/*` 工程 |
| `release/<version>` | 发布候选（可选） |
| `upstream-sync` | 仅 `ams-harness`：定期合并上游 |

提交信息：Conventional Commits（`feat:`, `fix:`, `docs:`, `build:`, `chore:`），中文说明可用；每个 PR 关联 Issue；合并用 Squash。

## 4. 版本号与标签

- 全部组件 SemVer：`MAJOR.MINOR.PATCH`；预发布 `-beta.N`/`-rc.N`。
- Tag 规则：`v<version>`（如 `v0.3.0`）；monorepo 内插件用 `<plugin-id>@<version>` 二级 tag（如 `music.trainer@1.2.0`）。
- 版本来源单一：`package.json`/`pyproject.toml` + `CHANGELOG.md`；CI 校验 tag 与版本一致。
- 内核 `ams-harness` 额外记录 `upstream.json`（上游 commit、对应官方版本、我们的 patch 列表）。

## 5. CI（GitHub Actions，windows-latest）

每个仓库最小流水线：
1. **ci.yml**：安装依赖 → 类型检查/lint → 单元测试 → 构建（不签名）；PR 必跑。
2. **release.yml**（tag 触发）：构建 → 打包（NSIS/zip/tarball/wheel）→ 生成 sha256 → 签名（Windows EV/云签名；插件 tarball 由 CI 生成哈希）→ 发布 GitHub Release → 触发生成/提交更新清单到 `ams-release`。
3. **upstream-sync.yml**（仅内核）：定时检查上游新 tag，创建合并 PR（人工审核冲突）。
4. **backup.yml**（每周）：`git bundle create ams-<repo>-<date>.bundle --all` 作为 Release/Artifact 备份（GitHub 之外的本地副本也保留一份）。
5. **codeql/dependabot**（可选）：依赖安全与更新提示。

Secrets：`SIGN_CERT`/`SIGN_PASSWORD`、`ED25519_PRIVATE_KEY`、`GH_TOKEN`、可选 CDN 凭据；只在受保护分支/环境可用。

## 6. 发布流程（组件独立发布）

~~~text
1. 功能合入 main → 2. 更新 CHANGELOG/版本号 → 3. 打 tag → 4. CI 构建+签名 →
5. 发布 GitHub Release（资产：安装包/tarball/wheel/sha256）→ 6. 生成更新清单（ams-release）→
7. 签名清单 → 8. 推送 stable/beta 通道 → 9. 启动器端验证更新/回滚 → 10. 公告
~~~

发布粒度：
- 启动器：全量安装包 + 可选差分；
- 内核：全量安装包（NSIS），非必要不发；发版必须写兼容矩阵与迁移说明；
- 插件：`.tgz` 包 + manifest 条目，可单独更新；
- 服务：wheel/便携 zip + 运行时版本约束；
- 工具/模型目录：清单 + 文件包（不进 Git）。

## 7. 更新清单与分发

- 优先用 **GitHub Releases** 作为下载源（免费、可校验），国内再挂镜像/CDN；清单 URL 可配置。
- `ams-release` 仓库保存 `manifests/<channel>/manifest.json` 与签名公钥；私钥只在 CI Secret。
- Manifest 字段见 `06_方案设计/00_总体设计与约定.md` 第 11 节；生成脚本校验：版本递增、URL 可达、sha256 匹配、签名有效。
- 回滚：保留上一版本资产至少 90 天；启动器本地保留快照。

## 8. 备份策略

| 层级 | 手段 | 频率 |
|---|---|---|
| 代码 | GitHub 仓库 + 分支保护 | 每次 push |
| 发布物 | GitHub Releases + 本地/对象存储副本 | 每次发布 |
| 文档 | `ams-docs` + 本目录 | 每次修改 |
| 应急 | `git bundle --all` 周备份（Artifact + 本地盘） | 每周 |
| 密钥 | 离线密码管理器/硬件令牌，另存恢复流程 | 创建时 |

用户资产（音源/模型/工程）**不进 Git**；提供应用内导出/备份功能（见插件方案）。

## 9. 安全与合规

- 仓库私有期不公开代码；发布后保留 LICENSE/NOTICE（DSH MIT、第三方依赖许可）。
- `ams-harness` 保留上游版权声明与 `THIRD_PARTY_NOTICES`；品牌改名，不暗示 DeepSeek 官方背书。
- 代码签名证书与私钥分离保存；Ed25519 私钥仅 CI；泄露立即轮换并作废旧清单。
- 依赖锁定（pnpm-lock/uv.lock），CI 校验锁文件；禁止提交模型/音源/二进制大文件（Git LFS 仅限小图标可选）。
- 发布前检查：无遥测、无密钥、无用户数据、无未授权权重。

## 10. 发布检查清单（Release Checklist）

1) 版本号/CHANGELOG 已更新；2) CI 全绿；3) 安装包/包已签名；4) 更新清单与签名已生成并验证；5) 兼容矩阵已更新（kernel/plugin/service）；6) 安装/升级/回滚三路径实测；7) 隐私与许可文件齐全；8) Release Notes 已写；9) 旧版本资产保留；10) 通知渠道已发布。
