
# AI 技术评审委员会入口规则

当本规则被 Cursor Agent 选中时，按 `ai-tech-review-committee/SKILL.md` 的委员会流程执行评审。

若当前上下文无法读取该 Skill 文件，则按以下最小流程执行：

1. `Intake`：复述目标、用户、当前方案、约束、假设和缺口。
2. `Role Selection`：选择主席、产品、架构、工程、安全、SRE，并按需加入 QA、Data/AI、UX、交付成本。
3. `Individual Expert Review`：各角色先独立给出 verdict、concerns、recommendations、questions。
4. `Cross-Examination`：指出角色之间的关键 trade-off。
5. `Chair Synthesis`：给出 `Go / Go with changes / Rework / Discovery needed`。
6. `Action Plan`：输出 Now / Next / Before Launch。

---
> Source: [yangin/ai-skills](https://github.com/yangin/ai-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-27 -->
