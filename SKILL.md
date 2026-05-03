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
| 🔨 Builder (构建者) | Reviews what was built, documents intended behavior and assumptions | Read code, specs, tests; produce a "behavior manifest" of what the system should do |
| ⚔️ Challenger (挑战者) | Actively tries to break the system, finds edge cases and violations | Probe boundaries, inject faults, stress test, look for contradictions in Builder's manifest |
| ⚖️ Judge (裁决者) | Arbitrates findings, separates real bugs from false positives, prioritizes | Evaluate Challenger's evidence, confirm/deny issues, assign severity |

#### Execution Flow

```
1. Builder produces behavior manifest:
   - What should each feature do?
   - What are the invariants?
   - What are the assumptions?
   - What are the known constraints?

2. Challenger attacks the manifest:
   - Which invariants can be violated?
   - Which assumptions are wrong?
   - Where do boundaries break?
   - What inputs cause unexpected behavior?
   - What sequences of actions lead to bad states?
   - What happens when external dependencies fail?

3. Judge evaluates findings:
   - Is this a real bug or expected behavior?
   - What is the severity (P0/P1/P2/P3)?
   - What is the root cause?
   - Is this reproducible?
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

**Execution:**
- Launch 3 parallel subagents (Builder, Challenger, Judge)
- Builder produces manifest first
- Challenger attacks manifest + code
- Judge evaluates all findings
- Merge into unified issue set

### Step 2: 发现问题 (Discover Issues)

Merge adversarial findings with systematic analysis.

```
1. Collect Builder's behavior manifest
2. Collect Challenger's attack results
3. Collect Judge's evaluations
4. Categorize: functional / UX / performance / architecture / data
5. Prioritize: P0 (crash/security) → P1 (broken feature) → P2 (UX/arch) → P3 (minor)
6. Identify root causes — not just symptoms
7. Create issue tracker (ID, description, severity, source: builder/challenger/judge)
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
- Run all tests: `npm test` → record pass/fail
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

P2/P3 issues are logged for next round but don't block the current round.

### Step 9: 核验环节 (Verification Phase)

**开发完成后必须进入核验环节**，使用独立子任务/subagent逐个流程进行核验，避免上下文干扰。

#### 核验原则

```
1. 独立性: 每个核验任务使用独立subagent，避免上下文污染
2. 完整性: 必须覆盖100%功能点和流程链路
3. 真实性: 必须真实执行测试并验证结果，不能假设通过
4. 链路性: 必须测试完整流程链路，不只是短流程集成测试
```

#### 核验流程

```
1. 梳理所有功能点和流程链路
   - 从 PLAN.md 提取所有功能点
   - 梳理完整流程链路（从入口到出口）
   - 识别关键路径和分支路径

2. 逐个流程核验
   - 每个流程使用独立subagent
   - subagent必须: 运行测试 → 验证结果 → 记录问题
   - 不通过的流程立即进入修复循环

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

#### 核验任务模板

每个核验任务使用独立subagent:

```
任务: 核验 [流程名称]
描述:
1. 运行相关测试: npm test -- [test-file]
2. 验证测试结果: 通过/失败/跳过
3. 检查完整链路: [入口] → [步骤1] → [步骤2] → ... → [出口]
4. 记录发现的问题
输出: 核验结果 + 问题清单
```

#### 核验完成标准

```
✅ 所有功能点有对应测试
✅ 所有测试通过
✅ 所有流程链路有完整测试（非短流程）
✅ 核验结果真实（实际运行验证）
✅ 问题清单已记录并分类
```

### Step 10: 文档记录 (Document Round)

Create iteration report and plan next round. All docs go to `docs/iterations/{name}/round-N/`.

**必做事项:**
1. 生成本轮报告 → `docs/iterations/{name}/round-N/report.md`
2. **生成下轮计划** → `docs/iterations/{name}/round-N+1/plan.md`
   - 从 `PLAN.md`(总计划) 拆解下轮焦点
   - 合并本轮剩余问题 + 新发现
   - 每3轮还需合并复盘改进措施
   - 包含：焦点领域、对抗性评测重点、质量目标
3. **每3轮进行复盘**（当 N % 3 == 0 时）→ 写入 `report.md` Section 9
   - 3轮趋势分析（问题发现/修复趋势、质量指标变化）
   - 流程改进点（哪里做得好、哪里可以改进）
   - 工具/方法改进建议
   - 改进措施列入下轮 `plan.md`

```markdown
# Round N 迭代报告

> **日期**: YYYY-MM-DD
> **迭代周期**: 第N轮 — [主题]

## 1. 对抗性评测发现
### Builder 行为清单
| ID | 功能 | 预期行为 | 假设 |
|----|------|---------|------|
| B-01 | ... | ... | ... |

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
> 详见 `docs/iterations/{name}/round-N+1/plan.md`

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
| 全局 | 迭代总计划 | `PLAN.md` | `docs/iterations/{name}/` | 迭代开始前，贯穿全局 |
| 轮次 | 轮次计划 | `round-N/plan.md` | `docs/iterations/{name}/round-N/` | 每轮开始前（上轮结束时生成） |
| 轮次 | 轮次报告 | `round-N/report.md` | `docs/iterations/{name}/round-N/` | 每轮结束 |
| 轮次 | 问题追踪 | `round-N/issues.md` | `docs/iterations/{name}/round-N/` | 对抗性评测后 |
| 轮次 | 架构审查 | `round-N/arch-review.md` | `docs/iterations/{name}/round-N/` | 架构审查后 |
| 轮次 | 行为清单 | `round-N/behavior-manifest.md` | `docs/iterations/{name}/round-N/` | Builder输出 |
| 全局 | 迭代总结 | `SUMMARY.md` | `docs/iterations/{name}/` | 迭代完成时 |

### Directory Structure

项目按**迭代主题**组织，每个迭代有独立计划和多轮子流程：

```
docs/
└── iterations/
    ├── {iteration-name}/              # 迭代主题，如 map-system, combat-system
    │   ├── PLAN.md                    # 迭代总计划(贯穿全局，迭代开始前写)
    │   ├── round-1/                   # 第1轮
    │   │   ├── plan.md               # 本轮计划(第1轮从PLAN.md拆解)
    │   │   ├── report.md
    │   │   ├── issues.md
    │   │   ├── arch-review.md
    │   │   └── behavior-manifest.md
    │   ├── round-2/                   # 第2轮
    │   │   ├── plan.md               # 本轮计划(上轮结束时生成)
    │   │   └── ...
    │   ├── round-3/                   # 第3轮 (含复盘)
    │   │   ├── plan.md
    │   │   └── ...
    │   └── SUMMARY.md                 # 迭代完成总结
    ├── {another-iteration}/
    │   ├── PLAN.md
    │   ├── round-1/
    │   └── SUMMARY.md
    └── ...
```

**两层计划关系**:
- `PLAN.md` = 迭代总计划，定义整个迭代的目标、范围、验收标准
- `round-N/plan.md` = 本轮计划，从总计划拆解 + 上轮遗留问题 + 复盘改进措施

**流程**: `PLAN.md`(总) → `round-1/plan.md`(拆解) → 执行 → `round-2/plan.md`(上轮生成) → ... → `SUMMARY.md`

**命名规则**:
- 迭代目录: `docs/iterations/{kebab-case-name}/` — 按功能模块命名
- 每轮子目录: `round-{N}/` — 轮次编号
- 文件名统一不带轮次前缀（目录已包含）

### Template: ITERATION-PLAN.md

```markdown
# 迭代计划

> **项目**: [项目名]
> **创建日期**: YYYY-MM-DD
> **最后更新**: YYYY-MM-DD

## 目标
[项目质量目标]

## 迭代记录
| 轮次 | 日期 | 主题 | P0 | P1 | 测试通过率 | 状态 |
|------|------|------|:--:|:--:|:---------:|:----:|
| R1 | YYYY-MM-DD | ... | N | N | N% | ✅/🔄 |
| R2 | ... | ... | N | N | N% | ... |

## 质量目标
| 指标 | 目标 | 当前 |
|------|------|------|
| P0 | 0 | N |
| P1 | 0 | N |
| 测试通过率 | 100% | N% |
| 架构问题 | 0 | N |
```

### Template: `docs/iterations/{name}/PLAN.md`

```markdown
# {迭代名称} 迭代计划

> **创建日期**: YYYY-MM-DD
> **目标**: [本轮迭代要达成的目标]
> **范围**: [涉及的模块/功能]

## 背景
[为什么要做这个迭代，解决什么问题]

## 目标
- [ ] 目标1
- [ ] 目标2
- [ ] 目标3

## 质量目标
| 指标 | 目标 |
|------|------|
| P0 | 0 |
| P1 | 0 |
| 测试通过率 | 100% |
| 架构问题 | 0 |

## 预估轮次
| 轮次 | 焦点 |
|------|------|
| R1 | ... |
| R2 | ... |

## 验收标准
1. [可验证的验收条件]
2. [可验证的验收条件]
```

### Template: `docs/iterations/{name}/round-N/plan.md`

> 轮次计划：从总计划(PLAN.md)拆解 + 上轮遗留 + 复盘改进

```markdown
# Round N 计划

> **迭代**: {iteration-name}
> **轮次**: Round N
> **来源**: `PLAN.md` + Round N-1 `report.md` + 复盘改进(如适用)
> **日期**: YYYY-MM-DD

## 本轮焦点
> 从 PLAN.md 拆解，结合上轮遗留问题

| 优先级 | 领域 | 来源 | 原因 |
|:------:|------|------|------|
| P1 | ... | PLAN.md | 总计划第X项 |
| P1 | ... | R(N-1)遗留 | 上轮未修复 |
| P2 | ... | 新发现 | 上轮对抗性评测发现 |

## 对抗性评测重点
- [ ] [重点攻击方向1]
- [ ] [重点攻击方向2]

## 质量目标
| 指标 | 目标 |
|------|------|
| P0 | 0 |
| P1 | 0 |
| 测试通过率 | N% |

## 改进措施（来自复盘，仅 R3/R6/R9... 有）
| ID | 措施 | 验收标准 |
|----|------|---------|
| IMP-01 | ... | ... |
```

### Template: `docs/iterations/{name}/SUMMARY.md`

```markdown
# {迭代名称} 总结

> **日期**: YYYY-MM-DD — YYYY-MM-DD
> **轮次**: N轮
> **状态**: ✅ 完成 / 🔄 进行中 / ❌ 中止

## 目标达成
| 目标 | 状态 | 备注 |
|------|:----:|------|
| 目标1 | ✅/❌ | ... |
| 目标2 | ✅/❌ | ... |

## 迭代趋势
| 指标 | R1 | R2 | ... | RN |
|------|:--:|:--:|:---:|:--:|
| 对抗性发现 | | | | |
| 修复数 | | | | |
| P0 | | | | |
| P1 | | | | |
| 测试通过率 | | | | |
| 内部循环次数 | | | | |

## 关键发现
1. [重要发现1]
2. [重要发现2]

## 经验教训
- [经验1]
- [经验2]

## 后续工作
- [ ] [遗留问题1]
- [ ] [遗留问题2]
```

### Template: `docs/iterations/{name}/round-N/report.md`

```markdown
# Round N 迭代报告

> **日期**: YYYY-MM-DD
> **迭代周期**: 第N轮 — [主题]
> **内部循环次数**: N

## 1. 对抗性评测发现

### Builder 行为清单
| ID | 功能 | 预期行为 | 假设 | 状态 |
|----|------|---------|------|:----:|
| B-01 | ... | ... | ... | ✅/❌ |

### Challenger 攻击结果
| ID | 攻击维度 | 攻击方式 | 结果 | Judge判定 |
|----|---------|---------|------|----------|
| C-01 | 边界/序列/数据/状态/集成 | ... | 崩溃/异常/正常 | P0/P1/P2/P3/否决 |

### Judge 综合评定
| ID | 严重度 | 可复现 | 根因 | 建议 |
|----|:------:|:------:|------|------|
| J-01 | P0 | 是/否 | ... | ... |

## 2. 修复内容
| ID | 对应问题 | 文件:行 | 修复方式 | 影响 |
|----|---------|---------|---------|------|
| F-01 | C-01 | file.ts:123 | ... | ... |

## 3. 内部循环记录
| 子轮次 | 发现 | 修复 | 剩余P0 | 剩余P1 | 备注 |
|--------|:----:|:----:|:------:|:------:|------|
| N.1 | 5 | 3 | 2 | 0 | 首次对抗 |
| N.2 | 2 | 2 | 0 | 0 | 修复后重测 |
| **合计** | **7** | **5** | **0** | **0** | |

## 4. 测试结果
| 测试套件 | 通过 | 失败 | 跳过 |
|----------|:----:|:----:|:----:|
| [suite] | N | N | N |
| **总计** | **N** | **N** | **N** |

## 5. 架构审查结果
| 检查项 | 状态 | 问题描述 |
|--------|:----:|----------|
| 依赖方向 | ✅/❌ | ... |
| 层级边界 | ✅/❌ | ... |
| 类型安全 | ✅/❌ | ... |
| 数据流 | ✅/❌ | ... |
| 代码重复 | ✅/❌ | ... |

## 6. 回顾(跨轮趋势)
| 指标 | R1 | R2 | ... | RN | 趋势 |
|------|:--:|:--:|:---:|:--:|:----:|
| 测试通过率 | | | | | ↑/↓/→ |
| P0问题 | | | | | |
| P1问题 | | | | | |
| 对抗性发现 | | | | | |
| 内部循环次数 | | | | | |
| 架构问题 | | | | | |

## 7. 剩余问题(移交下轮)
| ID | 问题 | 优先级 | 来源 | 备注 |
|----|------|:------:|------|------|

## 8. 下轮计划
1. [Focus area]
2. [Focus area]
```

### Template: `docs/iterations/{name}/round-N/issues.md`

```markdown
# Round N 问题清单

> **日期**: YYYY-MM-DD
> **来源**: 对抗性评测

## 问题列表
| ID | 严重度 | 类型 | 来源 | 描述 | 文件 | 状态 |
|----|:------:|------|------|------|------|:----:|
| I-01 | P0 | 功能 | C-01 | ... | ... | 🔄/✅/❌ |
| I-02 | P1 | 数据 | C-02 | ... | ... | ... |

## 统计
- P0: N (修复: N)
- P1: N (修复: N)
- P2: N (修复: N)
- P3: N (backlog)
```

### Template: `docs/iterations/{name}/round-N/arch-review.md`

```markdown
# Round N 架构审查

> **日期**: YYYY-MM-DD

## 审查结果
| 检查项 | 状态 | 问题描述 | 涉及文件 | 建议 |
|--------|:----:|----------|---------|------|
| 依赖方向 | ✅/❌ | ... | ... | ... |
| 层级边界 | ✅/❌ | ... | ... | ... |
| 类型安全 | ✅/❌ | ... | ... | ... |
| 事件总线 | ✅/❌ | ... | ... | ... |
| 数据流 | ✅/❌ | ... | ... | ... |
| 代码重复 | ✅/❌ | ... | ... | ... |
| 死代码 | ✅/❌ | ... | ... | ... |

## 统计
- 通过: N/7
- 失败: N/7
- 需修复: N
```
```

## Execution Model

### Use Subagents

**CRITICAL**: Main session orchestrates only. All work via subagents.

```
Main session: Task planning, status tracking, user communication
Subagents:   Code changes, test runs, reviews, screenshots
```

### Parallel Execution

Independent tasks run in parallel:

```
# 对抗性评测 — 3 roles in parallel
Task 1: Builder — produce behavior manifest
Task 2: Challenger — attack the system
Task 3: Judge — evaluate findings
(Builder must finish before Challenger starts; Judge runs after Challenger)

# 修复独立问题 — parallel
Task A: Fix P0 bug in SystemA
Task B: Fix P2 issue in SystemB
```

### Sequential Execution

Dependent tasks run sequentially:

```
Step 1: Adversarial test → finds issues
Step 2: Fix issues (depends on Step 1)
Step 3: Re-evaluate (depends on Step 2)
Step 4: Architecture review (depends on Step 3)
Step 5: Fix new issues (depends on Step 4)
→ Loop until clean
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
10. **Skipping verification phase** — Must use independent subagents to verify after development
11. **Assuming tests pass** — Must actually run tests and verify results, never assume
12. **Incomplete coverage** — Must cover 100% of features and flow paths

## Trigger Phrases

- "迭代" / "iterate"
- "对抗性评测" / "adversarial evaluation"
- "架构审查" / "architecture review"
- "循环改进" / "iterative improvement"
- "继续迭代" / "continue iterating"
- "builder challenger judge"
- "多维度评测" / "multi-dimensional evaluation"
