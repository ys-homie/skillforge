# Codex + Figma：API Key 模式下读取设计稿

> 当你使用 API Key 登录 Codex 时，左侧 Plugins 不可点击，无法通过 OAuth 安装 Figma 插件。  
> 本文介绍如何通过 **Figma Personal Access Token (PAT)** 让 Codex 直接调用 Figma REST API 读取设计稿。

## 背景

| 登录方式 | Plugins / Figma OAuth | Figma API (PAT) |
|---|---|---|
| **ChatGPT 账号登录** | ✅ 可用 | ✅ 可用 |
| **API Key 登录** | ❌ 不可用（按钮灰掉） | ✅ 可用 |

API Key 登录只提供模型调用和代码代理能力，不具备应用侧的插件登录/OAuth 授权能力。  
因此需要通过 Figma PAT 作为替代方案。

## 操作步骤

### 1. 生成 Figma Personal Access Token

1. 打开 [Figma](https://www.figma.com) 文件浏览器首页
2. 点击左上角头像 → **Settings**
3. 切到 **Security** 标签
4. 找到 **Personal access tokens** 区域
5. 点击 **Generate new token**
6. 名称填写有意义的标识（如 `codex-figma`）
7. Scope 至少选择 **File content: Read**（`files:read`）
8. 生成后**立即复制** — Token 只显示一次

### 2. 持久化存储 Token

将 Token 写入 shell 配置文件，永久生效：

```bash
# 写入 ~/.zshrc（zsh）或 ~/.bashrc（bash）
echo 'export FIGMA_ACCESS_TOKEN="<your_token>"' >> ~/.zshrc
source ~/.zshrc
```

验证：

```bash
echo $FIGMA_ACCESS_TOKEN
```

> ⚠️ 确保 `~/.zshrc` 不会被提交到任何 Git 仓库。如果使用 dotfiles 管理，将该行加入 `.gitignore` 或单独放在 `~/.zshrc.local` 再 source。

### 3. 让 Codex 使用 Token 读取设计稿

告诉 Codex：

> Figma Token 已配置在环境变量 `FIGMA_ACCESS_TOKEN` 中，请读取后调用 Figma API 获取以下节点：
> - File key: `<your_file_key>`
> - Node ID: `<your_node_id>`

Codex 会：
1. 从环境变量读取 Token
2. 通过 Figma REST API 请求节点数据（需联网权限授权）
3. 返回节点的 JSON 结构 + 截图

### 4. Figma URL 中的 File Key 和 Node ID

```
https://www.figma.com/design/<FILE_KEY>/...?node-id=<NODE_ID>

示例：
https://www.figma.com/design/PlLSwQiTA4fRvlTYC5tUQs/...?node-id=17773:18537
                              ^^^^^^^^^^^^^^^^^^^^^^^^           ^^^^^^^^^^^
                              File Key                           Node ID
```

## Figma REST API 参考

Codex 实际调用的 API：

```bash
# 获取节点数据
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/files/<FILE_KEY>/nodes?ids=<NODE_ID>"

# 获取节点截图
curl -H "X-Figma-Token: $TOKEN" \
  "https://api.figma.com/v1/images/<FILE_KEY>?ids=<NODE_ID>&format=png&scale=2"
```

如果需要在代码中调用（Node.js 示例）：

```javascript
const axios = require('axios');

async function getFigmaNode(fileKey, nodeId) {
  const token = process.env.FIGMA_ACCESS_TOKEN;
  
  const response = await axios.get(
    `https://api.figma.com/v1/files/${fileKey}/nodes?ids=${nodeId}`,
    { headers: { 'X-Figma-Token': token } }
  );
  
  return response.data;
}
```

## 安全注意事项

- ⚠️ **永远不要**把 Token 发到聊天/代码仓库中
- ✅ Token 存在 `~/.zshrc` 环境变量中，不进入版本控制
- ✅ 如果 Token 泄露，立即到 Figma Settings 中 revoke
- ✅ 建议定期轮换 Token（如每季度更新）

## 两种 Figma 接入方式对比

| 方式 | 适用场景 | 优点 | 缺点 |
|---|---|---|---|
| **Plugins OAuth** | ChatGPT 账号登录 Codex | 一次授权，持续可用 | API Key 模式不可用 |
| **PAT + REST API** | API Key 登录 Codex | 任何登录方式都可用，持久化配置一次即可 | 需手动生成 Token |

> **推荐：** 如果能用 ChatGPT 账号登录，优先使用 Plugins OAuth 方式。  
> API Key 模式下才需要 PAT 方案。
