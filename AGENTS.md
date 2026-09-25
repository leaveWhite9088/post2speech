# AGENTS.md — AI 迭代守则

本仓库迭代一个口语化改写 Skill：`SKILL.md` 是入口，规则主体是 `references/core.md`（通用，所有模型共享）；某模型相对 core 的实测差异写在 `references/overlays/<model>.md`；2026-09-16 之前的分模型规则文件保留在 `references/legacy/`，不再迭代。任何 AI 参与本仓库的工作时，必须遵守以下守则。方法论详见 `lab/docs/methodology.md`，本文件是唯一的行为准则来源。

## 仓库分区（先分清再动手）

- **运行时区**（装 skill 后被加载，改动必须走完整归因+回归流程）：`SKILL.md`、`references/`、`profiles/`。
- **治理区 `lab/`**（迭代的日常工作区）：`calibration/`、`corpus/`、`regression/`、`ab-tests/`、`docs/`、`demo/`、`scripts/`，全部在 `lab/` 下。
- **门面与规矩**（根目录）：`README.md`、`README_EN.md`、`AGENTS.md`、`CHANGELOG.md`。
- 2026-09-17 起治理目录迁入 `lab/`；此前归档记录中的旧路径（`calibration/`、`corpus/`、`regression/` 等无前缀形式）一律对应 `lab/` 下同名目录。

## 核心铁律

1. **ASR 转写稿是唯一的真理来源。** 规则只能来自"AI 输出 vs 真人朗读（ASR 转写）"的真实差异。禁止凭空想象"人大概会这么说"然后加规则。
2. **归因，不改个案。** 从差异中提炼的是"这类结构如何处理"的通用规律，禁止只把单个句子改顺就完事。
3. **每条规则必须可追溯。** 修改 `references/` 下任一规则文件（core.md 或 overlay）的同一轮，必须在 `CHANGELOG.md` 追加归因日志（差异类型 → 规则改动 → 来源校准记录编号），条目中注明校准时用的模型名。没有日志的规则改动视为无效。
4. **改完必须回归。** 每次修改 `references/` 下任一规则文件后，必须用触发本次校准的那个模型：
   - 用 `lab/corpus/` 全部输入跑一遍，检查各题材无退化；
   - 用 `lab/regression/` 全部负样本跑一遍，检查旧问题无复发。
5. **bad case 必须入库。** 迭代中发现的任何"不口语"输出，必须按 `lab/regression/TEMPLATE.md` 的格式存入负样本库，不允许"这次记住就好"。

## 目录使用规则

| 场景 | 写到哪 | 格式 |
|------|--------|------|
| 提炼出的改写规则 | `references/core.md`（模型特异性、有校准佐证的差异 → `references/overlays/<model>.md`） | 按现有分节结构追加/修改，默认只动 core.md |
| 规则归因日志 | `CHANGELOG.md` | 按文件头部的条目格式，新条目在最上方 |
| 每轮校准的完整记录 | `lab/calibration/YYYYMMDD-NN.md` | 按 `lab/calibration/TEMPLATE.md` |
| 负样本（bad case） | `lab/regression/cases/NNN-简短描述.md` | 按 `lab/regression/TEMPLATE.md` |
| 回归结果归档 | `lab/regression/results/YYYYMMDD-模型-版本.md` | 按 `lab/regression/results/TEMPLATE.md`，执行规程见 `lab/regression/RUNBOOK.md` |
| 差异率汇总 | `lab/calibration/metrics.md` | 追加一行，数值与当轮校准记录一致 |
| A/B 盲测记录 | `lab/ab-tests/YYYYMMDD-NN.md` | 按 `lab/ab-tests/TEMPLATE.md`，执行规程见 `lab/ab-tests/RUNBOOK.md` |
| A/B 盲测汇总 | `lab/ab-tests/summary.md` | 追加一行，累计胜率看趋势 |
| 新的测试输入 | `lab/corpus/` 对应题材文件 | 追加到文件末尾，标注来源 |

- 编号 `NN`/`NNN` 从现有最大编号递增，不允许跳号或复用。
- 多会话并行时的编号分段约定（2026-09-11 起）：校准记录 `cal-YYYYMMDD-NN` 按模型分奇偶——**Kimi K3 用奇号，GLM 5.3 用偶号**，各自按号段递增（号段内跳号属正常，因另一模型占用了中间的号）。该约定只约束新增记录，已归档记录（含历史上两模型同号的 -05、-06）不改号。后续新增模型在 `references/overlays/` 下登记时，同时在本条登记其号段。
- 禁止修改已归档的校准记录和负样本（它们是历史事实）；发现错误时在 `CHANGELOG.md` 里说明，而不是改历史。

## 规则写作要求（改 `references/` 下的规则文件时）

- 规则要写成**可判定的指令**（"遇到 X 结构时做 Y"），不要写成模糊的期望（"尽量自然一点"）。
- 条目按固定字段格式书写（core v1.5 起，对齐 humanizer 的工程化结构）：`### N. 短名` + 至多三个字段——`**触发：**`（何时命中，含黑名单词表，无触发形式的条目省略）、`**处理：**`（怎么改；例外、限量、判据都揉进这段散文，不单列字段）、`**例：**`（反例 → 正例，需要时）；每条规则尽量附带一个正例或反例，例子必须来自真实校准记录。
- **溯源信息一律不写进规则文件**（校准编号、负样本编号、用户判据引语），它们占用模型上下文；溯源统一由 `CHANGELOG.md` 的归因日志承担。
- 通用规则写进主体；因人而异的分歧写法收进「风格开关」一节，默认值明确标注。
- 发现某条规则连续多轮从未被触发，在 `CHANGELOG.md` 提议删除并说明理由，由人确认后再删。

## 迭代流程（AI 侧职责）

1. 加载 `references/core.md`（当前模型在 `references/overlays/` 下有实测分歧条目时叠加对应 overlay），用它改写指定输入，输出版本号（模型名 + 规则文件版本 + 日期）。
2. 拿到 ASR 转写版后，逐条 diff，按停顿/语气/标点/冗余词/即兴改写五类归因。
3. 完整记录写入 `lab/calibration/`，提炼的规则改动写入 `references/core.md`（仅当有证据表明是该模型特有、且与 core 冲突时写入对应 overlay），归因写入 `CHANGELOG.md`（注明校准时用的模型名）。
4. 本轮新发现的 bad case 存入 `lab/regression/`。
5. 用当前模型跑 `lab/corpus/` + `lab/regression/` 回归（步骤和判定标准见 `lab/regression/RUNBOOK.md`），结果按 `lab/regression/results/TEMPLATE.md` 归档；有退化就先修复再收尾。
6. 统计本轮差异率（改动处数量 / 总句数），追加到校准记录中，并在 `lab/calibration/metrics.md` 汇总表加一行。

## 禁止事项

- 禁止在没有真实校准数据的情况下新增规则。
- 禁止跳过回归直接宣称"已收敛"。
- 禁止为了让 diff 变少而改 ASR 转写稿或测试输入。
- 禁止把规则文件无限加长：新增规则时检查是否能并入已有规则；overlay 只写与 core 的实测分歧，几行以内。
- 禁止把单一模型上的校准结论直接定性为"模型特性"：规则对齐的是人怎么说话，默认进 core.md；只有换模型复现失败、确认是模型特异性行为时，才移入 overlay。
