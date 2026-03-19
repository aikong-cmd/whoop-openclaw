# WHOOP 健康数据同步 🏋️

将 WHOOP 手环数据同步到 Markdown 文件，让 AI 帮你分析健康状况。

纯 Python 实现，零依赖（仅用 stdlib + curl），10 分钟搞定。

## 功能

| 类别 | 数据 |
|------|------|
| **恢复** | 恢复分数 (🔴🟡🟢)、HRV (RMSSD)、静息心率、血氧、皮肤温度 |
| **睡眠** | 表现/效率/一致性、各阶段时长（浅睡/深睡/REM/清醒）、呼吸频率、睡眠需求、睡眠盈亏 |
| **日间负荷** | 负荷值 (0-21)、消耗热量、平均/最高心率 |
| **运动** | 运动类型、时长、负荷、心率、心率区间、距离、海拔 |
| **周报** | 恢复/HRV/心率/睡眠/负荷的周均值 |

## 安装

### 方式一：OpenClaw 一键安装

```bash
clawhub install whoop
```

### 方式二：手动

```bash
git clone https://github.com/aikong-cmd/whoop-openclaw.git
cp -r whoop-openclaw ~/.openclaw/workspace/skills/whoop
```

## 设置

### 第 1 步：创建 WHOOP 开发者应用

1. 打开 [developer-dashboard.whoop.com](https://developer-dashboard.whoop.com/)
2. 用 WHOOP 账号登录
3. 点 **Create Application**
   - **Name**: 随便填，比如 "AI Health Sync"
   - **Redirect URI**: `http://localhost:9527/callback`（必须完全一致）
   - **Scopes**: 勾选所有 `read:*` + `offline`
4. 记下 **Client ID** 和 **Client Secret**

### 第 2 步：配置凭证

**环境变量（最简单）：**

```bash
export WHOOP_CLIENT_ID="你的-client-id"
export WHOOP_CLIENT_SECRET="你的-client-secret"
```

**或 1Password（OpenClaw 用户推荐）：**

在 1Password 创建登录项，名称 `whoop`，用户名填 Client ID，密码填 Client Secret。

### 第 3 步：授权（只需一次）

#### 🖥️ 本地电脑（浏览器在同一台机器）

```bash
python3 scripts/auth.py
```

打开链接 → 登录 WHOOP → 授权 → 自动完成 ✅

#### 🌐 远程服务器（无头模式）

```bash
# 获取授权链接
python3 scripts/auth.py --print-url

# 在本地浏览器打开链接并授权
# 浏览器会跳转到一个打不开的页面（正常！）
# 复制地址栏完整 URL，然后：

python3 scripts/auth.py --callback-url "http://localhost:9527/callback?code=xxx&state=yyy"
```

**授权一次，永久有效。** Token 通过 `offline` scope 自动续期。

## 使用

```bash
# 同步今天
python3 scripts/sync.py

# 同步最近 7 天
python3 scripts/sync.py --days 7

# 周报
python3 scripts/sync.py --weekly

# 指定日期
python3 scripts/sync.py --date 2026-03-07
```

输出文件：`~/.openclaw/workspace/health/whoop-YYYY-MM-DD.md`

## 每日自动推送（配合 OpenClaw cron）

```bash
openclaw cron add \
  --name whoop-daily \
  --schedule "0 10 * * *" \
  --timezone Asia/Shanghai \
  --task "Run: python3 ~/.openclaw/workspace/skills/whoop/scripts/sync.py --days 2. Then read the generated markdown files and send me the latest day's report."
```

每天 10:30 自动同步 → 生成报告 → 推送到飞书/Telegram。

> 为什么是上午？因为 WHOOP 的睡眠数据需要等起床后才能最终确认。

## 输出示例

```markdown
# WHOOP — 2026-03-09

## Recovery 🟢
- **Recovery Score**: 66%
- **HRV (RMSSD)**: 41.4 ms
- **Resting HR**: 62.0 bpm
- **SpO2**: 96.3%
- **Skin Temp**: 33.7°C

## Sleep
- **Performance**: 🟡 61%
- **Total in Bed**: 5h47m
- **Stages**: Light 2h08m | Deep 1h25m | REM 1h38m | Awake 35m
- **Sleep Need**: 9h41m
- **Balance**: 4h29m deficit

## Day Strain
- **Strain**: 0.1 / 21.0
- **Calories**: 2233 kJ (534 kcal)

## Workouts
- Walking · 16m28s · Strain 4.9 · Avg HR 114 bpm
```

## 常见问题

| 问题 | 解决 |
|------|------|
| `No tokens found` | 先跑 `auth.py` |
| `Token refresh failed (403)` | 重新跑 `auth.py` 授权 |
| `error code: 1010` | Cloudflare 拦截，脚本已用 curl 绕过。检查网络 |
| `No data for date` | WHOOP 睡眠数据要等起床后才有，稍后再试 |
| 端口 9527 被占用 | `kill $(lsof -ti:9527)` 后重试 |

## 文件结构

```
whoop/
├── SKILL.md              # AI agent 指令
├── README.md             # 英文文档
├── README_CN.md          # 中文文档（本文件）
├── scripts/
│   ├── auth.py           # OAuth 授权
│   └── sync.py           # 数据同步 + 周报
└── data/
    ├── tokens.json       # OAuth token（自动续期，已 gitignore）
    └── .gitignore
```

## 系统要求

- Python 3.10+
- curl
- WHOOP 会员账号

## License

MIT
