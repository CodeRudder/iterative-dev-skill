---
name: iterative-dev
description: Iterative development with adversarial testing inner loop. Each round: adversarial test (builder/challenger/judge) → discover → fix → multi-dimensional eval → architecture review → fix → ... cycle until all issues cleared. Use when "iterate, 迭代, adversarial evaluation, architecture review, 对抗性评测, 循环改进, quality loop" mentioned.
---

# Iterative Development Cycle

## Identity

**Role**: You are a senior software engineer who practices disciplined iterative development. Each iteration round contains an **inner fix loop** that cycles until all issues are cleared. Adversarial testing is always the entry point — using the Builder/Challenger/Judge tripartite methodology to maximize issue discovery.

## The Outer Loop: Rounds

```
Round 1 → Round 2 → Round 3 → ... → Round N (done)
```

Each round has the same structure. Rounds repeat until quality targets are met.

## The Inner Loop: Per-Round Sub-Process

Each round follows this **inner loop** that cycles until all issues are fixed:

```
┌──────────────────────────────────────────────────────┐
│  Round N                                             │
│                                                      │
│  1. 对抗性评测 (Adversarial: Builder/Challenger/Judge)│
│         ↓                                            │
│  2. 发现问题 (Discover Issues)                        │
│         ↓                                            │
│  3. 修复问题 (Fix Issues)                             │
│         ↓                                            │
│  4. 多维度评测 (Multi-dimensional Eval)               │
│         ↓                                            │
│  5. 架构审查 (Architecture Review)                    │
│         ↓                                            │
│  6. 修复问题 (Fix Issues)                             │
│         ↓                                            │
│  7. 多维度评测 (Re-eval)                              │
│         ↓                                            │
│  8. ...循环直到所有问题清零                             │
│         ↓                                            │
│  9. 核验环节 (Verification: 独立subagent逐流程核验)    │
│         ↓                                            │
│  10. 文档记录 (Document Round)                        │
└──────────────────────────────────────────────────────┘
```

### Step 1: 对抗性评测 (Adversarial Test — Builder/Challenger/Judge)

**ALWAYS FIRST in every round.** Uses the tripartite adversarial methodology.

#### The Three Roles

| Role | Responsibility | Method |
|------|---------------|--------|
| 🔨 Builder (审核者) | **客观审核**开发计划是否完成、集成测试是否通过且测试是否有效 | 读代码、运行测试、验证测试有效性；产出**客观事实清单** |
| ⚔️ Challenger (挑战者) | **推翻**Builder的结论，找出漏洞/幻觉/无证据/无效证据 | 质疑每个"已实现"的结论，检查测试是否真实覆盖，验证证据链完整性 |
| ⚖️ Judge (裁决者) | **裁决**双方论据，存在疑点时发起二次辩论 | 综合判断，如果疑点未消除则要求二次辩论，否则依据规则裁决 |

#### Builder 职责：客观审核而非辩护

Builder不是"证明"系统已经实现了什么（这会导致确认偏误），而是**客观审核**开发计划的完成状态：

```
Builder 的核心职责：
1. 审核开发计划（rounds/round-N/plan.md）中每个任务的完成状态
2. 客观报告哪些已完成、哪些未完成、哪些部分完成
3. 运行集成测试并报告结果
4. 验证测试的有效性（测试是否真正验证了生产代码路径）
5. 不遗漏、不夸大、不美化——如实报告

Builder 产出客观事实清单，每项必须包含：
1. 计划任务ID：对应 plan.md 中的哪个任务
2. 完成状态：✅完成 / ⬜未完成 / 🔄部分完成
3. 实现证据：哪个文件哪行代码实现了它（如已完成）
4. 测试证据：哪个测试文件验证了它（如已完成）
5. 测试结果：实际运行测试的通过/失败记录
6. 测试有效性评估：测试是否使用了真实子系统（非mock关键路径）
7. 未完成原因：为什么未完成（如适用）

Builder 必须遵循的原则：
- 对未完成的任务诚实报告"未完成"，不回避
- 对测试有效性严格评估：mock了关键依赖的测试标记为"有效性存疑"
- 不遗漏计划中的任何任务，即使未完成也要列出
- 结论基于客观事实，不基于"看起来正确"或"应该实现了"
```

#### Challenger 职责：推翻而非补充

Challenger不是"补充"更多测试，而是**推翻**Builder的结论：

```
Challenger 攻击方向：
1. 漏洞攻击：Builder声称实现了X，但代码中找不到对应实现
2. 幻觉攻击：Builder声称测试通过，但测试实际未运行或mock了关键逻辑
3. 无证据攻击：Builder声称功能完整，但缺少集成测试或E2E测试
4. 无效证据攻击：Builder的测试通过了，但测试本身有bug（如断言错误）
5. 集成断裂攻击：单元测试通过，但系统间集成未验证
6. 流程断裂攻击：短流程测试通过，但完整链路未覆盖

Challenger 的每个质疑必须指出：
- 具体哪个结论有问题
- 为什么这个结论不可信
- 缺少什么证据才能可信
```

#### Judge 职责：裁决而非妥协

Judge不是"各打五十大板"，而是**基于证据裁决**：

```
Judge 裁决流程：
1. 逐条审查Builder的证据链
2. 逐条审查Challenger的质疑
3. 判断每个质疑是否成立
4. 如果存在疑点 → 发起二次辩论（Builder补充证据，Challenger再次攻击）
5. 如果疑点消除 → 裁决通过
6. 如果疑点未消除 → 判定为问题，进入修复流程

Judge 不能因为"看起来合理"就裁决通过，必须基于证据
```

#### Adversarial Attack Dimensions

The Challenger probes these dimensions:

```
1. Boundary attacks:
   - Empty/null/undefined inputs
   - Zero, negative, max values
   - Empty collections, single element
   - Concurrent access

2. Sequence attacks:
   - Actions in unexpected order
   - Rapid repeated actions
   - Interrupt mid-operation
   - Navigate away and back

3. Data attacks:
   - Mismatched data sources (ID format, coordinate system)
   - Duplicate entries
   - Missing required fields
   - Type mismatches

4. State attacks:
   - System in unexpected state
   - Stale references
   - Race conditions
   - Memory leaks / accumulation

5. Integration attacks:
   - Dependency returns wrong type
   - Dependency throws error
   - Event bus message ordering
   - Async operation failures
```

**Execution (文件输出模式):**

```
1. 主会话创建输出目录: docs/iterative-devs/{name}/rounds/round-N/verification/

2. 启动 Builder subagent (run_in_background: true):
   prompt中包含: "先读取本轮计划 (rounds/round-N/plan.md)，对照计划中的每个任务客观审核完成状态"
   prompt中包含: "将完整客观事实清单写入 docs/iterative-devs/{name}/rounds/round-N/verification/builder-manifest.md"
   prompt中包含: "对未完成的任务也要列出，标明原因"
   同时: "完成后只返回一行摘要: 'Builder完成, X个已完成/Y个未完成/Z个部分完成'"

3. 等待Builder完成:
   Read builder-manifest.md 确认输出存在

4. 启动 Challenger subagent (run_in_background: true):
   prompt中包含: "读取 docs/iterative-devs/{name}/rounds/round-N/verification/builder-manifest.md 作为Builder的清单"
   prompt中包含: "将完整攻击报告写入 docs/iterative-devs/{name}/rounds/round-N/verification/challenger-attack.md"
   同时: "完成后只返回一行摘要: 'Challenger完成, 发现X个有效质疑'"

5. 等待Challenger完成:
   Read challenger-attack.md 确认输出存在

6. 启动 Judge subagent (run_in_background: true):
   prompt中包含: "读取 builder-manifest.md 和 challenger-attack.md"
   prompt中包含: "将完整裁决写入 docs/iterative-devs/{name}/rounds/round-N/verification/judge-ruling.md"
   同时: "完成后只返回一行摘要: 'Judge完成, 确认X个P0/Y个P1/Z个P2问题'"

7. 等待Judge完成:
   Read judge-ruling.md 获取问题清单摘要
   → 进入 Step 2 (发现问题)
```

**注意**: Builder/Challenger/Judge 必须串行执行（后者依赖前者的输出文件），不可并行。

### Step 2: 发现问题 (Discover Issues)

Merge adversarial findings with systematic analysis.

```
1. Collect Builder's behavior manifest
2. Collect Challenger's attack results
3. Collect Judge's evaluations
4. Categorize: functional / UX / performance / architecture / data
5. Prioritize: P0 (crash/security) → P1 (broken feature) → P2 (UX/arch) → P3 (minor)
6. Identify root causes — not just symptoms
7. 创建 issues.md（问题追踪清单）→ 写入 docs/iterative-devs/{name}/rounds/round-N/issues.md
   - 每个问题分配唯一 ID（I-01, I-02, ...）
   - 记录: 严重度、类型、来源(B/C/J)、描述、涉及文件、状态
   - issues.md 是本轮唯一的问题追踪源，后续所有修复状态变更都在此更新
```

### Step 3: 修复问题 (Fix Issues)

Fix by priority, one at a time.

```
P0: Security/crash/data loss — fix immediately
P1: Core functionality broken — fix this iteration
P2: UX degradation or architecture violation — fix if time permits
P3: Minor issues — backlog
```

**Rules:**
- Fix ONE issue at a time
- Run tests after each fix to catch regressions
- Use subagents for independent fixes (parallel)
- Never skip verification after a fix
- **P0/P1 应在本轮修复**，正常情况下不允许遗留到下一轮
- **每修复一个问题，即时更新 issues.md 中该问题的状态为 ✅**，不要等到轮次结束

### Step 4: 多维度评测 (Multi-dimensional Evaluation)

After fixing, evaluate from multiple dimensions to verify fixes and catch regressions.

| Dimension | Check | Method |
|-----------|-------|--------|
| 功能正确性 | Does each feature work as specified? | Unit tests + E2E |
| 数据一致性 | Are data sources aligned? | Cross-check data flow |
| UI/UX体验 | Is the experience smooth? | Screenshot + manual check |
| 性能指标 | Render time, frame rate, memory | Profiling |
| 类型安全 | Any `any` escapes? | TypeScript strict check |
| 边界条件 | Empty, max, negative, null | Edge case tests |
| 错误处理 | Graceful failure? | Error injection |

**Execution:**
- Run targeted tests by module: `npx vitest run [相关测试文件/目录]` → record pass/fail
  - **禁止全量测试**（`npm test` / `vitest run` 无参数），全量测试消耗过多CPU和时间
  - 只运行本轮修改涉及的模块测试（如 `npx vitest run src/games/three-kingdoms/engine/map/`）
  - 修复后回归验证也只运行受影响模块的测试
  - 全量测试仅在最终验收时执行一次
- Take screenshots of key UI states
- Check for console errors/warnings
- Verify data flow consistency

### Step 5: 架构审查 (Architecture Review)

Check structural integrity after changes.

```
1. Dependency direction — correct layering?
2. Layer violations — are abstractions respected?
3. God components — any file doing too much?
4. Type safety — any `any` type escapes?
5. Event bus consistency — are all systems using the same bus?
6. Data flow — single source of truth?
7. Code duplication — can anything be consolidated?
8. Dead code — anything unused after changes?
```

### Step 6: 修复问题 (Fix Issues Again)

Fix any new issues discovered in Steps 4-5. Same rules as Step 3.

### Step 7: 多维度评测 (Re-evaluate)

Re-run evaluation to verify Step 6 fixes didn't create new problems.

### Step 8: 循环检查 (Loop Check)

**Are all issues cleared?**

```
IF P0 > 0 OR P1 > 0:
  → Go back to Step 3 (fix remaining issues)
  → Then Step 4 (re-evaluate)
  → Then Step 5 (architecture review)
  → Then Step 6 (fix again)
  → Loop until P0=0 AND P1=0

IF P0=0 AND P1=0:
  → Proceed to Step 9 (核验环节)
```

**问题传递规则 (CRITICAL):**

所有未修复问题必须传递到下轮，**禁止丢失**：
P0/P1 正常情况下应在本轮修复，不限制传递但应尽力避免。

```
1. P0/P1 处理规则:
   - 内循环 (Step 3-7) 持续到 P0=0 AND P1=0 才能退出
   - 每修复一个问题，即时更新 issues.md 状态为 ✅
   - 正常情况下 P0/P1 应在本轮修复完成
   - 如传递到下轮，需标注原因

2. P2/P3 传递规则:
   - ✅ 已修复 (本轮内关闭)
   - ➡️ 传递下轮 (写入 PROGRESS.md 待修复项 + 下轮 plan.md)

3. 禁止出现以下情况:
   - ❌ 问题出现在 issues.md 但 report.md "剩余问题"中找不到
   - ❌ report.md "剩余问题"中有条目但 PROGRESS.md "待修复项"中没有
   - ❌ PROGRESS.md "待修复项"中有条目但下轮 plan.md 中没有对应任务

4. 传递链完整性校验 (Step 10 文档记录时执行):
   issues.md 全部问题 → 逐条核对 → report.md "剩余问题" → PROGRESS.md "待修复项" → 下轮 plan.md "遗留任务"
   任一环节缺失 = 校验失败，必须补齐
```

### Step 9: 核验环节 (Verification Phase)

**开发完成后必须进入核验环节**，使用对抗性三角色进行深度核验，不能只是"运行测试看是否通过"。

#### 核验原则

```
1. 对抗性: 使用Builder/Challenger/Judge三角色，不是简单的测试执行
2. 独立性: 每个角色使用独立subagent，避免上下文污染
3. 完整性: 必须覆盖100%功能点和流程链路
4. 真实性: 必须真实执行测试并验证结果，不能假设通过
5. 深度性: 不只是"测试通过"，要验证测试本身是否有效
6. 从零核验: 必须忽略之前所有轮次的结果和结论，重新验证PLAN.md中的全部功能点
   - 不引用之前Builder/Challenger/Judge的输出
   - 不假设任何功能"已经验证过"
   - 每次核验都是全新的、独立的验证
   - 核验范围 = PLAN.md 中的全部功能点（A~Z所有系列），不是"本轮新增"
```

#### 核验流程

```
1. 梳理所有功能点和流程链路
   - 从 PLAN.md 提取所有功能点
   - 梳理完整流程链路（从入口到出口）
   - 识别关键路径和分支路径

2. 对抗性核验（每个流程独立执行）
   - Builder: 证明该流程已实现且测试通过
   - Challenger: 推翻Builder的结论，找漏洞/幻觉/无效证据
   - Judge: 裁决，存在疑点则发起二次辩论

3. 核验结果汇总
   - 统计通过/失败/未覆盖的流程
   - 识别断链的流程（缺少测试或实现）
   - 生成核验报告
```

#### 完整流程链路测试要求

**不只是短流程集成测试，必须覆盖完整链路:**

```
短流程(不够): A → B → C
完整链路(必须): 入口 → A → B → C → D → ... → 出口
```

#### 核验任务模板（对抗性三角色 — 文件输出模式）

**每个流程的核验必须使用三个独立subagent，结果写入文件：**

```
═══════════════════════════════════════════════════════
任务1: Builder — 客观审核 [流程名称] 的完成状态
═══════════════════════════════════════════════════════

你是Builder角色（客观审核者）。你的任务是**客观审核**以下流程的开发完成状态，而非辩护或证明。

**先读取本轮计划**: {round_plan_path}

流程: [完整链路描述]

请执行以下步骤，如实报告：

1. 计划完成状态
   - 对照 plan.md 中的每个任务，逐一报告完成状态
   - ✅完成：代码已实现 + 测试通过 + 测试有效
   - ⬜未完成：代码未实现或测试未通过
   - 🔄部分完成：代码实现但测试缺失/无效
   - 对未完成的任务诚实说明原因

2. 实现证据（仅对已完成的任务）
   - 指出每个功能点的具体实现位置（文件:行号）
   - 确认代码逻辑覆盖了所有场景

3. 测试证据
   - 运行相关测试（按模块，禁止全量）: `npx vitest run [相关测试文件或目录]`
   - 记录每个测试的通过/失败状态
   - 确认测试覆盖了正常/异常/边界场景

4. 测试有效性评估（关键步骤）
   - 确认测试不是mock了关键依赖 → 标记为"有效"
   - 如果mock了关键系统核心路径 → 标记为"有效性存疑"并说明
   - 确认测试验证了系统间的真实交互
   - 确认完整链路有端到端测试

5. 产出客观事实清单
   每项格式:
   | 计划任务ID | 完成状态 | 实现位置 | 测试文件 | 测试结果 | 测试有效性 | 覆盖场景 |
   未完成任务也必须列出，标明"未完成"及原因。

**将完整结果写入文件: {output_path}/builder-manifest.md**
完成后只返回一行摘要: "Builder完成, X个已完成/Y个未完成/Z个部分完成, A个测试有效性存疑"
```

```
═══════════════════════════════════════════════════════
任务2: Challenger — 推翻Builder的结论
═══════════════════════════════════════════════════════

你是Challenger角色。你的任务是**推翻**Builder的结论。

**先读取Builder的行为清单: {output_path}/builder-manifest.md**

请从以下方向攻击：

1. 漏洞攻击
   - Builder声称实现了X，代码中是否真的存在？
   - 读取Builder引用的源代码，验证实现是否完整

2. 幻觉攻击
   - Builder声称测试通过，测试是否真的运行了？
   - 测试是否mock了关键逻辑，导致测试无效？
   - 测试断言是否正确（如expect(true).toBe(true)）？

3. 无证据攻击
   - 哪些功能点缺少集成测试？
   - 哪些流程链路只有短流程测试？
   - 哪些边界场景未被覆盖？

4. 集成断裂攻击
   - 单元测试通过，但系统间集成是否验证？
   - 测试中的依赖是否真实（非mock）？

5. 流程断裂攻击
   - 完整链路是否覆盖（入口→...→出口）？
   - 还是只测了中间某一段？

每个质疑格式:
| 质疑点 | Builder的结论 | 为什么不可信 | 缺少什么证据 |

**将完整攻击报告写入文件: {output_path}/challenger-attack.md**
完成后只返回一行摘要: "Challenger完成, X个有效质疑(其中P0:Y, P1:Z)"
```

```
═══════════════════════════════════════════════════════
任务3: Judge — 裁决双方论据
═══════════════════════════════════════════════════════

你是Judge角色。你的任务是**裁决**Builder和Challenger的论据。

**先读取两份报告:**
- Builder行为清单: {output_path}/builder-manifest.md
- Challenger攻击报告: {output_path}/challenger-attack.md

请执行以下步骤：

1. 逐条审查Builder的证据链
   - 每个功能点的证据是否充分？
   - 测试是否真实有效？

2. 逐条审查Challenger的质疑
   - 每个质疑是否成立？
   - 质疑的证据是否可靠？

3. 裁决
   - 如果Challenger的质疑全部不成立 → 裁决通过
   - 如果存在疑点 → 发起二次辩论
   - 如果疑点未消除 → 判定为问题

4. 二次辩论（如有疑点）
   - 要求Builder补充证据
   - 要求Challenger再次攻击
   - 直到疑点消除或确认为问题

**将完整裁决报告写入文件: {output_path}/judge-ruling.md**
完成后只返回一行摘要: "Judge完成, 确认P0:X, P1:Y, P2:Z个问题"
输出格式:
| 质疑点 | Challenger观点 | Builder补充 | Judge裁决 | 理由 |
```

**主会话编排流程:**

```
1. 创建输出目录: mkdir -p docs/iterative-devs/{name}/rounds/round-N/verification/

2. 启动Builder subagent (run_in_background: true)
   → 等待完成 → 确认 builder-manifest.md 已生成

3. 启动Challenger subagent (run_in_background: true)
   prompt中指定: "先读取 {path}/builder-manifest.md"
   → 等待完成 → 确认 challenger-attack.md 已生成

4. 启动Judge subagent (run_in_background: true)
   prompt中指定: "先读取 {path}/builder-manifest.md 和 {path}/challenger-attack.md"
   → 等待完成 → 确认 judge-ruling.md 已生成

5. 主会话读取 judge-ruling.md 的结论部分（只读摘要，不读全文）
   → 汇总问题清单 → 进入修复流程或报告给用户
```

#### 核验完成标准

```
✅ 所有功能点有对应测试
✅ 所有测试通过
✅ 所有流程链路有完整测试（非短流程）
✅ 核验结果真实（实际运行验证）
✅ Builder的每个结论有证据支撑
✅ Challenger的每个质疑已裁决
✅ 问题清单已记录并分类
```

### Step 10: 文档记录 (Document Round)

Create iteration report and plan next round. All docs go to `docs/iterative-devs/{name}/rounds/round-N/`.

**必做事项:**
1. 生成本轮报告 → `docs/iterative-devs/{name}/rounds/round-N/report.md`
2. **更新进度文档** → `docs/iterative-devs/{name}/PROGRESS.md`
   - 追加本轮摘要行到轮次记录表
   - 更新质量指标当前值
   - **同步待修复项**: 将本轮未修复问题追加到"待修复项"表，已修复的标记为 ✅
   - 更新当前阶段和下一步
3. **生成下轮计划** → `docs/iterative-devs/{name}/rounds/round-N+1/plan.md`
   - 从 `PLAN.md`(总计划) 拆解下轮焦点
   - **必须包含 PROGRESS.md 中所有状态为 🔄 的待修复项**，逐条列为"遗留任务"
   - 每3轮还需合并复盘改进措施
   - 包含：焦点领域、对抗性评测重点、质量目标
4. **每3轮进行复盘**（当 N % 3 == 0 时）→ 写入 `report.md` Section 9
   - 3轮趋势分析（问题发现/修复趋势、质量指标变化）
   - 流程改进点（哪里做得好、哪里可以改进）
   - 工具/方法改进建议
   - 改进措施列入下轮 `plan.md`

```markdown
# Round N 迭代报告

> **日期**: YYYY-MM-DD
> **迭代周期**: 第N轮 — [主题]

## 1. 对抗性评测发现
### Builder 客观事实清单
| ID | 计划任务 | 完成状态 | 测试结果 | 测试有效性 |
|----|---------|:--------:|---------|:---------:|
| B-01 | ... | ✅/⬜/🔄 | pass/fail | 有效/存疑 |

### Challenger 攻击结果
| ID | 攻击维度 | 攻击方式 | 结果 | Judge判定 |
|----|---------|---------|------|----------|
| C-01 | 边界 | ... | 崩溃 | P0 确认 |
| C-02 | 数据 | ... | 显示异常 | P1 确认 |
| C-03 | 状态 | ... | 无影响 | P3 否决 |

### Judge 综合评定
| ID | 严重度 | 可复现 | 根因 | 建议 |
|----|:------:|:------:|------|------|
| J-01 | P0 | 是 | ... | ... |

## 2. 修复内容
| ID | 对应问题 | 文件 | 修复方式 | 影响 |
|----|---------|------|---------|------|
| F-01 | C-01 | ... | ... | ... |

## 3. 内部循环记录
| 轮次 | 发现 | 修复 | 剩余P0 | 剩余P1 |
|------|:----:|:----:|:------:|:------:|
| 1.1 | 5 | 3 | 2 | 0 |
| 1.2 | 2 | 2 | 0 | 0 |
| **合计** | **7** | **5** | **0** | **0** |

## 4. 测试结果
| 测试套件 | 通过 | 失败 |
|----------|:----:|:----:|
| [suite] | N | N |
| **总计** | **N** | **N** |

## 5. 架构审查结果
| 检查项 | 状态 | 问题 |
|--------|:----:|------|
| 依赖方向 | ✅/❌ | ... |
| 层级边界 | ✅/❌ | ... |
| 类型安全 | ✅/❌ | ... |

## 6. 回顾(跨轮)
| 指标 | R1 | R2 | ... | RN | 趋势 |
|------|:--:|:--:|:---:|:--:|:----:|
| 测试通过率 | | | | | ↑/↓ |
| P0问题 | | | | | ↑/↓ |
| 对抗性发现 | | | | | ↑/↓ |
| 内部循环次数 | | | | | ↑/↓ |
| 架构问题 | | | | | ↑/↓ |

## 7. 剩余问题(下轮)
| ID | 问题 | 优先级 | 来源 |
|----|------|:------:|------|

## 8. 下轮计划
> 详见 `docs/iterative-devs/{name}/rounds/round-N+1/plan.md`

## 9. 复盘（每3轮，当 N % 3 == 0 时）

> 仅在第3、6、9...轮时填写

### 9.1 趋势分析（近3轮）
| 指标 | R(N-2) | R(N-1) | R(N) | 趋势 | 分析 |
|------|:------:|:------:|:----:|:----:|------|
| 对抗性发现 | | | | ↑/↓/→ | ... |
| 修复数 | | | | | ... |
| P0 | | | | | ... |
| P1 | | | | | ... |
| 内部循环次数 | | | | | ... |
| 测试通过率 | | | | | ... |

### 9.2 流程改进
| 项目 | 做得好 | 可改进 | 改进措施 |
|------|--------|--------|----------|
| 对抗性评测 | ... | ... | ... |
| 修复效率 | ... | ... | ... |
| 架构审查 | ... | ... | ... |
| 文档质量 | ... | ... | ... |

### 9.3 工具/方法改进
| 改进项 | 当前方式 | 建议方式 | 预期效果 |
|--------|---------|---------|----------|
| ... | ... | ... | ... |

### 9.4 改进措施（列入下轮计划）
| ID | 改进措施 | 负责 | 验收标准 |
|----|---------|------|---------|
| IMP-01 | ... | ... | ... |
```

## Documentation System

### Doc Types and Save Locations

| 层级 | Doc Type | Filename | Save Path | When |
|:----:|----------|----------|-----------|------|
| 全局 | 迭代总计划 | `PLAN.md` | `docs/iterative-devs/{name}/` | 迭代开始前，贯穿全局 |
| 全局 | 迭代进度 | `PROGRESS.md` | `docs/iterative-devs/{name}/` | 每轮结束时更新 |
| 全局 | 业务流程进度 | `FLOW-PROCESS.md` | `docs/iterative-devs/{name}/` | 迭代开始时创建，开发过程中实时更新 |
| 轮次 | 轮次计划 | `rounds/round-N/plan.md` | `docs/iterative-devs/{name}/rounds/round-N/` | 每轮开始前（上轮结束时生成） |
| 轮次 | 问题追踪 | `rounds/round-N/issues.md` | `docs/iterative-devs/{name}/rounds/round-N/` | 对抗性评测后（Step 2 创建） |
| 轮次 | 轮次报告 | `rounds/round-N/report.md` | `docs/iterative-devs/{name}/rounds/round-N/` | 每轮结束（Step 10 生成） |
| 轮次 | 架构审查 | `rounds/round-N/arch-review.md` | `docs/iterative-devs/{name}/rounds/round-N/` | 架构审查后 |
| 轮次 | 行为清单 | `rounds/round-N/behavior-manifest.md` | `docs/iterative-devs/{name}/rounds/round-N/` | Builder输出 |
| 全局 | 迭代总结 | `SUMMARY.md` | `docs/iterative-devs/{name}/` | 迭代完成时 |

### Directory Structure

项目按**迭代主题**组织，每个迭代有独立计划和多轮子流程：

```
docs/
└── iterative-devs/
    ├── {iteration-name}/              # 迭代主题，如 map-system, combat-system
    │   ├── PLAN.md                    # 迭代总计划(贯穿全局，迭代开始前写)
    │   ├── PROGRESS.md                # 迭代进度(每轮结束更新，快速了解全局状态)
    │   ├── FLOW-PROCESS.md            # 业务流程进度(面向用户，流程完成情况)
    │   ├── rounds/                    # 每轮过程文档
    │   │   ├── round-1/               # 第1轮
    │   │   │   ├── plan.md           # 本轮计划(第1轮从PLAN.md拆解)
    │   │   │   ├── report.md
    │   │   │   ├── issues.md
    │   │   │   ├── arch-review.md
    │   │   │   └── behavior-manifest.md
    │   │   ├── round-2/               # 第2轮
    │   │   │   ├── plan.md           # 本轮计划(上轮结束时生成)
    │   │   │   └── ...
    │   │   └── round-3/               # 第3轮 (含复盘)
    │   │       ├── plan.md
    │   │       └── ...
    │   └── SUMMARY.md                 # 迭代完成总结
    ├── {another-iteration}/
    │   ├── PLAN.md
    │   ├── rounds/
    │   │   └── round-1/
    │   └── SUMMARY.md
    └── ...
```

**两层计划关系**:
- `PLAN.md` = 迭代总计划，定义整个迭代的目标、范围、验收标准
- `rounds/round-N/plan.md` = 本轮计划，从总计划拆解 + 上轮遗留问题 + 复盘改进措施

**流程**: `PLAN.md`(总) → `rounds/round-1/plan.md`(拆解) → 执行 → `rounds/round-2/plan.md`(上轮生成) → ... → `SUMMARY.md`

**命名规则**:
- 迭代目录: `docs/iterative-devs/{kebab-case-name}/` — 按功能模块命名
- 每轮子目录: `rounds/round-{N}/` — 轮次编号
- 文件名统一不带轮次前缀（目录已包含）

### Template: `docs/iterative-devs/{name}/PLAN.md`

> 进度追踪见 `PROGRESS.md`，每轮结束时自动更新，不在 PLAN.md 中重复记录。
>
> 模板文件: `references/tmpl-plan.md`

### Template: `docs/iterative-devs/{name}/PROGRESS.md`

> 迭代开始时创建，每轮结束时更新。主会话恢复上下文时优先读取此文件。
>
> 模板文件: `references/tmpl-progress.md`

### Template: `docs/iterative-devs/{name}/rounds/round-N/plan.md`

> 轮次计划：从总计划(PLAN.md)拆解 + 上轮遗留 + 复盘改进
>
> 模板文件: `references/tmpl-round-plan.md`

### Template: `docs/iterative-devs/{name}/FLOW-PROCESS.md`

> **业务流程进度**（面向用户）。来自业务流程文档，展示业务流程的完成情况。
> 每个业务流程/子流程一行。每个子流程开始及完成时即时更新状态和完成度。
>
> 模板文件: `references/tmpl-FLOW-PROCESS.md`

### Template: `docs/iterative-devs/{name}/SUMMARY.md`

> 模板文件: `references/tmpl-summary.md`

### Template: `docs/iterative-devs/{name}/rounds/round-N/issues.md`

> **问题追踪源文件**。对抗性评测后（Step 2）创建，是本轮唯一的问题追踪源。
> 修复过程中实时更新每个问题的状态（✅/➡️），每修复一个即时标记。
> 轮次结束时，report.md "剩余问题" 从此文件派生（取状态为 ➡️ 的项）。
>
> 模板文件: `references/tmpl-issues.md`

### Template: `docs/iterative-devs/{name}/rounds/round-N/report.md`

> **轮次汇总报告**。每轮结束时（Step 10）生成，是 issues.md 的下游文档。
> Section 1 "对抗性评测发现" 汇总 Builder/Challenger/Judge 的原始结果。
> Section 7 "剩余问题" 从 issues.md 中状态为 ➡️ 的问题派生，**不是独立记录**。
> 传递链: issues.md(源) → report.md "剩余问题" → PROGRESS.md 待修复项 → 下轮 plan.md 遗留任务
>
> 模板文件: `references/tmpl-report.md`

### Template: `docs/iterative-devs/{name}/rounds/round-N/arch-review.md`

> 模板文件: `references/tmpl-arch-review.md`


## Execution Model

### Use Subagents

**CRITICAL**: Main session orchestrates only. All work via subagents.

```
Main session: Task dispatch, status tracking, user communication, reading result files for summary
Subagents:   Code changes, test runs, reviews, screenshots, adversarial roles
```

### File-Based Output Protocol (CRITICAL)

**ALL subagent results MUST be written to files, NOT returned to the main session.**

Large outputs returned to the main session cause context pollution, making subsequent rounds unreliable. Every subagent that produces non-trivial output must write it to a designated file.

```
规则:
1. 每个子任务必须将结果写入指定文件路径
2. 主会话通过 Read 工具读取文件摘要，不接收完整返回
3. 子任务使用 run_in_background: true 运行，避免阻塞
4. 主会话通过 TaskOutput (block=false) 或 Read 工具检查进度和结果

违反此规则的后果:
- 主会话上下文被大量子任务输出填满
- 后续轮次质量严重下降（上下文不足）
- Builder/Challenger/Judge 三方辩论无法有效进行
```

#### Output File Naming Convention

```
docs/iterative-devs/{name}/rounds/round-N/
├── verification/                    # 核验输出目录
│   ├── builder-manifest.md         # Builder 客观事实清单
│   ├── challenger-attack.md        # Challenger 攻击报告
│   ├── judge-ruling.md             # Judge 裁决报告
│   ├── builder-round2.md           # 二次辩论 Builder（如有）
│   ├── challenger-round2.md        # 二次辩论 Challenger（如有）
│   └── verification-summary.md     # 核验汇总（主会话生成）
├── arch-review.md                   # 架构审查
├── issues.md                        # 问题清单
├── report.md                        # 轮次报告
└── plan.md                          # 轮次计划
```

#### Subagent Prompt Template (MUST follow)

```
每个子任务的 prompt 必须包含以下指令:

"将你的完整结果写入文件: {output_file_path}
使用 markdown 格式。不要将完整结果返回给调用方，只返回一行摘要。"
```

### Subagent Execution Patterns

#### Pattern 1: Sequential Dependency Chain (Builder → Challenger → Judge)

```
# Step 1: Builder (后台运行，输出到文件)
Task(B builder): 产出行为清单 → 写入 builder-manifest.md

# Step 2: 等待Builder完成
TaskOutput(builder, block=true) 或 Read builder-manifest.md

# Step 3: Challenger (后台运行，读取Builder输出文件)
Task(B challenger): 读取 builder-manifest.md → 攻击 → 写入 challenger-attack.md

# Step 4: 等待Challenger完成
TaskOutput(challenger, block=true) 或 Read challenger-attack.md

# Step 5: Judge (后台运行，读取Builder+Challenger输出文件)
Task(B judge): 读取 builder-manifest.md + challenger-attack.md → 裁决 → 写入 judge-ruling.md

# Step 6: 等待Judge完成，读取摘要
Read judge-ruling.md → 向用户报告结论
```

#### Pattern 2: Parallel Independent Tasks

```
# 并行启动多个独立修复任务（每个输出到文件）
Task(A fix-p0): 修复问题I-01 → 写入 fixes/I-01.md
Task(B fix-p1): 修复问题I-02 → 写入 fixes/I-02.md
Task(C run-test): 运行测试 → 写入 test-results.md

# 等待所有完成
TaskOutput(A, block=true)
TaskOutput(B, block=true)
TaskOutput(C, block=true)
```

#### Pattern 3: Verification Phase (Step 9)

```
# 按功能域分批核验，每批3角色串行，批间可并行
# 批次1: 编队系统 + 伤亡系统
Task(B builder-batch1): 验证G+H域 → 写入 verification/builder-GH.md
# 等待...
Task(B challenger-batch1): 攻击G+H → 写入 verification/challenger-GH.md
# 等待...
Task(B judge-batch1): 裁决G+H → 写入 verification/judge-GH.md

# 批次2: 地图+行军（与批次1的Judge可并行）
Task(B builder-batch2): 验证A+B域 → 写入 verification/builder-AB.md
...
```

### Main Session Responsibilities (ONLY)

```
核心原则: 主会话仅负责任务编排，所有实际工作（代码读写、测试运行、修复实现）
           通过 subagent 执行。主会话不直接动手，只做调度。

主会话只负责:
1. 创建任务列表 (TaskCreate)
2. 分发子任务 (Task with subagent)
3. 跟踪任务状态 (TaskUpdate/TaskList)
4. 读取结果文件摘要 (Read tool, 只读关键结论)
5. 向用户汇报进展
6. 生成汇总文档 (verification-summary.md, report.md)
7. 更新进度文档 (PROGRESS.md, FLOW-PROCESS.md)
8. 编排下一轮流程
9. 子任务未完成时，继续编排新的 subagent 接着处理，直到本轮问题全部修复

子任务失败/未完成时的处理:
- subagent 返回结果不完整 → 读取其输出文件，分析缺口，编排新 subagent 补齐
- subagent 修复不彻底 → 编排新 subagent 针对剩余问题继续修复
- 循环编排直到 issues.md 中所有 P0/P1 状态为 ✅ 或 ➡️

主会话禁止:
- 直接运行测试（交给subagent）
- 直接读取大量源代码（交给subagent）
- 直接修复代码（交给subagent）
- 将subagent的完整返回纳入上下文
- 因"子任务失败"而放弃问题 — 必须继续编排直到完成
```

## Metrics to Track

| Metric | Description | Target |
|--------|-------------|--------|
| Test pass rate | Tests passing / total | 100% |
| P0 issues | Critical bugs remaining | 0 |
| P1 issues | Important bugs remaining | 0 |
| Adversarial findings | Issues found per round | Track trend |
| Inner loop count | Fix cycles per round | Decreasing |
| Architecture violations | Layer/deps violations | 0 |
| `any` type escapes | Type safety violations | 0 |
| Feature completion | Features implemented / planned | 100% |
| Flow coverage | Complete flow chains tested / total | 100% |
| Verification pass rate | Flows verified by subagents / total | 100% |

## Anti-Patterns

1. **Fixing before adversarial testing** — Always test first
2. **Skipping Builder's manifest** — Need clear expectations before attacking
3. **Challenger without evidence** — Every finding needs reproducible steps
4. **Judge too lenient** — Don't dismiss findings without investigation
5. **Skipping inner loop** — Don't stop until P0=0 AND P1=0
6. **Fixing multiple issues at once** — One at a time
7. **Skipping re-evaluation** — Every fix needs verification
8. **Running everything in main session** — Use subagents
9. **Only short flow tests** — Must test complete flow chains, not just A→B→C
10. **Skipping verification phase** — Must use adversarial 3-role verification
11. **Assuming tests pass** — Must actually run tests and verify results, never assume
12. **Incomplete coverage** — Must cover 100% of features and flow paths
13. **Builder as defender** — Builder must objectively audit plan completion status, not defend/prove implementation. Report unfinished tasks honestly
14. **Challenger as complimenter** — Challenger must try to disprove, not add more tests
15. **Judge as compromiser** — Judge must decide based on evidence, not split the difference
16. **Verification as test execution** — Verification is adversarial debate, not running tests
17. **Returning subagent output to main session** — Subagent results MUST go to files, main session only reads summaries
18. **Running Builder/Challenger/Judge in parallel** — They are sequential: Builder output file → Challenger reads it → Judge reads both
19. **Main session doing real work** — Main session orchestrates only; code reading, test running, and fixing are subagent tasks
20. **Running full test suite during iteration** — Only run targeted module tests during inner loop; full suite reserved for final acceptance only
21. **Dropping issues between rounds** — Every issue must have a final state (✅/➡️). Issues in issues.md must appear in report.md, PROGRESS.md, and next round's plan.md. Break in the chain = lost issue
22. **Writing next plan without reading PROGRESS.md** — Must read PROGRESS.md "待修复项" before writing next round's plan; all 🔄 items must appear as "遗留任务"

## Trigger Phrases

- "迭代" / "iterate"
- "对抗性评测" / "adversarial evaluation"
- "架构审查" / "architecture review"
- "循环改进" / "iterative improvement"
- "继续迭代" / "continue iterating"
- "builder challenger judge"
- "多维度评测" / "multi-dimensional evaluation"
