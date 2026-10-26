# QQ 机器人一键部署 Skill

> 给 AI Agent 用的一份部署 skill：让它从零把 QQ 机器人搭起来，**人只需要回答 4 句话**。

这是一份写给 Agent 的操作手册，不是写给人手工照做的教程。装上它之后，你只要说一句"帮我部署 QQ 机器人"，剩下的下载、配置、注入、接线、验证都由 Agent 完成。

## 人类只剩 4 个动作

| # | 动作 | 能不能省 |
|---|---|---|
| 1 | 说明网络状况（直连 / 热点 / 加速器） | 不能省，但 Agent 会先自检，你只需点头 |
| 2 | 贴一段模型 API key | 不能省，一条字符串 |
| 3 | 把机器人账号登录到桌面 QQ | 已登录则**零操作** |
| 4 | 要不要装配套插件 | 一句话 |

## 它覆盖什么

```
QQ 客户端 → SnowLuma（注入）→ OneBot v11 反向 WS → AstrBot → 模型 API
```

- **协议端**：SnowLuma 注入桌面 QQ，绕开 supervisor、提权到 Medium
- **框架**：AstrBot 装依赖、写配置、加 OneBot v11 适配器、接 DeepSeek 等模型
- **面板**：自动打开 WebUI 并免密登录（CDP 注入 token）、自动改初始密码
- **顺手优化**：配好 GitHub 加速（插件下载不卡）
- **配套插件**：可选安装 [本少的qq机器人功能包](https://github.com/DanDingBenShao/astrbot_plugin_qq_kit)
- **排障**：双栈冲突（NapCat + SnowLuma 双回）、端口撞车、注入失败

## 内置 20 个实测坑

最值钱的部分在这里。每个坑都给了「症状 → 根因 → 修法」，例如：

| 症状 | 真实根因 |
|---|---|
| 注入失败，QQ 列表为空 | 解压出来的 exe 被打了 **Low 完整性标签**，需要 `icacls /setintegritylevel Medium` |
| `spawn EPERM` | 环境禁 IPC，SnowLuma 的 supervisor 起不来 |
| `PingException 拒绝访问` | Lagrange 硬调 ICMP，该 build 的参数关不掉 |
| 配置读不出来 | AstrBot 的 `cmd_config.json` 带 **UTF-8 BOM** |
| provider 测试报 `unknown` | 模型名必须放顶层 `model`，不是 `model_config.model` |
| 从链接安装报"压缩包格式错误" | 实际是下载回来的不是 zip，不是仓库的问题 |
| 消息被回两次 | 两套栈（NapCat + SnowLuma）同时消费同一个账号 |

## 安装

把 `SKILL.md` 放到 skills 目录，**目录名必须与 frontmatter 里的 `name` 一致**：

```bash
# 通用（Claude Code / DSH 等从 ~/.claude/skills 加载的实现）
mkdir -p ~/.claude/skills/qq机器人一键部署skill
cp SKILL.md ~/.claude/skills/qq机器人一键部署skill/
```

Windows：

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\qq机器人一键部署skill"
Copy-Item SKILL.md "$env:USERPROFILE\.claude\skills\qq机器人一键部署skill\"
```

装完**需要新开一个会话**才会被扫描到（skill 目录是会话启动时加载的）。

## 触发方式

显式：`用 qq机器人一键部署skill`

或者自然语言命中 description 里的触发词：部署 QQ 机器人 / 接个小号 / AstrBot / SnowLuma / NapCat / 机器人不回话 / 注入失败 / 消息回两次。

## 前置条件

- Windows 10/11
- 桌面版 QQ（NTQQ）已安装并登录
- 一个模型 API key（DeepSeek、硅基流动、OpenAI 兼容的任意一家）
- Python 3.12+ 与 [uv](https://docs.astral.sh/uv/)（Agent 会自己装）

## 免责

- 第三方 QQ 协议实现**有封号风险**，请用非主号，控制发送频率。
- SnowLuma 是**源码可见非商业许可**，商用需自行取得授权；AstrBot 是 AGPL-3.0。
- 本文档按"现状"提供，实测环境见文末，不保证在其他环境同样适用。

## 许可证

[MIT](LICENSE)
