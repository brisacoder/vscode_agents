
# Getting Started

Drop the agents under ~/.copilot/agents or your user data (specific to your VS Code profile)

https://code.visualstudio.com/docs/copilot/customization/custom-agents


In your Vscode IDE if you click on the Copilot icon in the sidebar, you should see your agents under "Available Agents" section:


![Available Agents](images/Screenshot%202026-05-20%20080310.png)


# Usage 

**The code reviewer is autonomous and comprehensive.** It will delegate to all other agents as needed to create a comprehensive review document. It will review the code, run tests, check for security vulnerabilities, check for performance issues, check for code quality issues, check for documentation issues and check for any other issues it can find.

Sub-agents do not consume the original contextual window so main agent will have access to the entire codebase and all the information it needs to create a comprehensive review document.

You can type:

```
Review this package <path to package>. 
```

You can also provide the path to a single file, group of files, PR or a folder. The agent will review the code and create a comprehensive review document.

# Code Executor Agent

The code executor agent can read the ouput of the code reviewer and fix everything, back to back. It is also autonomous and comprehensive. It will delegate to all other agents as needed to fix all the issues found by the code reviewer. It will fix the code, run tests, fix any issues found by the tests, fix any security vulnerabilities, fix any performance issues, fix any code quality issues, fix any documentation issues and fix any other issues it can find.

# Single Agents

You can also choose one of the individual agents to run specific checks. For example, you can choose the "Type Annotation Auithor" agent to fix all type annotation issues in your files. 


# Skills

Drop the contents of `src/skills/` under `~/.copilot/skills` (or your workspace's `.github/skills/` folder).

https://code.visualstudio.com/docs/copilot/customization/agent-skills

The agents in this repo are **not self-contained** — most of them load one or more shared skills at the start of every task via the `skill` tool, and will not behave correctly if the skills are missing. Skills hold the binding rules that are shared across many agents instead of being copy-pasted into each one: workspace coding-standards pre-reads, the canonical `uv` command set, the Zen-of-Python idiom ranking, the "fix the cause, never silence the symptom" suppression ban, the three-round saturation review loop every specialist runs in Review mode, the GitHub-native stacked-PR workflow, the post-submit stack-shepherding contract, and the consolidated code-review report format.

| Skill | Used by |
|---|---|
| `workspace-standards-preread` | Every code-writing and code-reviewing agent, at the start of any Write/Optimize/Rewrite/Review pass |
| `python-idioms-default` | Every agent that writes, reviews, or recommends Python code |
| `uv-toolchain` | Every agent that runs tests, formatters, linters, type checkers, or scripts |
| `no-suppression-hacks` | The 23 code-writing agents (not pure review/orchestrate/doc agents) |
| `saturation-review-loop` | Every Review-mode specialist, to drive findings to zero-delta closure |
| `github-stacking` | Any agent that creates branches, commits, or opens/updates/monitors pull requests |
| `stack-shepherding` | PR Stack Planner, PR Watch Agent, PR Review Resolver — what "done" means once a stack is submitted |
| `consolidated-review-report` | Code Reviewer Agent, when assembling the final multi-specialist report |

Without these installed, an agent that says "invoke the `skill` tool to load these N shared skills" will have nothing to load.


# Why VSCode agents?

Because VScode CoPliot agents have access to all extensions, plug-ins and MCP servers you have installed out of the box.

Example, this agent has access to GitHub, Markitdown, Playwright, Langchain MCP, PostgreSQL MCP, Jupyter, Browser, Mermaid Diagrams, Drawi.o and Pylance MCP server:

```yaml
tools: [vscode, execute, read, agent, edit, search, web, 'github/*', 'microsoft/markitdown/*', 'playwright/*', 'langchain-mcp/*', 'postgresql-mcp/*', browser, 'pylance-mcp-server/*', vscode.mermaid-chat-features/renderMermaidDiagram, github.vscode-pull-request-github/issue_fetch, github.vscode-pull-request-github/labels_fetch, github.vscode-pull-request-github/notification_fetch, github.vscode-pull-request-github/doSearch, github.vscode-pull-request-github/activePullRequest, github.vscode-pull-request-github/pullRequestStatusChecks, github.vscode-pull-request-github/openPullRequest, github.vscode-pull-request-github/create_pull_request, github.vscode-pull-request-github/resolveReviewThread, ms-azuretools.vscode-containers/containerToolsConfig, ms-ossdata.vscode-pgsql/pgsql_migration_oracle_app, ms-ossdata.vscode-pgsql/pgsql_migration_show_report, ms-python.python/getPythonEnvironmentInfo, ms-python.python/getPythonExecutableCommand, ms-python.python/installPythonPackage, ms-python.python/configurePythonEnvironment, ms-toolsai.jupyter/configureNotebook, ms-toolsai.jupyter/listNotebookPackages, ms-toolsai.jupyter/installNotebookPackages, todo]
```