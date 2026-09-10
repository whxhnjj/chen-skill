# 飞鱼神图部署

## 最新本次上传宝塔部署如何操作？

日期：2026-09-10。

### 本次范围与适用前提

- 飞鱼神图从仅管理员使用改为所有已登录 Harness 成员使用。成员共用一个 Token、历史和图片，均可替换 Token、生图、下载和重试；保留登录与 CSRF 校验。
- 本说明适用于仓库 `9458f64` 所带模板及本批修改。线上版本尚未核对；若曾自行修改栏目，先比较差异，不能直接假定本批文件覆盖全部历史变更。
- 本地修改和测试不代表服务器已更新。本次未部署服务器。

### 本地准备与上传

1. 将本次完整 `amazon-image-generator/` 源包上传到服务器独立的 Skill 源包目录。不要把源包目录选成正在运行的栏目目录。该源包应只含代码、说明和测试，不包含 Token、数据库、上传图片、生成图片、日志或缓存。
2. 本批变化位于 `references/harness-install.md`、`assets/codex-harness-app/app.json`、`assets/codex-harness-app/web/app.js`、`assets/codex-harness-app/backend/server.py`、`assets/codex-harness-app/backend/test_server.py`、`tests/test_harness_installer.py`、`tests/test_frontend_api.py`。一并保留本仓库的 `AGENTS.md` 和 `DEPLOY.md` 作为操作索引与说明。
3. 无前端构建步骤，但栏目清单、前端和后端必须配套更新。无需移除额外服务器文件，无数据库迁移，无新增依赖、环境变量或定时任务。Python 与 Node 测试环境要求沿用原项目，Node 仅用于本地前端脚本测试。

### 服务器操作：已有栏目升级

1. 在宝塔终端核对 `/opt/skilldeck/share/diy.md`、实际栏目目录及当前 HTTP/HTTPS 入口。默认栏目目录为 `/var/lib/skilldeck-custom/apps/amazon-image-generator`；无法确认目标时停止。
2. 等待正在生图或归档的任务结束，避免重启将进行中任务标记为失败。使用栏目自己的 `backend/stop.sh` 停止该后端，再将整个栏目目录备份到不被网站公开访问的服务器目录，限制备份访问权限；备份含密钥和用户数据，不要复制到 Skill 源包。停止或备份失败时不要继续。
3. 在刚上传的 Skill 源包目录，以 root 身份执行 `python3 scripts/install_harness_app.py`。安装器更新栏目清单、`web/` 和 `backend/`，保留 `data/`，另把旧代码保存在栏目 `.install-backups/` 下，并启动后端。不要删除或覆盖正式 `data/feiyushentu.toml`、SQLite、上传文件和生成图片。
4. 若原部署使用非默认 `HARNESS_ORIGIN` 或端口，先核对原值，按安装说明沿用；不要猜测。已有同源 API 代理应复用，无需为了本次权限调整修改宝塔主站配置、证书或端口。
5. 安装或启动失败时停止后续操作，不重复覆盖正式文件，按下方步骤回退。

### 服务器操作：首次安装

1. 确认目标目录没有其他应用或未知数据后，在上传的 Skill 源包目录以 root 身份运行 `python3 scripts/install_harness_app.py`。
2. 按 `references/harness-install.md` 核对同源 API 路由；缺少路由时仅在已验证、已被当前站点加载的专用 include 目录内配置模块代理。找不到此位置时停止，不修改主站配置来绕过检查。
3. 登录网站，在飞鱼神图页面配置一次共用 Token。无需把本机配置或其他网站的数据上传到服务器。

### 验收

- 检查栏目 `app.json` 的 `visibility` 为 `members`，原网站及模块公开健康地址 `/custom-api/amazon-image-generator/health` 正常；健康地址必须返回 JSON 且 `data.ok = true`，不能是 HTML。
- 分别使用管理员和普通成员登录并强制刷新：均能看见栏目，加载配置和共享历史，预览及下载已有图片；普通成员不再收到 `root_required`。
- 使用测试 Token 或计划中的 Token 更换验证普通成员可保存配置；不要为了验收随意替换正式共用 Token。需要真实生图验收时提交一项计划内任务，确认状态更新及结果归档，避免额外消耗额度。
- 退出登录后业务 API 和图片地址应返回 `401`；带会话但缺少或错误 CSRF 的写请求应返回 `403 csrf_invalid`。
- 已有数据升级后核对历史数量、已有图片和 Token 配置状态；不要打印 Token 值。

### 回退

1. 停止本栏目后端，将本次 `.install-backups/` 中的 `app.json`、`web/`、`backend/` 配套恢复；也可从操作前的受保护备份恢复这三个代码项。不要用旧数据覆盖更新后产生的 `data/`。
2. 保留原所有者和文件权限，使用原入口和端口配置运行 `backend/start.sh`，复查网站与健康地址。回退后栏目会恢复旧版管理员限制。
3. 本批未改数据库结构，无数据库回退步骤。首次安装需撤销时，按安装说明使用 `scripts/remove_harness_app.py` 可恢复地归档栏目；如新增了受管代理，仅移除该模块代理并验证站点，不删除保留的数据。
