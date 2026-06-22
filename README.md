# 2SOMEone 游戏资讯机器人

2SOMEone 官方游戏资讯机器人：定时拉取游戏媒体 RSS，把新鲜事发布成[泡泡动态](https://2some.ren)。

它从[资讯机器人模板](https://github.com/leaperone/2someone-news-bot)派生而来，只换了 `feeds.json` 的源（机核 / 游研社 / 触乐）。代码逻辑完全一致——只用公开 v1 API + bot runtime key，不享受任何内部特权。

## 工作原理

```
GitHub Actions（每小时） → 拉取 feeds.json 里的游戏媒体 RSS 源
  → 过滤最近 24 小时的新条目
  → 读取机器人自己发过的泡泡去重（无状态，不需要数据库）
  → POST /api/v1/bubbles 发布（单次最多 3 条，防刷屏）
```

核心只有一个文件 [`index.mjs`](./index.mjs)，唯一依赖是 `rss-parser`。

## 部署三步

### 1. 创建游戏机器人账号

安装 2SOMEone CLI 并登录，然后：

```bash
2s1 bot create --username game_news --nickname "游戏资讯姬" \
  --capability bubble.create --capability bubble.read

# 获取 runtime key（只显示一次，立即保存）
2s1 bot key rotate <botUserId> --yes
```

> 普通账号每人最多 3 个机器人；`bubble.create` + `bubble.read` 两个能力都需要（read 用于去重）。

### 2. 配置 Secret

**Settings → Secrets and variables → Actions → New repository secret**

| Name | Value |
|---|---|
| `TWOSOMEONE_BOT_API_KEY` | 上一步拿到的 `sk_...` runtime key |

### 3. 启用 Actions 并测试

去 **Actions** 标签页启用，然后手动跑一次 **Post news** workflow（勾选 `dry_run` 先看效果，不会真实发布）。

## 自定义

### 换 RSS 源

编辑 [`feeds.json`](./feeds.json)：

```json
[
  { "name": "来源显示名", "url": "https://example.com/feed.xml" }
]
```

任何标准 RSS / Atom 源都可以。没有官方 RSS 的站点可以用 [RSSHub](https://docs.rsshub.app/) 生成（建议自建实例，公共实例可能不稳定）。

### 调参数

通过环境变量（可在 workflow 的 `env:` 里加）：

| 变量 | 默认 | 说明 |
|---|---|---|
| `MAX_POSTS_PER_RUN` | `3` | 单次运行最多发布条数 |
| `FRESH_WINDOW_HOURS` | `24` | 只发布最近 N 小时内的新闻 |
| `TWOSOMEONE_BASE_URL` | `https://2some.ren` | 平台地址 |
| `DRY_RUN` | - | 设为 `1` 只打印不发布 |

### 本地运行

```bash
pnpm install
TWOSOMEONE_BOT_API_KEY=sk_xxx pnpm run dry-run   # 预览
TWOSOMEONE_BOT_API_KEY=sk_xxx pnpm start          # 真实发布
```

平台 API 速查与机器人能力见 [`SKILL.md`](./SKILL.md)。

## License

MIT
