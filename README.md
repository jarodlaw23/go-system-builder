# 轻量工作流

一套供 Codex 和 Claude Code 共用的项目工作方法。共同规则在 [AGENTS.md](AGENTS.md)，四种方法的正文在 [`.agents/skills`](.agents/skills)，Claude Code 通过 [CLAUDE.md](CLAUDE.md) 和 `.claude/skills` 读取同一份内容。

| 工作 | Codex | Claude Code |
| --- | --- | --- |
| 开发业务功能 | `$project-dev` | `/project-dev` |
| 修复已知问题 | `$project-fix` | `/project-fix` |
| 独立验证或回归 | `$project-test` | `/project-test` |
| 审查代码或方案 | `$project-review` | `/project-review` |

从项目根目录打开新会话。Codex 在输入框选择对应 Skill，或用 `$` 指定它。Claude Code 用对应的斜杠命令。随后给出本次目标和相关资料，例如 `/project-fix 保存后首次选人不刷新，验收见 Issue #42`。普通问答和小调整可以直接描述任务。

### 在项目中采用

1. 将 `AGENTS.md` 的通用约定合并进项目现有规则，保留项目自己的架构、权限和测试命令。将 `@AGENTS.md` 合并进 Claude 的项目入口。
2. 将四份 `.agents/skills/project-*`、对应的 `.claude/skills/project-*` 和 `templates/implementation-plan.md` 放入相同的项目相对路径。检查与现有 Skill 的命名冲突和忽略规则。
3. 从新会话确认入口可见，用一个限范围任务验证它能读取共享方法并运行项目已有检查。使用项目原有的权限、CI 和发布流程。旧项目启用过 Loop 的，先确认必要的安全控制由哪里执行，再安排切换。

工作流本身由 Markdown 和 Skill 元数据组成。项目测试、浏览器或数据库的运行前提仍由项目决定。

### 文档和交接

长期架构留在项目原有 `docs/architecture/`，设计和原型说明放在 `docs/design/`，可运行原型留在原源码目录。当前任务使用同一张 Issue 或一份 [实施计划](templates/implementation-plan.md)。交付记录包含版本、测试命令、环境、真实结果、遗漏和下一步。大产物放在项目已有的输出位置或 CI artifacts，交接时给接手者可读的入口。验证 JSON 由现有 runner 按需生成，接口和配置 JSON 跟随各自的消费者。

更换工具时，给接手者同一任务链接与完整提交。未提交的变更需要可取得的工作区或补丁。按改动风险选择验证和分工。

### 旧版和实施记录

旧框架保留在 `upstream-v0.3.2-workflow` 的 `0bea691fade6bd7dfccbb543dd26dfb14f57d854`。维护旧项目时，使用对应版本的二进制、配置和运行记录。新版范围、取舍与实施记录见[轻量化实施方案](docs/plans/lightweight-framework-plan.md)。

宿主说明见 [OpenAI 官方文档](https://developers.openai.com/codex/skills) 与 [Claude Code Skills 文档](https://code.claude.com/docs/en/skills)。
