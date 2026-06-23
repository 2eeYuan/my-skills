# my-skills

> ⚡ Fast `/my-skills` catalog plugin for Claude Code — scan all installed skills, plugins & commands in seconds, not minutes

[English](README.md) | [中文](README.md#中文文档)

---

## What it does

Run `/my-skills` in Claude Code to instantly see a complete catalog of:

- 🧩 **All installed skills** — from every marketplace, with descriptions
- ⚙️ **All plugin commands** — every slash command available
- 🔌 **All background plugins** — LSP servers, output styles, security tools, etc.
- 📝 **All custom commands** — user-global and project-level

## Performance

| Metric | Before | After |
|--------|--------|-------|
| Scan time | ~4 min | **~30 sec** |
| Tool calls | ~15 sequential | **1 bash call** |
| Items discovered | 106 | **129+** |

The original approach made 15+ sequential file-read tool calls. This plugin runs a single bash script that scans everything in one pass — same data, 8× faster.

## Installation

### Option 1: Add as a marketplace (recommended)

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "my-skills": {
      "source": {
        "source": "git",
        "url": "https://github.com/2eeYuan/my-skills.git",
        "ref": "main"
      }
    }
  },
  "enabledPlugins": {
    "my-skills@my-skills": true
  }
}
```

Then restart Claude Code and run `/my-skills`.

### Option 2: Copy as a custom command

Copy `skills/my-skills/SKILL.md` to `~/.claude/commands/my-skills.md`.

## Supported marketplace structures

The plugin automatically detects all common marketplace layouts:

| Structure | Example |
|-----------|---------|
| `marketplace/skills/skill-name/` | anthropics-skills |
| `marketplace/plugin-name/skills/skill-name/` | pm-skills |
| `marketplace/plugins/plugin-name/skills/skill-name/` | claude-plugins-official |

## How it works

```
1. Scan marketplace directories for SKILL.md files
2. Scan plugin directories for commands/*.md files
3. Fall back to marketplace.json for plugins without plugin.json
4. Scan user-global (~/.claude/commands/) and project-level commands
5. Output structured data → Claude formats into readable tables
```

## License

[MIT](LICENSE)

---

# 中文文档

> ⚡ 为 Claude Code 打造的快速 `/my-skills` 目录插件 —— 秒级扫描所有已安装的 skills、插件和命令

## 功能介绍

在 Claude Code 中运行 `/my-skills`，即刻查看完整目录：

- 🧩 **所有已安装的 skills** —— 来自每个 marketplace，附带描述
- ⚙️ **所有插件命令** —— 每个可用的斜杠命令
- 🔌 **所有后台插件** —— LSP 服务器、输出样式、安全工具等
- 📝 **所有自定义命令** —— 用户级和项目级

## 性能对比

| 指标 | 优化前 | 优化后 |
|------|--------|--------|
| 扫描时间 | ~4 分钟 | **~30 秒** |
| 工具调用 | ~15 次串行 | **1 次 bash** |
| 发现项目数 | 106 | **129+** |

原方案需要 15+ 次串行文件读取调用。本插件通过单个 bash 脚本一次性扫描所有内容 —— 相同数据，快 8 倍。

## 安装方式

### 方式一：添加为 marketplace（推荐）

在 Claude Code 设置文件（`~/.claude/settings.json`）中添加：

```json
{
  "extraKnownMarketplaces": {
    "my-skills": {
      "source": {
        "source": "git",
        "url": "https://github.com/2eeYuan/my-skills.git",
        "ref": "main"
      }
    }
  },
  "enabledPlugins": {
    "my-skills@my-skills": true
  }
}
```

然后重启 Claude Code，运行 `/my-skills` 即可。

### 方式二：复制为自定义命令

将 `skills/my-skills/SKILL.md` 复制到 `~/.claude/commands/my-skills.md`。

## 支持的 marketplace 结构

插件自动检测所有常见的 marketplace 布局：

| 结构 | 示例 |
|------|------|
| `marketplace/skills/skill-name/` | anthropics-skills |
| `marketplace/plugin-name/skills/skill-name/` | pm-skills |
| `marketplace/plugins/plugin-name/skills/skill-name/` | claude-plugins-official |

## 工作原理

```
1. 扫描 marketplace 目录中的 SKILL.md 文件
2. 扫描插件目录中的 commands/*.md 文件
3. 对于没有 plugin.json 的插件，回退到 marketplace.json 获取描述
4. 扫描用户级（~/.claude/commands/）和项目级自定义命令
5. 输出结构化数据 → Claude 格式化为可读表格
```

## 许可证

[MIT](LICENSE)
