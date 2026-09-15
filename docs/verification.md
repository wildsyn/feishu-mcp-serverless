# 验收记录（2026-09-15）

## 已通过

- Node 22 GitHub Actions：干净 `npm ci`、TypeScript 检查、OAuth 回归与构建。
- 新加坡 FC：自定义运行时、OSS 临时角色凭据、正式域名 HTTPS。
- OSS：加密读回、10 个并发单次消费请求中仅一个成功；测试对象已清理。
- 正式域名：飞书与 Multica 各自的健康、OAuth issuer、资源元数据、未授权 401 均正确区分。
- ChatGPT 网页：动态客户端注册、PKCE 授权、飞书登录、回调、工具列表加载、重新连接均成功。
- ChatGPT 实际调用：创建独立测试多维表格和数据表，配置文本/数字字段，写入一条合成记录，再更新并读取。
- 最终搜索记录与单条读取一致：交易编号 `MCP-TEST-001`、金额分 `2345`、备注 `更新验收`。另在飞书网页直接核对了相同值。
- 飞书返回权限错误时，MCP 保留错误标记与业务错误内容，没有将 HTTP 200 当成业务成功。

测试表保留供部署者检查，具体资源链接在私有部署记录中。未用真实账单进行验收。

## v0.2.0 工具与搜索更新

- 新加坡 FC 已部署，ChatGPT 刷新后实际加载 505 个不同工具，包括文档搜索和 Wiki 搜索。
- GitHub Actions 已通过类型检查、OAuth 回归和构建。
- 使用新授权直接调用云文档搜索成功，业务码 0，返回匹配项。ChatGPT 搜索对话的端到端结果仍在核验。

## 浏览器兼容与权限经验

- 原生表单在 `Referrer-Policy: no-referrer` 下可能携带 `Origin: null`。使用 `strict-origin` 保留来源校验，同时不发送路径与查询参数。
- CSP 的 `form-action` 会检查重定向链；飞书登录经过 open、accounts、passport、login 官方域名。
- 每个授权事务使用独立 Cookie，避免多个登录窗口互相覆盖。
- 云文档搜索需要 `drive:drive.search:readonly`，Wiki 搜索另需 `search:docs:read`；新增后需更新服务 OAuth scope 并重新连接。
- `offline_access` 必须在飞书应用用户权限中开通；仅在 OAuth 请求中填写不够。
- `base:record:read` 用于特定记录读取；搜索和列表需 `base:record:retrieve`。权限调整后需要重新连接。

## 验收范围

v0.1.0 的业务验收集中在多维表格，当时直接显示 10 个多维表格工具与 2 个通用入口。v0.2.0 将 503 个用户身份工具全部直接提供，加上 2 个通用入口共 505 个；本地已验证完整列表、命名唯一性、名称长度及搜索/写入标注。其他 API 的可调用性仍取决于对应授权和资源范围。二进制上传下载尚未适配。

本地回归覆盖错误 resource、错误 PKCE、重复兑换、刷新轮换、撤销、授权 Cookie 和到期状态；尚未做长时间压测及大规模账单导入测试。

## 运维事项

当前部署的 HTTPS 证书到期时间为 2026-12-14 06:03:53 UTC。续期后使用 `deploy/deploy.sh domain` 更新共享域名证书，并重新核对 HTTPS。尚未配置自动续期定时任务。

上游 OSS 依赖链仍有 npm audit 的 PAC 代理解析告警，当前固定 OSS endpoint 不使用 PAC。详见 README 当前边界。
