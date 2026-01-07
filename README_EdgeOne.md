# EdgeOne 部署说明

本项目已针对腾讯云 EdgeOne (Edge Functions) 进行了适配修改。

## 修改内容
1. **_worker.js**:
   - 移除了 Cloudflare 特有的 `cloudflare:sockets` 导入。
   - 增加了 `connect` 方法的全局兼容处理（EdgeOne 环境下通常需要确认 Runtime 支持 TCP Socket）。
   - 增加了 `request.cf` 的 Polyfill，将 EdgeOne 的请求头（如 `eo-region`, `eo-client-asn` 等）映射为 `request.cf` 对象，确保脚本中的地理位置逻辑正常工作。

## 部署方式 2: 使用 EdgeOne CLI (推荐 - 部署整个项目)

你可以直接将整个项目部署到 EdgeOne Pages，无需手动复制粘贴代码。

1.  **安装 EdgeOne CLI**:
    ```bash
    npm install -g edgeone
    ```

2.  **登录账号**:
    ```bash
    edgeone login
    ```
    (按提示选择 "Global" 或 "China" 区域并完成登录)

3.  **初始化/关联项目**:
    如果你是第一次部署，可能需要初始化或在控制台创建一个 Pages 项目。

4. **执行部署**:

    在项目根目录下运行：

    ```bash

    edgeone pages deploy

    ```

    CLI 会自动识别 `edgeone.json` 配置。

    **注意**: 我们已配置了 `edge-functions/[[path]].js` 和 `edge-functions/index.js` 来确保所有路径（包括 `/admin` 和 `/`）都能正确路由到处理函数。



5.  **配置环境变量**:
    部署完成后，别忘了在 EdgeOne 控制台的 Pages 项目设置中配置 `UUID`, `PROXYIP` 等环境变量，并绑定 `KV` 存储。

---

## 部署方式 1: 手动复制 (旧方法)

1. **新建 Edge Function**:
   - 在 EdgeOne 控制台创建一个新的 Edge Function。

   - 将修改后的 `_worker.js` 内容复制到代码编辑器中。

2. **环境配置 (Environment Variables)**:
   请在 EdgeOne 控制台设置以下环境变量（参考原 `wrangler.toml` 或原项目配置）：
   - `UUID`: 你的 UUID。
   - `HOST`: 你的域名（可选）。
   - `PROXYIP`: 代理 IP（可选）。
   - 其他原项目需要的变量。

3. **KV 存储配置**:
   - 原项目使用了 `env.KV`。
   - 在 EdgeOne 中，你需要创建一个 EdgeKV 命名空间，并在函数配置中将其绑定为 **`KV`** (绑定名称必须为 `KV`，以便代码中的 `env.KV` 能正确识别)。

4. **TCP Socket 与 WebSocketPair 支持**:
   - **重要限制**: 腾讯云 EdgeOne 的 Edge Functions 目前主要基于标准 Fetch API，可能不支持 Cloudflare 特有的 `connect` (TCP Socket) 和 `WebSocketPair` API。
   - **影响**: 如果缺少这些 API，此脚本的 WebSocket 代理功能将无法正常运行。
   - **替代方案**: 建议在 EdgeOne 环境中使用 **ProxyIP (HTTP 反代)** 模式，通过一个已有的 HTTP 代理服务器来转发流量，这通常只需要标准的 `fetch` API 即可运行。

5. **注意**:
   - `wrangler.toml` 是 Cloudflare 的配置文件，在 EdgeOne 中不生效。
   - `getCloudflareUsage` 函数是查询 Cloudflare 用量的，在 EdgeOne 环境下建议忽略。
   - 建议在部署后通过访问 `/admin` 页面查看配置和连接状态。

## 兼容性提示
代码中使用了 `globalThis.connect` 来尝试获取全局的 TCP 连接方法。如果 EdgeOne 提供的 API 名称不同（例如在 `net` 模块下），可能需要进一步调整代码头部。
