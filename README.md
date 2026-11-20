# opencode_teamagent

在 OpenCode (V2) 中通过纯配置实现的多模型团队讨论。一个对话中接入多个不同大模型的 agent 角色,针对同一个问题分别发言、互相质询,最后由主持人综合结论。

**日常对话不受影响** —— 团队模式**按需触发**:你不输入团队命令,OpenCode 就保持单模型;输入命令的那一刻才临时召集团队,用完即止。

## 特性

- ✅ 三个不同模型/厂商的成员:架构师 (`DeepSeek`) + 反方 (`Qwen`) + 写手 (`GLM`)
- ✅ 四个场景命令:`/team` 通用讨论、`/design` 架构决策、`/review` 代码审查、`/draft` 写作润色
- ✅ 纯配置,零依赖,无需编写插件,无需重启(OpenCode 自动热加载)
- ✅ 成员只读权限:可看代码/查资料,不能改文件、不能跑 shell
- ✅ skill 定向授权:各成员仅可加载与其职责相关的 skill

## 目录结构

```
.opencode/
├── agents/                  # 团队成员定义(每个绑定一个模型)
│   ├── architect.md         # 架构师  —— DeepSeek
│   ├── skeptic.md           # 反方    —— Qwen
│   └── writer.md            # 写手    —— GLM
└── commands/                # 触发命令
    ├── team.md              # /team   通用三模型多视角讨论
    ├── design.md            # /design 架构/技术方案决策
    ├── review.md            # /review 多成员代码审查
    └── draft.md             # /draft  写作与润色
opencode.example.jsonc       # 无密钥的示例配置(可安全提交)
```

## 快速开始

1. 把这个仓库里的 `.opencode/` 目录复制到你的项目根目录(与 `opencode.json(c)` 同级)。

2. 在你的 `opencode.jsonc` 中配置三个成员所用的模型(参考 `opencode.example.jsonc`):

   ```jsonc
   {
     "agents": {
       "architect": { "model": "your-provider/deepseek" },
       "skeptic":   { "model": "your-provider/qwen" },
       "writer":    { "model": "your-provider/glm" }
     }
   }
   ```

   或直接编辑 `.opencode/agents/*.md` 中的 `model:` 字段。

3. 启动 OpenCode,输入命令即可触发团队讨论:

   ```
   /team 这个系统该用微服务还是单体?
   /design 评估缓存方案 A vs B
   /review src/auth.ts
   /draft 打磨这段需求文档
   ```

## 模型替换指南

每个成员的文件开头都有 `model:` 字段,改这一行即可换模型:

```yaml
# .opencode/agents/architect.md
model: your-provider/deepseek
```

没有 `model:` 时,该成员会**继承**发起讨论的父会话模型。若想省成本/混合搭配,可让不同成员用不同模型。

**建议**:尽量让三个成员使用**不同厂商**的模型 —— 观点差异越大,讨论价值越高;同厂模型容易观点趋同。

## skill 授权指南

每个成员文件里有 `permissions` 段,通过 `skill` 动作控制其可加载的 skill:

```yaml
permissions:
  # 默认全部拒绝
  - action: skill
    resource: "*"
    effect: deny
  # 单独放行某个 skill(按 ID 精确匹配)
  - action: skill
    resource: think
    effect: allow
  - action: skill
    resource: mathematical-logic
    effect: allow
```

默认分队:

| 成员 | 可加载 skill |
| --- | --- |
| 架构师 | `think`, `mathematical-logic` |
| 反方 | `think`, `mathematical-logic` |
| 写手 | `think`, `norman-paper-reading`, `yq_paper_query_skill` |

想让某成员加载其他 skill(如 `pptx`),在它的 `permissions` 中按同样格式追加一条 `allow` 即可。

## 命令说明

| 命令 | 用途 | 参数 |
| --- | --- | --- |
| `/team` | 通用三模型多视角讨论 | 任何问题 |
| `/design` | 架构/技术方案决策 | 方案或选择项 |
| `/review` | 多成员代码审查(找 bug) | 文件路径或直接说"审查当前改动" |
| `/draft` | 写作与润色 | 写作任务描述 |

## 如何加新成员 / 新命令

- **新成员**:复制任意一个 `agents/*.md`,改名、换 `model:`、改 system prompt 角色描述,即可被 `/team` 等命令调用。
- **新命令**:在 `.opencode/commands/` 新增一个 md 文件,内容是"主持人流程指令",文件名即命令名。

## 注意事项

- **成本**:一次团队讨论会调用 3 个模型 × 多轮,明显比单模型贵,适合**高价值、难决策**的问题,不建议事事都开。
- **只读**:成员默认只读,不能修复 bug 或写文件 —— 修复由主持人(主会话)在讨论后执行。
- **隐私**:讨论内容会发送给你自己的多个模型 provider,注意敏感信息。
