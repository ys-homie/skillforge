---
name: "figma-parsing"
description: "Figma 设计稿解析：读取设计稿提取 UI 结构、样式、切图清单。需求拆解完成后使用。"
metadata:
  last_modified: "2026-06-01"

---
# Figma 设计稿解析

## 触发条件
需求拆解完成后，有 Figma 链接需要解析。

## Figma 获取方式

### 首选：Figma MCP / Plugins OAuth

- ChatGPT 账号登录 Codex 时，优先使用 Figma MCP / Plugins OAuth 获取设计稿。
- 必须先获取结构化设计信息，再获取截图；节点过大或返回被截断时，先获取元数据定位子节点，再重新获取精确节点。
- 实现过程中 UI 细节不确定时，继续通过 Figma MCP 回查，避免凭截图猜测。

### 兜底：Figma PAT + REST API

当 API Key 登录导致 Plugins / Figma OAuth 不可用，或 Figma MCP 受限时，使用 Figma Personal Access Token 兜底读取设计稿。

**Token 配置：**
- Token 必须存放在环境变量 `FIGMA_ACCESS_TOKEN` 中，不要写进聊天、代码、文档示例的真实值或 Git 仓库。
- 本地持久化建议写入 `~/.zshrc` 或 `~/.zshrc.local`，例如：`export FIGMA_ACCESS_TOKEN="<your_token>"`。
- 如果 Token 泄露，立即到 Figma Settings → Security → Personal access tokens 中 revoke。

**Figma URL 解析：**

```text
https://www.figma.com/design/<FILE_KEY>/...?node-id=<NODE_ID>
```

- `FILE_KEY`：`/design/` 后面的文件 key，例如 `PlLSwQiTA4fRvlTYC5tUQs`
- `NODE_ID`：`node-id=` 后面的节点 ID，例如 `17773:18537`

**REST API 请求：**

```bash
# 获取节点 JSON
curl -H "X-Figma-Token: $FIGMA_ACCESS_TOKEN" \
  "https://api.figma.com/v1/files/<FILE_KEY>/nodes?ids=<NODE_ID>"

# 获取节点截图 URL
curl -H "X-Figma-Token: $FIGMA_ACCESS_TOKEN" \
  "https://api.figma.com/v1/images/<FILE_KEY>?ids=<NODE_ID>&format=png&scale=2"
```

**Codex 执行规则：**
1. 从环境变量读取 `FIGMA_ACCESS_TOKEN`，禁止输出真实 Token。
2. 通过 `files/<FILE_KEY>/nodes?ids=<NODE_ID>` 获取节点 JSON。
3. 通过 `images/<FILE_KEY>?ids=<NODE_ID>&format=png&scale=2` 获取截图 URL；如需下载截图，仅保存到临时目录或任务相关资源目录。
4. 结合节点 JSON、截图、项目字体资源、项目颜色资源输出 UI 结构、字体、颜色、切图清单和交互说明。
5. 网络请求需要授权时，说明用途是「调用 Figma REST API 读取设计稿节点和截图」。

## 输出模板

```markdown
## UI 结构分析
- 页面/组件名：[名称]
- 设计稿链接：[Figma URL]
- 布局结构：[层级描述]
- 关键样式：[颜色/字号/间距/圆角等具体数值]
- 字体匹配：[从 Figma 获取的字体名 → 项目中对应的字体资源/工具类方法]
- 颜色匹配：[从 Figma 获取的色值 → 项目中对应的颜色常量]
- 切图资源清单：[需要哪些切图，命名建议，格式/倍率]
- 交互说明：[动画/手势/状态切换]
```

## 字体匹配规则

从 Figma 获取到字体信息后，**必须先到项目的字体管理类/资源中查找是否有可复用定义**，优先使用已有方法/常量，不要直接硬编码样式。

**通用流程：**
1. 从 Figma 提取字体名、字重、字号
2. 在项目字体管理文件中搜索匹配
3. 找到 → 使用已有定义；未找到 → 标注需新增

<!-- [项目特定] 在此列出项目的字体映射表 -->

## 颜色匹配规则

从 Figma 获取到颜色后，**必须先到项目的颜色管理类/资源中查找是否已有定义**：
- 已有 → 使用项目颜色常量
- 没有 → 标注需新增到颜色管理

<!-- [项目特定] 在此列出颜色管理文件路径和命名规范 -->

## 图片资源规则

| 类型 | 格式 | 倍率 |
|---|---|---|
| 位图（照片、复杂插画、渐变背景等） | **WebP**（推荐）或 PNG | 1x / 2x / 3x |
| 矢量小图（图标、简单装饰图形等） | **SVG** 或 PDF | 无需倍率 |

**判断依据：** 颜色少于 256 色且可无损缩放 → 矢量格式；否则 → 位图格式。

<!-- [项目特定] 补充资源目录结构和引用方式 -->
<!-- iOS: Assets.xcassets / SF Symbols -->
<!-- Android: res/drawable / VectorDrawable -->
<!-- Flutter: assets/images/ + assets/svgs/ -->

## Figma MCP 不可用时

1. 优先检查是否已配置 `FIGMA_ACCESS_TOKEN`，如已配置则走 PAT + REST API 兜底方案。
2. 如果没有 Token 或 REST API 也受限，告知负责人："Figma MCP 受限，且未检测到可用的 FIGMA_ACCESS_TOKEN，建议提供设计稿截图或先配置 Figma PAT"
3. 如提供截图，基于截图继续完成分析。

## 解析优先级

1. **整体布局方向** — 横向/纵向/层叠，识别主轴方向
2. **Auto Layout 参数** — padding、gap、alignment 映射到项目布局属性
3. **组件边界** — 识别可复用组件，确定抽离粒度
4. **交互状态** — default / hover / pressed / disabled / loading 各状态样式
5. **响应式行为** — 固定宽度 vs 填充父容器 vs 内容撑开
