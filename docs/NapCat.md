# NapCat 支持说明

本 fork 在原作者 [Derpyu520/qq-bridge](https://github.com/Derpyu520/qq-bridge) 的基础上，
增加了对 **NapCat** 协议端的支持。原版仅实测 SnowLuma，本 fork 让两种协议端都能用。

## 原理

SnowLuma 和 NapCat 都遵循 **OneBot v11** 协议。原版依赖的 `@snowluma/sdk` 本质上是
纯 OneBot v11 客户端（`{action, params, echo}` 收发 + `post_type` 事件，全是标准协议），
连 NapCat 直接可用，无需替换 SDK。

真正 SnowLuma 特有的只有一处：**token 自愈**读取的配置文件路径。本 fork 只改了这一处，
让 token 自愈同时识别两种协议端的配置目录。

## 与 NapCat 配合使用

### 1. NapCat 侧配置

在 NapCat WebUI（默认 `http://127.0.0.1:6099/webui/`）：

1. 扫码登录 QQ
2. 网络配置 → 新建 **WebSocket 服务端**（正向 WS），端口 `3001`
3. 新建 **HTTP 服务端**，端口 `3000`
4. 按需设置 `token`（公网部署务必设置）

也可以直接编辑 NapCat 的 `config/onebot11_<QQ号>.json`：

```json5
{
  "network": {
    "httpServers": [
      {
        "name": "httpServer",
        "enable": true,
        "port": 3000,
        "host": "0.0.0.0",
        "messagePostFormat": "array",
        "token": "你的token"
      }
    ],
    "websocketServers": [
      {
        "name": "WsServer",
        "enable": true,
        "host": "0.0.0.0",
        "port": 3001,
        "messagePostFormat": "array",
        "token": "你的token",
        "heartInterval": 30000
      }
    ]
  }
}
```

### 2. qq-bridge 侧配置

`config.json` 的 `snowluma` 段对 NapCat 同样适用（字段名保留以兼容原版）：

```json
{
  "snowluma": {
    "wsUrl": "ws://127.0.0.1:3001",
    "httpUrl": "http://127.0.0.1:3000",
    "accessToken": "你在 NapCat 配置的 token（没配就留空）",
    "followDiscoveredEndpoint": true,
    "homeDir": "/你的/NapCat/安装目录"
  }
}
```

- `homeDir` 必须指向 NapCat 安装目录（含 `config/` 的那个目录），token 自愈靠它。
- 换 QQ 账号后 token/端口变化，桥接会自动重新发现（同原版 SnowLuma 行为）。

### 3. token 自愈自动识别

本 fork 的 `readOneBotTokens()` 同时扫描两种配置：

| 协议端 | 配置文件名 | 顶层键 | token 字段 |
|---|---|---|---|
| SnowLuma | `config/onebot_<QQ号>.json` | `networks`（复数） | `accessToken` |
| NapCat | `config/onebot11_<QQ号>.json` | `network`（单数） | `token` |

NapCat 的 server 若 `enable: false` 会被视为不存在（不参与自愈）。

### 4. 其余步骤与原版相同

DSH 端安装（`node scripts/setup-dsh.mjs` + 重启 DSH）、启动桥接（`npm start`）等
步骤与原版完全一致，见原版 README。

## 与原版的差异

| 项 | 原版 | 本 fork |
|---|---|---|
| 协议端 | 仅 SnowLuma | SnowLuma + NapCat |
| token 自愈 | 只读 `onebot_*.json` | 同时读 `onebot_*.json` 和 `onebot11_*.json` |
| SDK / MCP | `@snowluma/sdk` / `@snowluma/mcp`（纯 OneBot v11） | 不变（本就是标准协议） |
| 其余功能 | — | 完全一致 |

## 改动文件清单

- `src/snowluma-conn.js` — token 自愈支持 NapCat 配置目录
- `package.json` — 描述更新
- `docs/NapCat.md` — 本文档

其余源码（`src/bridge.js` 等）未改动，与原版一致。

## 许可

沿用原版 [PolyForm Noncommercial License 1.0.0](LICENSE)：禁止商业使用。
