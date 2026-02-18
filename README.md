# OmniBox API Skill

Cursor Agent Skill，用于通过 [OmniBox](https://api.omnibox.pro) API 管理知识库资源。

## 功能

- **收藏网页** — 通过 URL 一键保存网页内容到 OmniBox
- **创建资源** — 直接写入文档/笔记，支持 `#hashtag` 自动打标签
- **上传文件** — 将本地文件（PDF、文档等）作为资源上传
- **AI 提问** — 向 OmniBox Wizard 提问，基于个人知识库回答
- **API Key 管理** — 查看或删除 API Key

## 文件

| 文件 | 说明 |
|------|------|
| `SKILL.md` | Skill 主文件，包含所有端点的使用示例 |
| `reference.md` | 完整的请求/响应 Schema 参考 |

## 配置

需要设置环境变量：

```bash
export OMNIBOX_API_KEY="your-api-key-here"
```

## 触发场景

在对话中提及以下关键词时，Agent 会自动加载此 Skill：

- OmniBox / omnibox
- 收藏网页、保存链接
- 上传文档到知识库
- 向 OmniBox 提问

## API 文档

官方文档：https://api.omnibox.pro/docs
