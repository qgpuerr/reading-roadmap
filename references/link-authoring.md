# 在线章节链接生成（link-authoring）

仅用于"有在线章节"的目标。核心纪律：**链接来自真实抓取或源码推算，逐条验证，禁止编造**。

## 1. 摸结构（先于链接）

- 有仓库：GitHub API 拉目录树，取章节 md 路径（`GET /repos/{owner}/{repo}/git/trees/{branch}?recursive=1`）。
- 纯站点：抓 README/导航，重建章节清单；站点为 docsify 时章节即 docs/ 下 md 文件。
- 无目录：给章节地图草案，请用户确认后再定链接。

## 2. 识别站点技术栈（决定链接形态）

- **docsify**：特征 `docsify.min.js` + `#/./路径` hash 路由。链接形如
  `https://site/#/./chapterN/文件名?id=小节slug`（文件名中的空格用 `%20`，中文可直接写）。
- 其他 SPA（GitBook/Docusaurus/VitePress）：锚点规则不同，抓页面确认实际 `href`，不套 docsify 规则。
- GitHub 仓库文件：`https://github.com/{owner}/{repo}/blob/main/路径.md`。

## 3. docsify 小节锚点（已验证，docsify 4.13 / 5.0 一致）

slug 由标题按官方 slugify 生成：

```
trim → 大写转小写 → 删 HTML 标签 → 删标点
（标点集 = /[\u2000-\u206F\u2E00-\u2E7F\\'!"#$%&()*+,./:;<=>?@[\]^`{|}~]/g）
→ 空白(含中文空格)转 '-' → 合并连字符 → 数字开头加 '_'（如 _113-智能体的类型）
```

- 中文标点（全角 ！？。，）**不在**删除集内，会保留在 slug 中并需编码。
- 百分号编码大小写等价，统一用小写（如 `%e6%99%ba`）。
- 链接内带中文/全角符号时在 HTML href 中直接写原文，浏览器自动编码。

## 4. 验证（必做）

- 每个在线链接实际抓取一次：返回内容匹配目标章节/小节，非 404。
- 页面 404 或未挂载的内容（如仓库 extra 目录未被站点服务）→ 回退到 GitHub 原文链接，并如实说明。
- 批量验证用脚本（scripts/resolve_anchors.py），逐条人工抽验。

## 5. 已知坑

- 站点的 docsify 版本可能不是固定值（如 `@latest`），slugify 规则以**站点实际加载版本**为准：抓站点 index.html 看引入的 docsify 版本，必要时从 CDN 取该版本源码核对 slugify。
- 站点可能有自定义 slugify 或 markdown 配置：检查 `window.$docsify` 配置块。
- 中文标题的 slug 依赖站点版本，推算后必须实测一条再批量。

## 6. 回退链

在线链接不可用（站点下线/无在线版）→ 告知用户当前版本不支持该目标 → 降级为对话内文字路线（章节清单 + 页码/章节名定位），不假装链接可用。
