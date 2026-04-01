# 飞书/Lark Channel 插件 - Claude Code

将[飞书](https://www.feishu.cn/)（或 [Lark](https://www.larksuite.com/)）接入 Claude Code 的消息桥接插件，内置访问控制。

## 前置条件

1. 一个飞书自建应用，需配置：
   - **事件与回调** → 订阅方式选择 **使用长连接接收事件**
   - **已订阅事件**：`im.message.receive_v1`（接收消息）
   - **权限**：「读取用户发给机器人的单聊消息」、「接收群聊中@机器人消息事件」
2. 安装 [Bun](https://bun.sh/) 运行时
3. Claude Code CLI

## 安装

在 Claude Code 中执行：

```bash
# 1. 添加 marketplace
/plugin marketplace add RoacherM/feishu-channel-plugin

# 2. 安装插件
/plugin install feishu@feishu-plugin
```

## 配置

在 Claude Code 中运行：

```
/feishu:configure <APP_ID> <APP_SECRET>
```

凭证会保存到 `~/.claude/channels/feishu/.env`。

## 使用

启动带飞书 channel 的 Claude Code：

```bash
claude --channels plugin:feishu@feishu-plugin --dangerously-load-development-channels
```

然后在飞书上给 bot 发私信，消息会转发到 Claude Code 会话中。

## 访问控制

管理谁可以访问你的 bot：

```
/feishu:access                    # 查看当前策略
/feishu:access pair <code>        # 批准配对请求
/feishu:access allow <open_id>    # 添加用户到白名单
/feishu:access remove <open_id>   # 移除用户
/feishu:access policy allowlist   # 锁定为仅白名单模式（禁止新配对）
/feishu:access policy pairing     # 允许新用户配对（默认）
```

## 功能特性

- **私聊（P2P）**：发给 bot 的私信会转发到 Claude
- **群聊**：将 bot 加入群组，@机器人 时响应
- **访问控制**：新用户配对流程 + 白名单机制
- **图片支持**：发送的图片会下载并转发
- **文件附件**：文档等文件自动处理
- **权限中继**：工具权限请求转发到飞书供用户审批
- **表情回应**：收到消息后自动添加确认表情

## 项目结构

```
.claude-plugin/
  marketplace.json        # Marketplace 定义
feishu/                   # 插件源码
  .claude-plugin/
    plugin.json           # 插件元数据
  .mcp.json               # MCP 服务器配置
  server.ts               # 主服务
  skills/
    access/SKILL.md       # /feishu:access 技能
    configure/SKILL.md    # /feishu:configure 技能
```

## 许可证

MIT
