# MCP FC 可观测性验收 · 2026-09-16

## 生产结果

| 项目 | 飞书 | Multica |
| --- | --- | --- |
| 地域 | ap-southeast-1 | ap-southeast-1 |
| 当前生产版本 | prod → 2 | prod → 2 |
| 回滚基线 | 版本 1 | 版本 1 |
| 代码 revision | 12a9aec3ec3d2f16b58b10fc1fa8baf9f5830a3f | 79914594e3a4fe08a3f28a63c4f7177bf69ff957 |
| SLS | wildflow-mcp-sg/feishu-mcp | wildflow-mcp-sg/multica-mcp |
| 请求/实例指标 | 开启并观察到入库 | 开启并观察到入库 |
| 字段分析索引 | 20 个字段 | 20 个字段 |
| 保留期与分片 | 7 天、1 shard | 7 天、1 shard |
| 进程健康检查 | 新增 /feishu/healthz | 保留 /multica/healthz |
| 云监控告警 | 6 条，已启用 | 6 条，已启用 |

共享域名 8 条路由及两个 HTTP 触发器都已回读确认指向 prod。保留原有 OAuth 路径、TLS 配置和基础资源参数。版本 1 是修改前的代码和函数配置；私有配置另存于本机受限目录，未提交。

## 验证证据

- 飞书 `npm run verify` 通过：TypeScript 检查、13 项测试、打包；新增并发关联 ID 与敏感内容不入日志测试。
- Multica `go test ./...`、`go vet ./...`、`go build -o bin/multica-mcp .` 通过。新增 HTTP 流式响应兼容、请求关联、工具成功/失败/协议错误及日志脱敏测试。
- 版本切换脚本用模拟 FC 响应检查只修改目标函数路由、保留另一函数路由、不提交 TLS 字段；真实发布后再次回读确认。
- 飞书通过现有连接完成工具搜索和测试多维表格结构读取；生产别名切换后再调用成功。SLS 查询得到工具名、成功状态和耗时。
- Multica 线上 smoke 完成健康、未授权拒绝、DCR、PKCE 授权、授权码重复兑换拒绝、初始化、395 工具列表、4 类真实只读调用、刷新、撤销和撤销后拒绝。所有步骤通过。
- Multica 现有连接完成生成式 API 项目列表读取。用不存在的项目 ID 做只读错误测试，返回上游 404 与 MCP isError=true，SLS 对应工具记录 outcome=error；成功和失败记录都有 request_id。版本切换后再次真实读取成功。
- 两个日志库均观察到应用日志、FCRequestMetrics 和 FCInstanceMetrics，字段 SQL 查询实际成功；不是只检查开关。
- 12 条云监控规则回读确认 EnableState=true、60 秒周期、NoDataPolicy=OK、正确函数维度与阈值；使用既有“云账号报警联系人”。联系人的邮箱和短信状态均为 OK，未新增联系人。

## 限制与费用

本次未制造生产崩溃或超时来触发告警，也没有验证通知最终投递。HTTP 200 内的 MCP 业务失败已可统计，但未另建 SLS 业务失败通知规则。进程健康检查不代表飞书、Multica 或 OSS 全链路可用。

SLS 日志写入、索引和存储会增加按量费用；实际增量需以后续账单核对，本次不虚构金额。没有启用预留实例、链路追踪或调整 CPU、内存和并发。未重复验证业务写入，也未更改工具目录或要求用户重新授权。

运维命令、SQL、告警阈值及回滚方法见 [部署文档](deployment.md)。
