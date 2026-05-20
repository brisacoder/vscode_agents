
# Getting Started

Drop the agents under ~/.copilot/agents or your user data (specific to your VS Code profile)

https://code.visualstudio.com/docs/copilot/customization/custom-agents


In your Vscode IDE if you click on the Copilot icon in the sidebar, you should see your agents under "Available Agents" section:


![Available Agents](images/Screenshot%202026-05-20%20080310.png)



# Why VSCode agents?

Because VScode CoPliot agents have access to all extensions, plug-ins and MCP servers you have installed out of the box.

Example, this agent has access to GitHub, Markitdown, Playwright, Langchain MCP, PostgreSQL MCP, Jupyter, Browser, Mermaid Diagrams, Drawi.o and Pylance MCP server:

```yaml
tools: [vscode, execute, read, agent, edit, search, web, 'github/*', 'microsoft/markitdown/*', 'playwright/*', 'langchain-mcp/*', 'postgresql-mcp/*', browser, 'pylance-mcp-server/*', vscode.mermaid-chat-features/renderMermaidDiagram, github.vscode-pull-request-github/issue_fetch, github.vscode-pull-request-github/labels_fetch, github.vscode-pull-request-github/notification_fetch, github.vscode-pull-request-github/doSearch, github.vscode-pull-request-github/activePullRequest, github.vscode-pull-request-github/pullRequestStatusChecks, github.vscode-pull-request-github/openPullRequest, github.vscode-pull-request-github/create_pull_request, github.vscode-pull-request-github/resolveReviewThread, ms-azuretools.vscode-containers/containerToolsConfig, ms-ossdata.vscode-pgsql/pgsql_migration_oracle_app, ms-ossdata.vscode-pgsql/pgsql_migration_show_report, ms-python.python/getPythonEnvironmentInfo, ms-python.python/getPythonExecutableCommand, ms-python.python/installPythonPackage, ms-python.python/configurePythonEnvironment, ms-toolsai.jupyter/configureNotebook, ms-toolsai.jupyter/listNotebookPackages, ms-toolsai.jupyter/installNotebookPackages, todo]
```