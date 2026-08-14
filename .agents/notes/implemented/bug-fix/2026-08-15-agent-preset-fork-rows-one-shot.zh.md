# Agent Note: CLI agent preset 把 fork 委派绑定为 one-shot

Status: implemented

[English](2026-08-15-agent-preset-fork-rows-one-shot.md) | 中文

## 问题

[fork 出的 child 保持 one-shot](../architecture/2026-08-10-fork-children-stay-one-shot.md) 要求所有随附的 fork 委派工具都绑定为 `backgroundMode: one-shot`，因为可继续 child 的 `report` 工具 schema 与 `tool:report` 提示词 section 位于请求头部、先于继承的历史，会抵消 fork 唯一的回报——提供方侧的前缀复用。落地该决策的改动对齐了 [base 组合包](../../../../packages/bundle/base/cordis.patch.yml)与两个可运行示例；三个 CLI agent preset——[standard](../../../../apps/cli/config/agent-presets/standard/agent.cordis.yml)、[code](../../../../apps/cli/config/agent-presets/code/agent.cordis.yml) 与 [cordis](../../../../apps/cli/config/agent-presets/cordis/agent.cordis.yml)——早于该决策创建，创建时复制了 base 组合包当时仍为 continuable 的行，且未被一并清理。

这些 preset 是默认 Web 部署的 agent 平面，而非边缘组合：[web-app 组合包](../../../../packages/bundle/web-app/cordis.patch.yml)禁用了 base 的委派工具行，每个会话的 agent 都从 preset 挂载（默认 `standard`），并把 base 的 `tool-subagent-report` 行保留在宿主平面。因此每个默认 Web 会话提供的都是可继续的 `subagent_fork`，其 child 同时携带两项请求头部增量——正是该决策所否决的、付出 fork 的复制成本却收不到任何收益的组合。这就是所属 Agent Note 已接受风险的成真实例：约束存在于配置且没有门禁，陈旧的行照常加载，没有任何响亮的失败。

## 决策

三个 preset 都把 `subagent_fork` 绑定为 `backgroundMode: one-shot`，每行都带上 base 组合包中说明原因并指向所属 Agent Note 的注释。`run_in_background` 保持可用，跟随 base 组合包而非示例的 `enableRunInBackground: false`：这些 preset 在宿主平面的后台任务注册表之上挂载了 `job_*` 控制工具，因此 one-shot 后台启动有自己的收集路径。

所属架构 Agent Note 仍是该决策的家；它的随附组合清单与配置文件计数现已把这些 preset 计入。

## 备选方案

**保持 preset 为 continuable 并反过来修订决策。** 这些行没有任何被记录的意图——它们早于决策，只是复制了 base 组合包的早期状态——而决策的分析对这些组合完全成立：它们与宿主平面的 report 注册并存，因此可继续的 fork child 会丢掉整段共享区间。这正是所属 Agent Note 已经否决的「照常随附可继续并接受损失」备选。

**加上这次回归所呼吁的加载期拒绝。** 所属 Agent Note 否决了它，因为 `tool-subagent` 观察不到 report 包，且没有该包时这一组合是合法的；一次成真实例不改变这一分析。重开这个问题将取代所属 Agent Note 已接受的风险——那是另一个独立决策，不属于这次对齐。

**像示例那样设置 `enableRunInBackground: false`。** 在这里形状不对：示例禁用后台启动是因为其组合没有任务收集路径；preset 组合有。

## 后果

- 每个 preset 组合——包括默认 Web 部署——都把 `subagent_fork` 绑定为 one-shot：其 schema 采用 one-shot 措辞，`send_message` 只寻址 spawn 出的 child，fork child 的请求前缀重新与其 parent 一致。
- 快照 fixture 无一变化：没有无密钥快照钉住 preset 组合的工具 schema，而[preset 目录 e2e](../../../../apps/cli/tests/web-agent-presets.e2e.ts) 断言的是工具名列表，本次改动保持不变。preset 组合面向模型的 schema 仍未被任何无密钥快照钉住；示例的伴随文件只为其自身组合钉住 one-shot 措辞。
