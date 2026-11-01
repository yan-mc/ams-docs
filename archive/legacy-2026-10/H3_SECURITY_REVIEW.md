# H3 安全评审：Host loopback 无 token（2026-10-25）

## 观察

- 带 Bearer token：200；
- 不带 token、来自 127.0.0.1 的 same-origin 请求：200；
- 带跨站 Origin / Sec-Fetch-Site: cross-site：401；
- token 没有返回给浏览器页面，页面使用 session。

## 设计意图

Host 的 authKind 允许：

1. Gateway Bearer token；
2. 浏览器 session cookie；
3. 同源 loopback fallback（兼容 WebView cookie 丢失）。

这是桌面本机场景的本地信任设计，不是远程开放 API。

## 风险

- 本机任意进程可以访问 /ams/api/*；
- 恶意本地程序不受 token 保护；
- 但跨站浏览器请求已被拒绝。

## H3 决策

H3 接受现状，理由：

- 服务只监听 loopback；
- 不写 C 盘 AMS data；
- token 不出 Host；
- 外部跨站已 401；
- 业务 UI 依赖 WebView 同源 fallback。

## 后续

- H3 文档明确“本地同机信任”边界；
- H4 可评估强制 session/Bearer、去掉 fallback；
- 若去掉 fallback，必须重新验证 WebView cookie。
