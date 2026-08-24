# AiBean Agent Skills

此仓库收集可复用的 AiBean 智能体技能。当前包含 `aibean-business-development`，用于在用户指定的本地 Vue 与 Java Biz API 工作区开发业务功能。

## 安装

克隆仓库后，将具体技能目录作为技能根目录的直接子目录链接或复制。不要把整个业务前端或后端仓库放入技能目录。

```sh
git clone https://github.com/zn3051644/aibean-agent.git
ln -s "$(pwd)/aibean-agent/skills/aibean-business-development" "$HOME/.codex/skills/aibean-business-development"
ln -s "$(pwd)/aibean-agent/skills/aibean-business-development" "$HOME/.agents/skills/aibean-business-development"
```

Codex 使用配置的技能目录（通常为 `$CODEX_HOME/skills` 或 `~/.codex/skills`）；deepseek-harness 的本地发现器会扫描 `~/.agents/skills`。两者均读取同一份 `SKILL.md`，`agents/openai.yaml` 仅提供可选的 Codex 界面元数据。详见 [设计文档](docs/aibean-business-development-design.md)。
