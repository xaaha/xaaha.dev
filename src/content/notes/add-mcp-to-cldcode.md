---
title: Adding MCP Servers to Claude Code
date: 2026-02-02
description: How to add mcp servers to claude code
draft: false
category: claude mcp ai
---

There is an extensive Claude documentation on how to configure MCP for Claude Code [here](https://code.claude.com/docs/en/mcp#mcp-installation-scopes). The most important part is the scope of the MCP server. `Local scope`  is only for personal test on specific directory, `project scope` is meant to be version controlled and shared with the team and `user scope` allows the MCP server to be available throughout your machine.

>[!Note]
> The command field in each mcp server requires the specific tool to be installed in system. For example, install node for `npx` and uv for `uvx`

## Examples:

### Github

- Add Github MCP with user scope.
```bash
claude mcp add --transport http github --scope user https://api.githubcopilot.com/mcp/
```

- This adds the  following config in `~/.claude.json`

```json
"mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    }
  }
```

where
```zsh
export GITHUB_TOKEN=$(gh auth token) # in your .zshrc or .bashrc
```

- `gh auth token` requires `gh cli`. Install it with `brew install gh`.
- or `GITHUB_TOKEN` is a literal PAT from Github. This is not recommended as it requires us to manage the scope and expiration date for each token created.

### Atlassian

- Atlassian mcp server:
	- Claude provides example for Atlassian but it failed to reconnect although authentication was successful. Instead use npx. Add this  inside  `~/.claude.json`, in the top level key `"mcpServers"` for user scope

```json
"mcpServers": {
	"atlassian": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.atlassian.com/v1/sse"]
	}
}
```

- To authenticate, launch `claude` and then use the command `/mcp`. Atlassian uses `auth2.0`.
- `/mcp` can also be used to check the connection of all installed servers.

### Other helpful MCPs 

```json
    "awslabs.aws-documentation-mcp-server": {
      "command": "uvx",
      "args": [
        "awslabs.aws-documentation-mcp-server@latest"
      ],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR",
        "AWS_DOCUMENTATION_PARTITION": "aws",
        "MCP_USER_AGENT": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
      },
      "disabled": false,
      "autoApprove": []
    },
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/mcp@latest"
      ]
    }
```

