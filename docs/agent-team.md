# Agent team

To build Mona's Project Pulse dashboard, I will run a custom multi-agent team in GitHub Copilot CLI from this Codespace. The Orchestrator coordinates the flow, the Planner defines implementation phases, the Designer shapes the dashboard experience, and the Coder implements the code changes.

- Agent: Orchestrator
	- Target model: Claude Opus 4.7 (copilot)
	- Responsibility: Coordinates Planner, Designer, and Coder; sequences phases; delegates file-scoped work; verifies integrated outcomes.
	- Definition: `.github/agents/orchestrator.agent.md`

- Agent: Planner
	- Target model: Claude Opus 4.7 (copilot)
	- Responsibility: Researches the repo and requirements, identifies risks and dependencies, and returns an ordered implementation plan with file ownership and validation expectations.
	- Definition: `.github/agents/planner.agent.md`

- Agent: Designer
	- Target model: Gemini 3.1 Pro (copilot)
	- Responsibility: Owns UI/UX quality for Project Pulse, including dashboard layout, accessibility, hierarchy, responsive behavior, and visual clarity.
	- Definition: `.github/agents/designer.agent.md`

- Agent: Coder
	- Target model: GPT-5.5 (copilot)
	- Responsibility: Implements logic and file changes with clear, testable behavior, explicit errors, and repository-consistent structure.
	- Definition: `.github/agents/coder.agent.md`

This setup is intentionally specialized: planning first, then design and implementation with clear file boundaries, all orchestrated through GitHub Copilot CLI in a Codespace environment.
