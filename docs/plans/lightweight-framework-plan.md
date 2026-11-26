# 跨工具轻量工作流待实施方案

日期：2026-11-26。状态：**首版已实施，Claude Code 与 Codex 的 Linux 试点通过**。

首版实施在 `codex/workflow-lightweight` 进行，范围为本分支源码及隔离试点。现役项目的配置切换和迁移另行安排。本文记录方案和实施依据，供查阅。

首批目标宿主为 **Claude Code 与 Codex**。“Koko”已由用户确认为 Claude Code。Antigravity 保留适配位置，在取得版本、配置和运行证据后再决定实现。

## 1. 推荐决定

推荐在现有 `codex/workflow-lightweight` 分支做一次**按白名单重建的轻量版**。保留 Git 历史和旧版分支，从旧版提取有效方法，不逐个修补后继续携带整套状态机。不要物理清空整个工作目录。

新默认流程是：**读目标与必要设计 → 明确实现和验收 → 完成一个业务能力 → 做与改动相称的验证 → 留下可接手的结果**。

| 现有负担 | 新版具体处理 | 需要保留的价值 |
| --- | --- | --- |
| 固定 S0–S11 阶段链，用户称 S0–S12 | 退出新版默认执行路径，不换名称继续保留 | 需求清楚、实现完整、结论有证据 |
| 前端、后端、Contract、TASK、Sync 层层拆分 | 默认一个执行者负责一个可验收的业务能力，任务可用同一方案中的清单表达 | 必要接口约定、跨层一致性、明确整合责任 |
| S7 之后固定 QA/E2E/返修/验收流程 | 大范围测试作为独立 `test` 任务；开发保留定向自检 | 真实缺陷验证、必要边界和回归 |
| 10 类框架 Hook 与旧 Runtime 互相依赖 | 干净试点首版默认 **0 个框架自定义 Hook** | 宿主权限、项目测试和 CI、实际需要的发布控制 |
| 47 个 Skill 及 8 个角色定义 | 4 个按需工作入口，默认 **0 份自定义 Agent 定义** | 领域知识与特殊风险方法按需要提取 |
| 阶段状态、问卷、回执、锁定代次等产物 | 不进入新版默认任务 | 任务目标、当前版本、验证结果、限制和下一步 |
| Go 二进制或考虑整体改写 Node | 首版不依赖框架可执行程序，先删除需求再决定是否保留代码 | 真正独立、有消费者的检查脚本 |

这是实质减掉执行机制的方案。只增加四个入口，却让它们继续调用旧 Loop，不算完成轻量化。

同时不推荐两个极端：把所有验证都取消；规定任何任务只能由一个 Agent 完成。参考项目的真实浏览器验证发现过组件缺陷，复杂金额与文档任务也确实使用了隔离并行。应去掉的是默认重流程，根据实际风险选择验证和分工。[R3][R4]

### 1.1 “完整实施”具体包含什么

**按本方案完整实施，指完成下面的首版范围及验证。** 文中的技术比较、旧版审计证据和未来扩展不是全部都要建设的功能。首版不同时实施 Go 保留、Node 迁移、插件包装等互斥或可选路线。

| 首版必做项 | 明确交付 | 完成条件 |
| --- | --- | --- |
| 受控替换旧框架 | 当前分支中的保留/提取/退出清单及可审查 diff | 新默认入口不再调用 S 阶段、Loop Runtime、强制 Contract/TASK/Sync、旧角色和 10 类框架 Hook；旧版历史及本地保护对象可保留 |
| 一份共享核心 | 短的共同规则、项目知识索引和一份可裁剪计划模板 | 权威来源明确；不全量加载历史文档，不重复维护三套流程 |
| 四个工作入口 | dev、fix、test、review 的共享正文与必要宿主包装 | 各自能独立使用；修 Bug 不自动进入全量 QA，纯测试/审查不强制开发 |
| Claude Code 与 Codex 适配 | 各自真实可发现的入口和使用说明 | 在选定版本中验证实际加载与调用；发现差异只调整薄入口，不能仅凭文件存在宣称兼容 |
| 文档、验证与交接约定 | 计划模板中的目录、结果和交接字段，引用项目现有脚本 | 同任务信息不重复录入；JSON 有实际用途；接手者能取得版本、结果、限制与下一步 |
| 小规模验收 | 第 13 节的小 Bug、跨层功能、独立测试/审查和双向接手记录 | 验收对应实际版本与环境；没有无关阶段阻断；未执行项不冒充通过 |

首版默认由当前执行者负责完整业务能力，0 份自定义 Agent 定义，0 个框架自定义 Hook，也不要求安装框架 Go/Node 程序。参考项目已有测试使用 Go、Node 或浏览器属于业务项目依赖，不受这个决定影响。

**首版不建设**：Antigravity 适配、模型专属流程、原生 Plugin 包装、Go→Node 重写、统一调度器、私有 Memory 同步、自动分发或多环境迁移。这些不会因为“完整实施”而被自动加入。

已有项目的必要安全控制若只能由旧 Hook 提供，必须在该项目切换前解决。它是切换前提，不是要求轻量首版重新实现整个安全引擎；未解决时可以先完成干净试点，不能宣称该现役项目已迁移。

### 1.2 实施顺序与完成边界

1. **先核对保护对象与提取内容。** 固定旧版提交，列明受控删除范围，保留许可证、本方案、Git 历史、忽略数据和独立 worktree；先提取知识再删除对应旧源码。
2. **再完成轻量源码。** 建立共享核心、四个入口和薄适配；同步处理旧 CI、打包与引用，避免新版又初始化旧 Runtime。
3. **然后做两个宿主的有限试点。** 使用明确的隔离副本、基线和任务，验证加载、实现、自检与接手；实际路径和调用语法由目标版本验证决定。
4. **最后交付可使用的首版。** 给出最终文件清单、使用方法、已验证宿主/环境、结果与限制，以及恢复旧版的方法。

完成源码精简只能称“实现完成”；两个宿主的必需试点通过后，才能称“首版验收完成”。环境不可达时如实报告对应未完成项，不以增加文档替代验证，也不擅自扩大为安装或迁移其他环境。

基线方案提交为 `5e8cc5964aa481f08683089bd9ff0fec55bbb132`。首版实施授权覆盖当前分支和隔离试点，现役业务项目、其他设备和生产环境按各自范围安排。

## 2. 审计基线与证据口径

### 2.1 仓库基线

| 对象 | 当前核实结果 |
| --- | --- |
| 框架仓库 | `/home/jarod.luo/go-system-builder-jarod/go-system-builder` |
| 当前分支 | `codex/workflow-lightweight`，HEAD `0bea691fade6bd7dfccbb543dd26dfb14f57d854` |
| 旧版分支 | `upstream-v0.3.2-workflow`，同一提交 |
| 本地 `main` | `a70d5948362b776e80a4a591ee1232cbb0ea6316`；与当前分支不同，`main...HEAD` 左侧独有 6 个提交、右侧独有 4 个提交 |
| 框架工作区 | 写本方案之前干净，基线跟踪 2,250 个文件 |
| 本地保护对象 | `.local/framework-maintenance/upstream-pr-evidence-recovery` 是另一个已注册 worktree；忽略目录还含本地配置与运行数据 |
| 参考业务项目 | `/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go`，`feature/test6`，HEAD `0bb5a89d3448445c91566b842f5d2b24cb9c99d5` |
| 参考项目未提交变化 | 仅 `docker-compose.dev.yaml` 未暂存，2 行增加、2 行删除；属于现有环境配置变化，本轮不处理 |

以上为本地 Git 静态结果，没有 fetch，不能代表远端当前状态。当前轻量分支是旧版副本，但不能据此说它和 `main` 一样。以后选材以明确的旧版提交为依据。

### 2.2 证据等级

- **已确认·文件/配置/代码**：本轮读到内容、引用、提交或生产者。只能证明实现与注册存在。
- **已确认·历史记录**：已有报告、结果 JSON 或日志记载过运行。本轮不重跑，不能把历史 PASS 当成当前版本已通过。
- **官方描述**：官方文档说明产品能力，不代表当前 App、CLI 和配置已经启用。
- **待验证**：缺少实际宿主调用、目标环境或消费者证据。
- **设计建议**：下面提出的新结构与取舍，不是已实施事实。

盘点覆盖旧版 47 个 Skill 的入口元数据，正文按引用与风险重点抽样；不是对每条指令有效性的实验。参考项目抽查原型、统一方案、一个已提交小修复、一组业务交付 JSON 和最近交接文档，没有审遍全部历史。没有精确 token 或耗时节省数据。

## 3. 从参考项目提取什么

### 3.1 原型与一份当前方案

参考项目最近的 FX、Non-HK、OCR、LPF 统一方案写清楚了现有代码、业务边界、数据和接口、实施任务、验收矩阵、环境及回退，并明确“旧讨论不是第二套执行口径”。这正是应借鉴的实施方案结构。[R1]

但该方案第 5 行明确**尚未实施正式业务代码、数据库、桶策略或外部调用**。原型文档也明确使用模拟资料和保存。它们能证明设计与计划的组织方式，不能证明这一批功能已完成。约 51 KB 的复杂方案也不应变成所有小任务的必填模板。[R1][R2]

所谓“Codex 详细实施方案”的可见构成，是目标、现有实现、关键决定、任务、验收和验证路径；本轮没有证据表明必须依赖某个 Codex 专属内部计划格式。把这些信息写成普通 Markdown，Claude Code 同样可以接手。

### 3.2 一个适合轻量修复的真实例子

注册地与法人代表董事工作有方案、实际提交和验证记录。浏览器曾发现“首次选人后姓名不刷新”；提交 `08a797d86` 只改了 2 个文件，7 行增加、3 行删除，包括响应式取值修复和模拟真实就地更新的组件测试。[R3]

新版对应做法：读取复现与目标组件 → 查清首次更新失效原因 → 修复取值并补定向验证 → 报告结果和边界。没有理由为这个修复再生成独立前后端合同、建立整套 QA 任务或推进全部阶段。

同一项目涉及真实 MySQL、历史签署文件和金额的变化有更广验证。应按风险选验证，不能把“小修复只做小验证”扩大成“所有功能只查几行代码”。

### 3.3 JSON 是验证证据，不是每个任务的作业表

- LPF 原型脚本在断言和浏览器错误检查后写出 `result.json`，归档文件由原型验收记录引用。没有找到它们作为全局门禁的消费者，也没有找到临时结果自动改名归档的持久化代码。[R2][R5]
- 转股浏览器 runner 记录 HEAD、7 个源码文件运行前后哈希、检查项和未覆盖范围，输出 `report.json`。归档报告有 22 项检查；脚本明确使用合成 API，不能证明真实后端事务通过。[R6]
- 后续最终清单记录 54 个文件。浏览器报告中的 7 个源码哈希与清单相同，只能证明这 7 个输入一致，不能说最终所有代码都在同一轮浏览器运行中通过。[R6]
- Word 等大产物留在忽略目录，仓库保留摘要和哈希。只有本机路径和哈希，仍不足以让另一台电脑取得文件。[R7]

建议保留“脚本产证据，一份报告说明结论与边界”。不要把 Markdown 结论机械抄成另一份交付 JSON。

### 3.4 通过 Issue 和代码版本交接

最近提交 `d0ad0c8f7` 的做法很适合通用框架：一个可独立验收的能力使用同一张 Issue，开发、测试和返修共享目标；交接带分支、已推送完整 SHA、测试入口、环境和实际结果；纯测试可以直接交报告。[R8]

本轮只核实已提交的交接设计，没有访问 GitLab 证明看板已按此运行。移动卡片也不等于启动模型。

**不要整套复制参考项目配置。** 它的 `AGENTS.md` 仍有 DRIVE、S0–S11，`CLAUDE.md` 仍要求每次响应前检查 Loop，并保留旧 Hook 配置。轻量经验来自具体工作产物，不代表这个项目已经完成框架轻量化。[R9]

## 4. 共享核心与薄适配

把三个维度分开：

| 维度 | 放什么 | 本版决定 |
| --- | --- | --- |
| 模型与提供商 | Claude、GPT、Gemini、MiniMax、GLM 的能力、协议和供应商配置 | 没有实测差异就不建模型专属流程；提供商凭据留宿主配置 |
| 执行宿主 | 规则发现、Skill 注册和调用、权限、Hook、会话 | Claude Code、Codex 各有薄入口；不假设协议相同 |
| 运行环境 | OS、路径、Node/Go/浏览器/数据库、网络与服务 | 在具体任务验证记录中写明，跨环境结果不自动等价 |

### 4.1 建议的权威来源

| 内容 | 唯一维护来源 | 各入口如何到达 |
| --- | --- | --- |
| 项目共同规范、目录和验证命令索引 | 项目根 `AGENTS.md` | Codex 由其规则机制发现；Claude 的 `CLAUDE.md` 引入共同文件；实际加载待试点验证 |
| 四种工作方法 | `.agents/skills/project-{dev,fix,test,review}/SKILL.md` | Codex 原生发现候选；Claude 对应薄 Skill 明确读取这份正文 |
| Claude 专属行为 | `CLAUDE.md` 和 `.claude/skills/.../SKILL.md` | 只放该宿主的导入、调用提示和必要元数据，不维护第二份方法正文 |
| 当前任务 | 现有 Issue 或 `docs/plans/<主题>.md` | 任务链接明确指定；二者并用时，一个保存细节，另一个链接它，不重复维护清单 |
| 项目业务与架构事实 | 项目现有业务文档、`docs/architecture/`、代码和实际接口定义 | 由本次任务定位后读取，不整目录常驻加载 |
| 验证 | 项目现有测试或脚本入口 | 各宿主调用同一命令，自己适配命令执行与权限 |

框架源码仓库是通用方法的上游来源；项目采用某个明确版本后，项目内文件是该项目执行口径。本版不做跨环境自动同步。项目已存在的根规则必须合并项目事实，不能用模板覆盖。

### 4.2 首版候选文件布局

```text
README.md                                  # 使用方法、目录和升级边界
LICENSE                                    # 保留来源与许可证
.gitignore                                 # 区分入口源码与私有运行产物
AGENTS.md                                  # 短的共同项目规则与索引
CLAUDE.md                                  # Claude 薄入口
templates/implementation-plan.md            # 一份可裁剪计划模板
.agents/skills/project-dev/SKILL.md           # 四份方法正文，唯一维护
.agents/skills/project-fix/SKILL.md
.agents/skills/project-test/SKILL.md
.agents/skills/project-review/SKILL.md
.claude/skills/project-dev/SKILL.md            # 四份 Claude 薄入口
.claude/skills/project-fix/SKILL.md
.claude/skills/project-test/SKILL.md
.claude/skills/project-review/SKILL.md
docs/plans/lightweight-framework-plan.md      # 本维护方案，不自动分发给业务项目
```

这是候选范围，不是按数量验收的指标。优先不用符号链接，避免 Windows 权限和归档差异。Claude 薄入口只描述何时使用以及应读取的共享正文；它是否会加载、是否重复发现同名 Skill，必须在具体版本中验证。

旧 `.gitignore` 忽略整个 `.claude/` 和 `AGENTS.md`，不能直接保留。批准后应精确允许新版入口源码，同时继续忽略本地设置、秘密和运行数据。[F1]

共同入口只含短规则和索引；四份 Skill 也不相互强制加载。调用一个 Skill 不代表卸载此前读过的内容，长会话切换任务时仍可用新会话加交接记录减少遗留上下文。

## 5. 四个入口怎么用

先用 `project-` 前缀减少与已有命令冲突；最后名称在检查宿主和项目已注册入口后确定。语义相同不要求调用符号相同，Claude 的斜杠命令和 Codex 的 Skill 选择由各自界面提供。

| 入口 | 输入与适用场景 | 最小步骤 | 输出与完成条件 |
| --- | --- | --- | --- |
| `project-dev` | 已知需求、原型或设计，做普通业务功能 | 找现有实现 → 在同一计划中明确范围和验收 → 按业务能力实现 → 定向验证 | 功能和验收对应；说明实际环境、未完成项；小任务可直接使用 Issue |
| `project-fix` | 可描述的问题、期望与复现，做 Bug 修复 | 核实问题 → 找原因 → 最小修复 → 验证原场景与受影响边界 | 原问题和必要反例有结果；不能复现时明确证据限制 |
| `project-test` | 给定版本、范围和环境，做独立功能验证或回归 | 核对版本与环境 → 选项目测试入口 → 执行与整理失败 | PASS/FAIL/SKIP 或环境阻塞及证据；不默认顺便重写实现 |
| `project-review` | 代码、diff、方案或产物，做独立审查 | 明确范围 → 读真实调用与边界 → 给可定位问题和影响 | 发现与依据、未覆盖项；默认只读，修复需属于已有授权或后续任务 |

测试发现缺陷，可在同一任务下转入修复；不要求另建一套 Contract/TASK。纯后端修改不强迫检查页面，纯文案修改不强迫启动数据库。测试与审查不是所有开发结束后的必经新阶段。

### 5.1 一份实施方案的最少内容

```markdown
# <业务能力或问题>
状态：待实施 / 进行中 / 已完成但有明确未验项
任务来源：<Issue 或需求>；基线：<仓库、分支、提交>

## 目标与验收
用户能做什么；如何判断成功；本次范围。

## 依据与关键决定
认可的原型/设计、现有实现入口、必要接口/数据约定。

## 实施清单
按业务结果拆的少量事项；依赖和需要顺序修改的公共文件。

## 验证
验收项对应的命令或人工步骤、所需环境、实际结果及限制。

## 当前交接
已完成/未完成、候选提交或未提交 diff、证据位置、下一步。
```

无需每次把上述内容复制成新文件；Issue 已包含就复用。小 Bug 可以只有几段，跨模块任务再展开验收矩阵、数据变更、兼容和回退。API schema 已在代码或 OpenAPI 维护时，计划链接它并记录变化，不生成竞争版本。

### 5.2 开发自检与专项测试的边界

- 小 Bug：验证复现路径及实际受影响的必要边界。已有检查足够则复用，不为固定数量或格式要求造测试。
- 普通跨层功能：相关界面、API、权限、持久化与反馈有共同验收。至少验证本次真正改动的调用关系，Mock 和真实环境分别标记。
- 只改样式或文案：对应界面或静态检查足够时，不升级为全套回归。
- 金额、迁移、权限、历史文件：仍需相称的领域验证；浏览器、数据库或文件检查可能必要。
- 大范围 QA/E2E、性能和跨浏览器：独立选择 `project-test`，说明范围及环境；不由每次保存或停机 Hook 自动触发。

如果验收要求真实服务而环境不可达，交付“代码完成、该项未验证”，不能把状态改成全部通过。是否必须先补齐该验证，由任务验收条件决定。

## 6. Skill 筛选清单

旧 47 项不整包安装。下面“提取”表示逐段改写，不是把旧全文装进新入口；“按需”不是取消相关业务或安全要求。

| 处理 | 完整名单 | 去向 |
| --- | --- | --- |
| 13 项提取有效方法 | `requirement-funnel`、`specification-planning`、`user-story-design`、`user-flow-design`、`frontend-engineering`、`backend-engineering`、`api-contracts`、`testing-strategy`、`integration-verification`、`code-quality`、`bug-resolution`、`impact-analysis`、`acceptance-and-handoff` | 目标/实现/契约进入 dev；定位与影响进入 fix；验证方法进入 test；问题定位和质量判断进入 review；共有完成条件放共同规则 |
| 设计类 4 项按需 | `design-foundation`、`design-critic`、`ui-prototyping`、`scenario-model-design` | UI/原型任务需要时再选入；已批准原型可直接作为输入 |
| 风险类 5 项按需 | `security-review`、`database-change`、`state-machine-design`、`performance-review`、`reliability-review` | 高风险任务按影响读取，不默认每次全套运行 |
| 浏览器类 2 项按需 | `e2e-browser-testing`、`playwright-e2e` | 优先复用项目与宿主已有浏览器能力；只留下项目专有操作知识 |
| 技术栈类 17 项按需 | `casbin-authorization`、`dag-design`、`domain-driven-design`、`eslint`、`gin`、`gorm`、`http-api-design`、`jwt-authentication`、`openapi-swagger`、`pinia`、`prettier`、`s3-object-storage`、`structured-logging`、`typescript-type-safety`、`vitest`、`vue-devtools`、`vue-router` | 通用语法说明不默认复制；保留项目独有规则、真实命令和踩坑经验 |
| 6 项退出新版默认入口 | `loop-orchestration`、`agent-dispatch`、`team-planning`、`document-verification`、`clean-round-evaluation`、`runtime-recovery` | 与旧阶段、派发和 Runtime 一起留旧版；需要恢复旧任务时使用匹配旧版 |

优先保留：可观察的成功条件、用户动作对应反馈、接口真实调用、状态边界及反例。删除强制阶段、锁定代次、重复角色回执、固定测试比例和为证明流程完成而生成的文件。[F2]

低频和没找到调用记录都不等于无用。设计、数据库、权限等选材留在旧分支可追溯；首版按一个具体任务需要选择，不把 28 个可选 Skill 重新全部注册。

## 7. Agent 数量与分工

**默认当前宿主执行者完成工作，不要求再创建一个 Worker；需要委派时使用宿主原生能力。** 首版不迁移旧 8 份角色定义。

理由是旧角色预载 `agent-dispatch`，Builder 有固定写域，S6 接受的角色也有约束。原样保留会把旧流程重新带回来。[F3]

只有下面情况值得增加 Agent：可以独立调查的范围、独立写域的大任务、需要独立判断的审查。任务描述写清目标、允许写入的范围、共同接口和完成条件即可，不建立新的角色注册、心跳或调度系统。

负责整合的执行者始终对完整验收负责。并行时先检查公共文件、端口、测试数据库和产物目录；不同 worktree 不会自动隔离服务、数据库和秘密。相同文件需要顺序集成，不能默认所有 Agent 可以同时写同一工作区。[R3][R8]

顺序接手只需明确当前版本与交接记录。跨设备未提交变化还需要补丁或其他明确传输方式；只写一个分支名，另一设备无法取得这些变化。

## 8. 文档、JSON 和产物归属

沿用项目已经合理的目录，不为满足框架强制搬动已有文件，也不创建没有内容的目录。框架模板不携带参考业务项目的财务规则、私人配置或原始业务资料。

| 内容 | 建议位置 | 生产者与消费者 | 版本控制、失效与清理 |
| --- | --- | --- | --- |
| 长期业务与架构事实 | 现有业务文档、`docs/architecture/` | 维护实现的人更新，相关任务按需读 | 纳入版本；行为变化时更新，避免复制到每份计划 |
| 设计依据、原型说明 | `docs/design/`；可运行原型沿用现有 `web/prototypes/` 等源码位置 | 设计工作产生，开发和验收引用 | 一个源码权威；导出图/HTML标明来源和认可范围 |
| 当前实施方案 | Issue 或 `docs/plans/<日期>-<主题>.md` | 执行者维护，同任务其他工具读取 | 同一任务更新同一份；复杂历史决定保留链接 |
| 交付或专项报告 | Issue 评论或 `docs/reports/` | 执行者/验证者总结，接手者读取 | 已有计划足够时不另建报告；注明版本和环境 |
| 接口、数据、配置 JSON | 原消费者所需位置，如现有 schema/config/fixture 目录 | 实际代码、配置加载器或测试读取 | 跟随原有协议，不能统一搬到“JSON目录” |
| runner 结果 JSON | runner 原输出目录；必要摘要归档 `docs/reports/evidence/<任务>/` | 验证脚本产生，报告、人或现有 CI 读取 | 绑定输入与本次运行；旧结果不覆盖新版本结论 |
| 交付 JSON | 只有现有机器消费者或明确交付需求时才指定 | 明确谁生成、谁读、哪些字段不能缺 | 没有消费者就不新增；不与 Markdown 双份手工维护 |
| 日志、截图、Word、临时数据库 | 已有忽略输出目录或 CI artifacts | 工具生成，定位问题或验收时读取 | 按任务保留期限清理；跨设备需提供实际可访问的附件 |
| 旧 Runtime JSON、journal、问卷、回执 | 旧发行组及旧项目运行位置 | 旧状态恢复与流程引擎消费 | 不搬入轻量任务；恢复必须使用匹配旧版和完整证据 |

保留 JSON 前只需回答：**谁生成、谁读取、它证明什么、何时过期**。来源与命令应可追溯；没运行、跳过和环境失败不能写成 PASS。报告可以记录命令、系统、候选 SHA、dirty/diff 信息、实际结果与未覆盖项，不为本版新造通用 JSON schema 或报告解析器。

如果项目 runner 继续使用固定 `report.json`，先避免失败时引用上次成功文件；适宜用独立运行目录或在现有报告中校验本次运行身份。是否需要修改 runner 应在对应项目单独批准，不由框架安装顺手重写。

旧框架的 `cases.json`、场景覆盖和 Hook 日志存在实际消费者，不能仅凭后缀或“可再生成”判定可删。它们随旧生成器和消费者退出轻量发布，属于整条机制退出，不是把现役项目的输出单独删除。[F4]

## 9. Hook 的推荐与限制

### 9.1 Codex 是否需要 Hook

**使用这套轻量工作流不需要先安装 Hook。** 规则、按需 Skill、项目脚本和可读交接已经能够组成首版。也不能说“Codex 不支持 Hook”：前序本地检查在 Codex 0.165.0 二进制中发现相关实现标识，但当前可见配置没有框架 Hook 注册，未启动真实事件验证。

Claude Code 有原生 Hook 文档和当前注册证据；两者存在同名概念不代表输入、退出码、阻断或 App 行为一致。Antigravity 的事件及兼容性未验证。首版不建立统一 Hook 调度器。

### 9.2 旧 Hook 按职责处理

旧 `settings.json` 注册 10 类事件，多个工具事件匹配 `*`，调用 `loop-harness`。Worker bootstrap 又生成这 10 类注册，迁移校验也要求其存在；只删主设置里的几条，会被其他入口重建或令旧校验失败。[F5]

| 旧事件 | 主要要处理的职责 | 轻量版建议 |
| --- | --- | --- |
| `PreToolUse` | 混有写域/发布控制、阶段限制、证据与格式要求 | 阶段与回执门禁退出；真实安全限制先映射到宿主、仓库或 CI 的执行机制 |
| `SessionStart` | Runtime 恢复、身份和流程上下文 | 不注入整套流程；按当前任务读计划与交接 |
| `SubagentStart` | 角色、工作包与任务上下文 | 改为委派时的明确任务描述，不强制建立旧工作包 |
| `SubagentStop` | 完成回执、阶段交付与整合检查 | 由负责人核对实际改动和约定验证，不以回执格式判业务完成 |
| `TeammateIdle` | 团队协作中的流程处理 | 首版不注册；使用宿主原生协作状态 |
| `Stop` | 阶段结束与交付门禁 | 首版不拦每次对话结束；完成条件写在任务，验证留实际结果 |
| `PostToolUse` | 状态/材料变动观察与反馈 | 不在每次工具成功后重跑验证或更新中央状态 |
| `PostToolUseFailure` | 失败观察，含 planning stamp 清理 | 随旧状态机制退出；它并非纯日志，旧运行环境不可单独盲删 |
| `ConfigChange` | 配置观察/审计 | 首版不迁移；真实组织配置控制由对应宿主机制承担 |
| `PreCompact` | 压缩前流程上下文 | 使用任务中的最新交接；不把私有会话恢复当跨工具基础 |

这是面向新架构的职责去向，不是对旧项目逐事件关闭的操作清单。[F6]

### 9.3 “0 个框架 Hook”成立的条件

干净试点不需要安装旧引擎。已有项目切换前，仍要核对旧 `PreToolUse` 的安全职责：源码中确有受保护发布命令的阻断，不能用一段 Markdown 冒充等效控制。[F7]

- 业务验收由功能与数据结果证明；质量检查由项目脚本及实际 CI 执行。
- 文件、命令和网络权限由各宿主实际权限配置、OS/容器与凭据边界执行；主分支与发布由仓库保护及发布环境执行。
- 这些机制不是自动等价。如果目标项目仍依赖旧 Hook 才能满足必要安全要求，先保留该项目旧版本，或单独实现一个必要的窄控制并验证，再切换。
- 不修改组织受管配置，不放宽权限，不建议所有 Hook 错误一律放行。

### 9.4 未来确实需要 Hook 时的设计约束

现有必要安全控制缺少可用替代，或实际试点证明存在值得自动化的重复问题时，考虑一个职责单一的适配器。必要控制在切换前补齐，不等出现安全失误才处理。共享检查继续放项目脚本，宿主包装只转换事件输入、调用和反馈。

输入按宿主版本解析，stdout 只写合法协议，诊断写 stderr；限制输入大小、运行时间与输出量。不同事件分别处理允许、拒绝、提醒、失败和超时，不复制另一宿主的退出码约定。结束类事件避免重复触发；不因证据缺失要求无限重试。安全控制失败不能冒充成功，质量提醒失败也不能自动升级成全工作流阻断。

Hook 不下载、不编译、不调用模型，不在每次工具操作时跑全量测试，不读写中央阶段状态。是否采用及具体安全语义另列可审查的小范围变更。

## 10. 审计时的跨宿主兼容性与验证边界

### 10.1 本机安装、当前配置与运行记录分别看

| 宿主 | 静态安装信息 | 当前框架工作区的可见配置 | 已有运行记录与缺口 |
| --- | --- | --- | --- |
| Claude Code | npm 包 `2.1.329`，包声明 Node ≥22，入口 `bin/claude.exe` | 根 `settings.json` 是模板；本工作区 `.claude/settings.json` 不存在，`settings.local.json` 仅有 env；用户设置无框架 hooks | 有历史 Claude 会话记录；不能据此证明当前模板全部生效。用户另启用 Ralph Loop 1.0.1，其插件有 Stop Hook |
| Codex | npm 包 `0.165.0`，入口 `bin/codex.js` | 用户 `~/.codex/config.toml` 无 Hook 命名配置项；用户/项目 `hooks.json` 未发现；本工作区 `.codex/`、`.agents/` 为空 | 当前会话元数据记载 Codex Desktop、CLI 0.165.0、当前仓库及 workspace-write；App 自身版本未核实 |
| Antigravity | 只发现 `~/.antigravity-hub-server/bin/2.35.0-x64` 目录名，内容为空 | 对应 data 为空，未见 `~/.gemini` | 不能以目录名认定安装成功；版本、界面、项目运行均未验证 |

当前框架工作区根目录没有 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md`，也没有 `.claude/rules/skills/agents/commands`。47 个 Skill 和 8 个 Agent 是仓库根目录里的源码资产，不能直接统计成当前会话全部已加载。参考业务项目则有已安装的旧入口和 10 类框架 Hook 配置，两处配置不同。[H1][H2][R9]

当前 Codex 会话记录的模型标识为 `gpt-6.2-astra`；这只能说明该次会话，不能确定其他宿主、项目或云端的实际供应商路由，也不证明需要模型专属工作流。未完整检查插件或受管层的所有注册，不能宣称任何宿主全局无 Hook。

本机存在 Claude 的 `~/.claude/projects/`，本框架和参考项目对应的项目目录中也存在 `memory/`；Codex 存在 `~/.codex/sessions/` 与 `archived_sessions/`，未见 `~/.codex/memories/`。这些只是可见目录事实，不证明每次自动加载、完整 Memory 机制或云端访问边界。项目中的旧 `.claude/loop-state.json` 是另一套框架状态，应与宿主会话区分。新任务以仓库/Issue 交接，不依赖另一工具读取这些私有目录，也不在本轮修改它们。

### 10.2 能力共用与适配

| 能力 | Claude Code | Codex | Antigravity |
| --- | --- | --- | --- |
| 共同业务文档、验收、任务记录 | 可通过文件读取共用 | 可通过文件读取共用 | 文本可共用，真实读取未验证 |
| 规则入口 | `CLAUDE.md`；参考项目已有 `@AGENTS.md` | `AGENTS.md`；当前 App 使用规则机制 | 真实发现路径待核实 |
| Skill 正文 | 拟由 `.claude/skills` 薄入口读取共享正文 | 拟由 `.agents/skills` 承载正文 | 不因支持 `SKILL.md` 就认定兼容 |
| 显式/自动调用 | 原生命令及描述匹配；具体元数据行为要按版本验证 | 原生 Skill 选择及自动匹配；界面/版本需验证 | 未验证 |
| 项目验证脚本 | 复用原命令，受实际权限和环境影响 | 同左 | 命令执行能力未验证 |
| Hook | 原生文档存在；旧项目已注册 10 类 | 本地版本有实现线索；当前注册/事件执行未证实 | 未验证 |
| 权限 | 宿主权限与项目环境，不能由共同文档强制 | App/CLI/云端权限与沙箱分别核实 | 未验证 |
| 私有 Memory/会话 | 宿主本地项目记录，不作为交接权威 | Codex 本地/远端会话各有可访问边界，不假设与 Claude 互通 | 未验证 |
| 原生插件 | 可做后续包装，首次不采用 | 包装格式和版本单独核实 | 不承诺兼容 |

配置存在、元数据被发现、正文被加载、某任务实际使用是四件事。试点要分别观察；不能以文件同名或 Agent 自述替代证据。首版先把多步入口作为显式选项，避免自动触发范围太宽；是否关闭某个 Skill 的隐式调用，用各宿主真实支持的元数据，不能自造通用字段。

Claude 官方 Hook 说明见 [Hooks reference](https://code.claude.com/docs/en/hooks)。前序读取成功，本次补读遇到 DNS 限制。OpenAI 官方 Hooks/配置页面本轮访问遇到 403，不能据此对最新 Codex App 的全部行为作结论；安装信息与本地配置只能作为另一层证据。未启动 Claude、Codex 或 Antigravity 做兼容演练。

本地 Linux 的静态读取，不证明 Windows 原生、macOS、SSH VM 或云端可运行；WSL 结果也不能替代 Windows 原生。不同环境是否能访问仓库、GitLab、数据库和外部模型必须分别记录。

模型差异首版只保留实际调用需要的供应商/模型配置，不把旧规则按 Claude/GPT/Gemini/MiniMax/GLM 各复制一套。本轮没有同任务对照结果，不能证明某个模型需要固定更多反思、角色或阶段。只有出现可复现的工具参数、结构化输出或上下文限制，才增加有版本与任务证据的差异项。

## 11. Go 与 TypeScript/Node 的路线

**推荐先让轻量首版不依赖框架程序。** 如果旧 Runtime、派发、状态和门禁都不采用，把它们整体翻译为 TypeScript 只会得到另一个重框架。

现有 Go 模块不仅启动 Hook，还承担 CLI、状态恢复、策略、工作包、语义校验和发布/安装。静态统计为 1,789 个跟踪 Go 文件、374,411 行；其中 `cmd/` 与 `internal/` 的非测试代码为 604 个文件、163,660 行。行数包括注释和空行，只表示规模，不表示全部需要迁移。

`go.mod` 声明 Go 1.26，直接依赖 JSON Schema 库，`x/text` 为间接依赖；扫描未发现 `import "C"` 或 `#cgo`。发布脚本已经使用 `CGO_ENABLED=0` 构建 Linux/macOS amd64/arm64、Windows amd64 五种产物并注入版本；发布流程已有验证与校验和。`build-info` 实现包含实际程序路径、OS/arch、Go 版本和 VCS 信息。因此保留 Go 路线应完善现有机制，不应声称多平台自动发布尚未建立。[F8][F9]

用户观察的 Windows 偶发错误是迁移动机，尚不是“Go 导致错误”的结论。当前 Hook 命令包含 `$CLAUDE_PROJECT_DIR` 和 Shell 引号；路径、Shell、`.exe` 选择、权限、二进制版本、协议和运行状态都应分别检查。交叉编译成功也不证明目标平台 Hook 已实际通过。

可定位的证据包括：历史 `CI-AUDIT.md` 记录 Windows 失败及路径归一化、测试断言、目录 fsync 修复；PowerShell 启动器只接受 X64；检查执行器使用 `sh -c`；非 Linux 实现明确不支持隔离 Worker checks。现有平台矩阵没有证明真实宿主→启动器→Hook 的完整链路已通过。这些分别属于平台实现、Shell/能力前提和测试覆盖，不应统一归因为 Go 二进制分发。[F10]

用户所述偶发报错的原始日志仍缺失，具体发生在启动、协议处理、检查执行还是状态恢复尚未定位。批准后的旧版故障调查应先取得命令、错误输出、OS/CPU、宿主版本与实际二进制信息，不先改语言。

| 方案 | 成本与依赖 | 跨平台/可靠性 | 回退与推荐 |
| --- | --- | --- | --- |
| 保留 Go，完善启动和发布 | 复用现有代码；运行时不要求 Go；继续分发匹配二进制 | 需核验架构、校验值、版本和原生启动，已有构建基础 | 旧版维护或必须保留独立程序时合理；不能只换打包就称流程已轻量 |
| 精简后仅迁轻量适配模块到 Node | 小 CLI/配置处理可较容易共享 JS；新增明确 Node 前提 | 解决部分二进制分发问题，仍需处理 PATH、Shell、协议与进程回收 | 若试点证实需要，优先这一方案；只迁已划清职责的模块 |
| 精简后整体迁 Node | 需重做保留模块与验证，风险随保留范围增大 | 单 JS 包仍不等于三宿主/三平台兼容；原生依赖可能重新引入分发问题 | 首版无足够收益证据，不推荐预先承诺重写 |

以后若采用 Node，候选基线为 **Node.js 24.x LTS**，实施前确认仍受支持的安全版本并固定测试范围；这是拟定要求，不是当前所有 Agent 环境已满足。

具体要求：

1. 安装前核实 Agent 进程环境能找到已批准的 Node。GUI App 不一定继承终端 PATH；可按宿主配置明确可执行文件位置，不能仅以用户终端运行成功为依据。
2. 发布预构建 JavaScript，运行端不需要 TypeScript 编译器。优先标准库、无原生扩展、无平台专用依赖；引入例外时重新评估分发。
3. Hook 必须可离线执行，不能在回调里 `npx` 下载、安装依赖或编译。正常安装和升级与回调分开。
4. 宿主协议包装分开，公共检查返回普通结果。参数传递避免不必要的 Shell 拼接；stdout/stderr、退出码、超时和子进程结束按各宿主验证。
5. 代表模块选择无状态的配置读取或结果摘要适配，不能选旧 Loop Runtime 作为首个迁移对象。
6. 批准后覆盖 Windows 原生、Linux、实际目标 macOS；包括空格/中文路径、非登录 GUI 环境、Node 缺失、无网络、输入损坏、超时、并发以及重复注册。
7. 一个事件只有一个活动实现。先在隔离副本验证，再一次切换注册；失败恢复旧版本和匹配配置。短期对照只作为离线测试，不能 Go/Node 同时执行有副作用回调，不长期维护两套业务逻辑。

## 12. 模块化还是插件打包

| 选择 | 适用规模与收益 | 当前决定 |
| --- | --- | --- |
| 少量共享 Skill + 各宿主必要入口 | 文件少、容易审查与按项目调整，无需引入安装器 | 首版推荐 |
| 同仓共享正文/脚本 + 各宿主原生 Plugin 包装 | 多项目需要版本化分发、统一升级或一键启用时有价值 | 等两个宿主试点稳定、重复维护成本真实出现后再做 |

未来插件只能包装同一份源码，可在构建时生成平台需要的布局；不能把共享方法复制成三份人工维护。插件能打包入口不代表统一权限、会话或 Memory，也不代表所有内容都只在调用命令时加载。不要另造插件管理器。

## 13. 批准后实施顺序与变更清单

第一、二批组成第 1.1 节已批准的首版实施范围。条件扩展按实际需要另议。

### 第一批：确认选材并产出可审查轻量版

1. 保留当前分支名，固定旧版提交；按第 1.1 节执行。用户有新补充时先更新其影响范围，不把它默默加入所有任务的默认流程。
2. 对受 Git 跟踪的文件生成“保留/提取/退出”清单，展示将退出的文件和仍需保留的引用。保护本方案、许可证、`.git`、本地忽略数据和已注册 worktree。
3. 在该分支按白名单替换产品结构。退出 `cmd/loop-harness`、旧 `internal/` 执行机制、阶段控制文档、旧角色与安装/迁移链；其中有独立价值的方法先提取再退出。它们在旧分支和历史仍可取回。
4. 创建共同规则、四份方法正文、Claude 薄入口、计划模板和短 README。修改 `.gitignore` 的源码白名单；旧 CI/打包入口同步退出或改为实际轻量检查，避免继续初始化 Runtime。
5. 检查剩余引用和默认注册，确保新版入口没有再调用旧 Loop，也没有自动加载旧分支或归档中的 Skill。

“清空”仅指批准后的受控源码替换。不能执行整目录删除或 `git clean -fdx`。当前目录中确有独立 worktree 和忽略状态；旧仓库源码清理与业务项目运行配置切换是两件事。

#### 源码选取清单

| 处置 | 基线文件或来源 | 新版落点 |
| --- | --- | --- |
| 保留 | `LICENSE`、本文 | 原路径 |
| 替换 | `README.md`、`.gitignore` | 同路径提供用法、旧版恢复入口和准确的忽略边界 |
| 提取规范 | `AGENTS-template.md`、`docs/DOCUMENT-MAP.md`、需求和原型模板 | 根 `AGENTS.md`、实施计划模板与 README 的目录及交接约定 |
| 提取方法 | 第 6 节列出的 13 项方法 | `.agents/skills/project-{dev,fix,test,review}/SKILL.md`，各自的 `agents/openai.yaml` 声明显式调用 |
| 适配 Claude | 项目规则导入与项目 Skill 机制 | `CLAUDE.md` 和 `.claude/skills/project-{dev,fix,test,review}/SKILL.md` |
| 退出当前源码 | 基线 `5e8cc59` 中除上述 4 个保留或替换路径外的全部跟踪文件，共 2,247 个。含 `cmd/`、`internal/`、`agents/`、旧 `skills/`、`tests/`、`tools/`、`packaging/`、`blueprint/`、旧 `docs/` 与 `.github/workflows/` | 在旧版提交中查阅或恢复，具体文件以 `git diff --name-status 5e8cc59` 为准 |

本地 `.claude` 设置和状态、`.local`、`dist`、旧二进制及其他未跟踪文件留在原处。源码替换以受跟踪路径清单为边界。

### 第二批：一个修复试点，然后一个跨层功能

| 试点 | 做法 | 通过标准 |
| --- | --- | --- |
| 小 Bug | 在隔离副本使用一个类似“首次选择未刷新”的代表问题；两个宿主分别在等价基线执行 | 找对共享规范，修复与复现对应，定向验证有真结果；无阶段状态、无无关 QA、无框架 Hook 依赖 |
| 普通跨层功能 | 选有明确原型与 API/数据验收的有限能力 | 一个负责人完成相关前后端与整合；真实和模拟验证区别明确 |
| 纯 test / review | 指定候选 SHA，仅测试或审查 | 不强迫创建开发合同、改实现、推进阶段；交付失败/阻塞也是有效结果 |
| 跨工具接手 | Claude 写交接后 Codex 接手，再做一次反向接手 | 不依赖前一工具私有 Memory；能取得准确版本、证据、限制和下一步 |

不直接用已修好的历史问题宣称“修复成功”。如选历史案例，先在专门副本固定含缺陷的基线；绝不把回退故障引入用户正在开发的参考项目。

观察入口发现、实际读取、脚本调用、退出结果和生成文件。元数据出现不算已加载全文，模型自述不算使用证明。检查普通问答与无关任务是否误触发流程，也检查显式调用后所需正文是否确实读取。

记录任务完成与返修、框架导致的无关步骤/阻断、交付物是否有消费者、验证耗时；能取得宿主 usage 才记录 token。不同模型、缓存和任务复杂度分别注明，不承诺未经测量的节省比例。

### 条件扩展：决定是否需要一个可选 Hook，再考虑分发

先完成必要安全控制映射。存在尚无替代的必要控制，或试点暴露值得自动化的明确缺口时，才新增窄 Hook 或 CLI，再根据剩余功能决定 Go/Node。没有这些需求就维持零 Hook。插件包装也按真实多项目分发需求另行决定。

### 具体范围与恢复

| 变更对象 | 实施范围 | 依赖与恢复 |
| --- | --- | --- |
| 当前框架分支的跟踪源码 | 白名单重建、退出旧阶段/Runtime/分发链 | 先保留提取内容与可审查 diff；用旧提交取回单项，已有批准改动用反向提交恢复 |
| 47 Skills / 8 Agents | 按第 6、7 节筛选，不整包迁移 | 旧分支保留原文，必要时按需重新提取 |
| 共同入口与 Claude 包装 | 新建/精简，选择真实发现位置 | 先验证路径与版本；失败可移除新增入口，不触碰旧业务项目 |
| Hook、权限和运行配置 | 仅在明确批准迁移的目标副本做切换 | 先确认控制由谁执行；旧二进制、配置、状态/journal/证据按版本成组恢复 |
| 参考业务项目 | 首批只作为证据和候选试点来源 | 本方案不授权修改或安装；实际试点位置和范围另行确定 |
| 其他设备、SSH、云端 | 无自动同步或迁移 | 每个环境独立核实网络、依赖与授权 |

## 14. 审计时的限制

- 项目专用的原型、设计规范和交付要求继续由该项目维护。
- 当前未完整核实各宿主受管配置、每类 Memory 的实际自动读写、所有规则发现优先级，以及 IDE/App/CLI/云端之间的差异。
- 没有在 Windows 原生或目标 macOS 复现用户报错，不能完成原因归属；没有访问远端 CI 运行历史证明当前所有平台通过。
- 方案形成时没有启动旧工作流或重跑参考项目。历史报告的部分原始产物位于 `/tmp`，其可取得性未验证。

## 15. 首版实施与试点验收

源码版本为 `373b1112c35547ff2f49e4ed1306643a4c38cd73`。当前框架跟踪 19 个文件，退出 2,247 个旧路径。入口、共享正文、禁用隐式调用的元数据、指向和忽略边界均通过检查；`git diff --check` 通过。比对了 17,102 个受保护本地文件的大小、mtime 和 inode，记录一致。旧版分支仍指向 `0bea691fade6bd7dfccbb543dd26dfb14f57d854`，独立 worktree 仍指向 `bb16be211289a16768e6565a0f64c314a6db4076`。

试点使用 Linux x86_64、Node.js 24.19.0 和两个隔离的合成联系人项目。宿主为 Codex 0.165.0 的 `app-server` 和 Claude Code 2.1.329 的 CLI。Claude 未配置独立登录，两个试点使用同一个既有模型代理和 GPT 模型，通过各宿主的协议请求。Claude 的临时配置和会话用于隔离本轮试点。

| 验收项 | 实测结果 |
| --- | --- |
| 发现和共同规则 | 两个宿主均列出 4 个项目 Skill。Codex `thread/start` 报告 `AGENTS.md` 为指令来源。在单独探针副本中，两者都给出只写于 `AGENTS.md` 的应答字词，且没有工具调用，验证普通问答可读取共同规则而无需启动工作流。 |
| 小 Bug | 两个副本从同一基线 `4e34ac8dc78bfe97b150fd9203cacf4e3d7113ce` 起步。两者都读取共享的 `project-fix` 正文，只修改 `web.mjs`，用同一命令将首次及后续选人检查从退出码 1 修复到 0。 |
| 普通跨层开发 | Codex 读取 `project-dev`，实现联系人归档界面反馈、PATCH 路由和临时 JSON 持久化。Codex 的工具沙箱拒绝监听回环端口，交接记录保留实际错误与进程内检查的通过结果。 |
| Codex → Claude 独立测试 | 新会话从 `handoff.md` 接手。Claude 读取 `project-test`，在候选 `b55f60a6bc76379bd42359b36b00bec40b851ede` 使用真实回环 HTTP 验证成功归档、文件落盘、重新读取活跃列表、未知记录失败且反馈不变。2 项通过，工作区清洁。 |
| Claude → Codex 独立审查 | Codex 从 Claude 的交付结果和候选 `01b0e308865bdf58b316cb7d693d31ea7fa8891e` 接手，读取 `project-review` 并复跑通过原测试。独立审查发现空姓名会清除现有标签，证实原验收缺少这条兼容边界。 |
| 审查发现的修复与复验 | 两个宿主在各自副本修复该边界，并复用同一条新回归用例。Claude 副本 `e4ba49c1e2828650d994c8629a134438e55d42dc` 的选人与回退检查全部通过（2 项）。Codex 副本 `3488446e6c34dc0d66fc993ddd55b68465a05c0f` 的选人、回退与真实 HTTP 归档检查全部通过（3 项）。 |

小修复与真实 HTTP 测试沿用合成项目自己的 `node --test --test-isolation=none check.test.mjs` 和 `node --test --test-isolation=none check.test.mjs feature.test.mjs`。复验加跑 `fallback.test.mjs`。表中合成项目的 SHA 属于本地隔离仓库，运行记录以本节为汇总。界面检查用标签对象，覆盖交互逻辑。图形客户端、真实浏览器、Windows、macOS 和 Antigravity 未纳入首版试点。OpenAI 官方 Skill 页面本轮返回 403，Codex 行为结论依据安装版本的真实协议与运行记录；Claude 路径与导入参考官方文档并实跑。

## 证据索引

框架证据以 `0bea691fade6bd7dfccbb543dd26dfb14f57d854` 为基线；未来白名单重建后应从旧版分支或该提交查阅，不要求新版为保留链接继续携带旧运行机制。

- **[F1]** [当前忽略规则](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/.gitignore)，第 2、5、51 行分别涉及 `.claude/`、`AGENTS.md`、`.local/`。
- **[F2]** [需求可观察成功](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/docs/requirements/REQ-template.md) 第 29 行；[原型用户动作](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/docs/design/prototypes/USER-FLOW-template.md) 第 69 行；[规格规划方法](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/skills/specification-planning/SKILL.md) 第 480 行；[旧测试固定要求](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/skills/testing-strategy/SKILL.md) 第 27 行。
- **[F3]** [QA 预载](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/agents/qa.md) 第 6 行；[后端角色限制](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/agents/backend-builder.md) 第 47 行；[S6 Builder 选择](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/cli/s6_prepare.go) 第 140 行。
- **[F4]** [场景产物生成](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/scenario/engine.go) 第 50 行；[语义消费者](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/semantic/contracts.go) 第 92 行；[Hook 日志统计](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/metrics/metrics.go) 第 381 行。
- **[F5]** [10 类 Hook 注册](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/settings.json)；[Worker 重建注册](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/workspace/bootstrap.go) 第 123 行；[迁移强制检查](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/migration/templates.go) 第 157 行。
- **[F6]** [失败事件清理 planning stamp](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/cli/planning_tool_change.go) 第 91 行；[旧文档与 Runtime 权威](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/docs/DOCUMENT-MAP.md) 第 17 行。
- **[F7]** [发布控制实际执行](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/policy/engine.go) 第 1437 行；[受保护命令表](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/docs/control/protected-commands.json)。
- **[F8]** [Go 依赖](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/go.mod)；[多平台构建](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/Makefile) 第 35 行；[发布流程](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/.github/workflows/release.yml) 第 33 行；[平台矩阵](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/.github/workflows/runtime-portability.yml)。
- **[F9]** [发布构建与版本注入](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/packaging/build-release.sh) 第 64 行；[版本识别实现](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/cli/build_info.go) 第 13 行；[CLI 职责入口](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/cli/run.go) 第 114 行。
- **[F10]** [已有 Windows 失败审计记录](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/CI-AUDIT.md) 第 156 行；[PowerShell 启动器](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/tools/loop-harness-launcher.ps1) 第 3 行；[Shell 检查执行](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/integration/check_runner.go) 第 44 行；[非 Linux 隔离检查边界](https://github.com/jarodlaw23/go-system-builder/blob/0bea691fade6bd7dfccbb543dd26dfb14f57d854/internal/workspace/check_other.go) 第 11 行。
- **[H1]** [Claude 包版本](/home/jarod.luo/.nvm/versions/node/v24.19.0/lib/node_modules/@anthropic-ai/claude-code/package.json:3)；[Codex 包版本](/home/jarod.luo/.nvm/versions/node/v24.19.0/lib/node_modules/@openai/codex/package.json:3)。仅静态读取，没有启动程序。
- **[H2]** [Ralph Loop 插件 Hook 注册](/home/jarod.luo/.claude/plugins/cache/claude-plugins-official/ralph-loop/1.0.1/hooks/hooks.json)。与框架 Hook 区分，不在本次清理授权范围内。
- **[R1]** [参考统一实施方案](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/plans/2026-11-24-fx-non-hk-ocr-lpf-implementation.md:5)，另见第 7、28、317、337、436 行；[LPF 原型复审](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/plans/2026-11-24-lpf-prototype-reuse-review.md:3)。
- **[R2]** [原型 README](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/web/prototypes/project-workspace-template/README.md:3)；[原型验收记录](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/design/prototypes/project-workspace-template/checks/verification.md:13)，第 19 行记录边界。
- **[R3]** [注册地与法人签署方案](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/plans/2026-11-19-registration-place-and-corporate-signatures.md:47)；[验证记录](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/plans/2026-11-19-registration-place-and-corporate-signatures-verification.md:14)；[姓名取值实现](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/web/src/view/crm/company/composables/corporateRepresentativeValues.ts:17)；[定向组件测试](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/web/src/view/crm/company/__tests__/corporate-representative.test.ts:106)。
- **[R4]** [转股交付记录](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/reports/2026-11-13-transfer-paid-capital-delivery.md:22)，第 33 行开始列验证，第 43 行区分合成 API 与真实事务。
- **[R5]** [原型层级结果生产者](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/web/prototypes/company-transfer-workspace/fund-intake/verify-scenario-hierarchy.mjs:91)；[交互结果生产者](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/web/prototypes/company-transfer-workspace/fund-intake/verify-scenario-directory.mjs:109)。
- **[R6]** [业务浏览器 runner](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/web/test/browser/transfer-paid-capital/run-cli.mjs:18)，第 47 行组成结果，第 51 行处理失败；[归档结果](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/reports/evidence/transfer-paid-capital-browser-20261113/report.json)；[最终来源清单](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/reports/evidence/transfer-paid-capital-20261113/source-manifest.json)。
- **[R7]** [证据索引](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/reports/evidence/transfer-paid-capital-20261113/README.md:5)；[生成产物忽略规则](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/.gitignore:173)。
- **[R8]** [跨设备交接](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/docs/local-development.md:141)，第 147 行交接版本，第 166 行短模板；第 133 行已有 worktree/服务隔离说明。
- **[R9]** [参考项目旧规则](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/AGENTS.md:176)；[Claude 旧入口](/home/jarod.luo/hkbcs-admin-go/hkbcs-admin-go/CLAUDE.md:9)。
