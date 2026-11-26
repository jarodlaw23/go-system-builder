# 项目工作约定

- 从用户目标和可观察的验收条件出发。按需读取现有 README、`docs/architecture/`、`docs/design/`、接口定义和相关测试。现行行为以代码和验证为据，变更目标以用户确认的需求、Issue 或计划为据。
- 使用项目已有的实现、测试入口和安全控制。保护用户已有改动、凭据和真实数据。写入、网络、发布等操作遵守当前宿主权限和项目授权。
- 普通功能由当前执行者负责相关界面、接口、权限、数据和反馈的整体结果。根据范围选择验证；确需并行时先明确写入范围、共享接口和整合责任，隔离工作区、服务和产物。
- 任务有明确的执行入口时，使用相应的 [开发](.agents/skills/project-dev/SKILL.md)、[修复](.agents/skills/project-fix/SKILL.md)、[测试](.agents/skills/project-test/SKILL.md) 或 [审查](.agents/skills/project-review/SKILL.md) 方法。其他请求按实际范围直接处理。只读取本次需要的正文。
- 小任务在现有 Issue 或回复中记录结果。范围大、需要交接或有关键决策时，使用一份 [实施计划](templates/implementation-plan.md)。交接写清仓库、分支、提交或可取得的未提交改动、验收、命令、环境、实际结果和下一步。区分通过、失败、跳过和未执行。
