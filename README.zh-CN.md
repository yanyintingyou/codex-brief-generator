# Codex Brief Generator

简体中文 | [English](README.md)

这是一个兼容 Hermes / Agent Skills 规范的 skill，用来让 Hermes 生成可以直接复制给 OpenAI Codex CLI 的自包含任务提示词。

它服务于一种很实用的双 Agent 工作流：Hermes 负责研究总控、任务拆解和提示词规划；Codex CLI 负责在本地执行，修改文件、运行命令、验证结果，并把结果汇报回来。

## 动因

长时间运行的本地代码任务、数据处理任务，往往不适合完全塞进一次远程 Agent 对话里。实际使用中经常会遇到三个问题：

1. **上下文交接很脆弱。** Codex 不一定能看到 Hermes 前面的完整对话，所以“按刚才说的做”这类提示很危险。
2. **本地执行高度依赖环境。** Windows、WSL、Linux、macOS 的路径、shell、包管理器、本机应用都不一样。
3. **Agent 任务必须可验证。** 好提示词不只是告诉 Codex “做什么”，还应该明确“不要动什么”“怎么验证”“最后如何汇报证据”。

`codex-brief-generator` 的目的，就是让 Hermes 生成一份完整、有边界、可执行、可验证的 Codex 任务书。任务书会包含研究背景、目标、工作目录、输入输出、执行范围、约束条件、验证命令和最终报告格式。

这个 skill 的直接动因，是一种常见工作流：Hermes 作为持续在线的研究与规划助手运行，而 Codex 则由用户在本地 Windows 或 WSL 环境中手动运行，用来处理长时间任务、本地仓库修改、计量经济学/数据 pipeline，或者需要调用本机应用的工作。

## 它会生成什么

这个 skill 会指导 Hermes 生成以下结构的提示词：

- 运行模式
- 背景信息
- 任务目标
- 工作目录与运行环境
- 输入材料
- 预期输出
- 工作范围
- 约束与安全边界
- 实现建议
- 验证命令
- 最终报告格式

同时还包含几类附加模板：

- 数据与计量经济学 pipeline
- Windows 本机应用相关任务
- 代码修改 / 重构任务
- 文档写作任务

## 仓库结构

```text
.
├── README.md
├── README.zh-CN.md
├── LICENSE
├── .gitignore
└── skills/
    └── codex-brief-generator/
        └── SKILL.md
```

其中 `skills/codex-brief-generator/SKILL.md` 是正式的 skill 文件。

## 在 Hermes 中安装

添加这个仓库作为 skill tap：

```bash
hermes skills tap add yanyintingyou/codex-brief-generator
hermes skills install yanyintingyou/codex-brief-generator/codex-brief-generator --force
```

然后开启一个新的 Hermes 会话，或显式加载：

```text
/skill codex-brief-generator
```

## 使用示例

可以对 Hermes 说：

```text
用 codex-brief-generator 给我生成一份可以复制给 Windows Codex 的提示词。
任务：清洗原始 CSV 文件，构建面板数据，加入验证检查，并在每一步汇报行数。
工作目录：C:\Users\me\projects\fomc-capital-flow
```

Hermes 应该返回一份结构化的 `# Codex Task Brief`，可以直接复制到 Codex CLI 中执行。

## 为什么不直接让 Hermes 调 Codex？

Hermes 直接调用 Codex 适合短任务、边界清楚的仓库修改。但对于长时间任务、本地 Windows 应用、复杂数据 pipeline，手动交接往往更稳：

- 用户可以让 Codex 在真正的本地环境里运行。
- Codex 可以更自然地访问本地文件和应用。
- Hermes 专注于研究背景、任务规划、约束条件和结果审查。
- 生成出来的任务书本身可审计、可复用。

## 兼容性

这个仓库遵循 Hermes 期望的 Agent Skills 目录结构：

```text
skills/<skill-name>/SKILL.md
```

skill 本身是带 YAML frontmatter 的 Markdown 指令文件，因此也可以改造成其他支持 skill-style instruction pack 的 Agent 运行时使用。

## License

MIT License.
