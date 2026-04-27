You are assisting with the Mandos-AI design and implementation planning effort.

Please read the files in the `designdocs/` folder first. These documents define our current Mandos-AI strategy, including the full design, phased implementation plan, Model Knowledge Pack standard, agents/skills/prompts/MCP architecture, and Staff+ implementation prompts.

Your goal in this discovery pass is NOT to implement anything yet. Instead, build a clear understanding of:
1. What Mandos-AI is supposed to become.
2. Why we are building it.
3. How it should support Mandos users during onboarding, validation, reporting, alert triage, and eventually persisted evidence analysis.
4. How Model Knowledge Packs should standardize model-specific documentation.
5. How GitHub Copilot customization patterns — instructions, agents, skills, prompt files, hooks, and MCP tools — should shape the implementation.
6. What the first small, careful implementation milestones should be.

Important context:
- Mandos is an internal model monitoring / data quality tool.
- Mandos-AI should never report model facts, metric values, thresholds, or monitoring conclusions from memory.
- Markdown artifacts provide domain context.
- Configs provide policy.
- Mandos outputs and persisted evidence provide observed facts.
- Mandos-AI provides interpretation and recommendations.
- Do not assume stale Mandos API names, method names, table names, or schemas from the design docs. Inspect the current Mandos codebase when implementation begins and map the design to the latest APIs.
- For now, focus on understanding and summarizing the design.

External references that influenced the design:
- GitHub awesome-copilot: https://github.com/github/awesome-copilot
- GitHub Copilot CLI: https://github.com/github/copilot-cli
- GitHub Copilot SDK: https://github.com/github/copilot-sdk
- GitHub Copilot custom agents documentation
- GitHub Copilot custom instructions documentation
- GitHub Copilot agent skills documentation
- GitHub Copilot MCP documentation

Please produce a concise discovery summary with:
1. The main goals of Mandos-AI.
2. The intended architecture.
3. The key evidence/governance rules.
4. The planned agent/skills/prompt structure.
5. The purpose of Model Knowledge Packs.
6. The phased implementation path.
7. Any risks, ambiguities, or questions you need answered before implementation.
8. Your recommended first 3 implementation tasks.
