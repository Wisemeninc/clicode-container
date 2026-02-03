# OpenCode Docker Container

A pre-configured Docker container with OpenCode CLI, Claude Code, and Personal AI Infrastructure (PAI) for AI-assisted development.

## Features

- **OpenCode CLI** - AI coding assistant at `/root/.opencode/bin/opencode`
- **Claude Code** - Anthropic's official CLI at `/root/.local/bin/claude`
- **PAI v2.5** - Personal AI Infrastructure from danielmiessler at `/root/.claude`
- **Bun Runtime** - JavaScript/TypeScript runtime for PAI scripts
- **SSH Access** - Remote access on port 2222
- **Persistent Volumes** - Configs survive container restarts

## Quick Start

### 1. Build and Start

```bash
docker-compose up -d --build
```

### 2. Connect via SSH

```bash
ssh root@localhost -p 2222
# password: password123
```

### 3. Run OpenCode or Claude Code

```bash
opencode    # OpenCode CLI
claude      # Claude Code CLI
pai         # PAI launcher (uses Claude Code)
```

## Using PAI (Personal AI Infrastructure)

PAI v2.5 is pre-installed at `/root/.claude`. OpenCode has built-in compatibility with Claude Code's configuration format, so some PAI features work automatically.

### PAI Compatibility Matrix

| PAI Feature | Works with OpenCode | Notes |
|-------------|:-------------------:|-------|
| **Skills** (`~/.claude/skills/`) | ✅ Yes | OpenCode natively reads `SKILL.md` files |
| **CLAUDE.md** | ✅ Yes | Used as fallback system prompt |
| **Fabric Patterns** | ✅ Yes | Loaded via skill system |
| **Agents** (`~/.claude/agents/`) | ❌ No | Different format - use `.opencode/agents/` |
| **Hooks** (`~/.claude/hooks/`) | ❌ No | Different system - use `.opencode/plugins/` |
| **pai CLI** | ✅ Yes | Works with Claude Code (included in container) |
| **VoiceServer** | ⚠️ Partial | Works if server running on host |
| **MCP Servers** | ⚠️ Different | Configure in `opencode.json` instead |

### What Works

When you run `opencode`, it automatically reads:

1. `~/.claude/CLAUDE.md` - System instructions (fallback if no `AGENTS.md`)
2. `~/.claude/skills/*/SKILL.md` - All 29 PAI skills are available

**Available Skills:** Research, OSINT, Fabric, WebAssessment, Recon, Browser, Apify, BrightData, Documents, Art, Agents, Council, RedTeam, and more.

### What Doesn't Work

- **PAI Agents** - OpenCode uses a different agent format. PAI's `.md` agents in `~/.claude/agents/` won't load. To use custom agents, create them in `.opencode/agents/` following [OpenCode agent format](https://opencode.ai/docs/agents/).

- **PAI Hooks** - OpenCode uses JavaScript/TypeScript plugins instead of Claude Code's hook scripts. PAI's hooks for voice notifications, session summaries, etc. won't execute with OpenCode. Use `pai` or `claude` directly for full hook support, or see [OpenCode Plugins docs](https://opencode.ai/docs/plugins/) to create equivalent functionality.

### Using PAI with Claude Code

For full PAI functionality including hooks, agents, and the Algorithm, use the `pai` command which launches Claude Code:

```bash
pai              # Launch Claude Code with PAI
pai -r           # Resume last session
pai -m bd        # Launch with Bright Data MCP
claude           # Launch Claude Code directly
```

### Initial Setup (Optional)

To customize PAI with your name and preferences:

```bash
cd /root/.claude
bun run INSTALL.ts
```

Note: This configures PAI settings but won't enable incompatible features.

### PAI Documentation

See the PAI skill files in `/root/.claude/skills/PAI/` for detailed documentation:
- `SKILL.md` - Core system overview
- `PAI_ARCHITECTURE.md` - Technical architecture

## Volume Mounts

| Host Path | Container Path | Purpose |
|-----------|----------------|---------|
| `~/.claude` | `/root/.claude` | PAI config & skills |
| `~/.opencode` | `/root/.opencode` | OpenCode config |
| `./code_output` | `/workspace/output` | Code output directory |

## Configuration

### Environment Variables

Copy the example environment file and add your API keys:

```bash
cp .env.example .env
# Edit .env with your API keys
```

See `.env.example` for all options.

#### Claude Code Providers

**Anthropic Direct API:**
```bash
ANTHROPIC_API_KEY=sk-ant-xxxxx
```

**Microsoft Foundry (Azure):**
```bash
CLAUDE_CODE_USE_FOUNDRY=1
ANTHROPIC_FOUNDRY_RESOURCE=your-resource-name
ANTHROPIC_FOUNDRY_API_KEY=your-azure-api-key
# Or omit API key for Microsoft Entra ID auth
```

**Amazon Bedrock:**
```bash
CLAUDE_CODE_USE_BEDROCK=1
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=xxxxx
AWS_SECRET_ACCESS_KEY=xxxxx
```

**Google Vertex AI:**
```bash
CLAUDE_CODE_USE_VERTEX=1
CLOUD_ML_REGION=us-east5
ANTHROPIC_VERTEX_PROJECT_ID=your-project-id
```

**Ollama with Claude Code:**
```bash
# See: https://docs.ollama.com/integrations/claude-code
ANTHROPIC_AUTH_TOKEN=ollama
ANTHROPIC_API_KEY=
ANTHROPIC_BASE_URL=http://host.docker.internal:11434
# Then run: claude --model qwen3-coder
```

**Corporate Proxy / LLM Gateway:**
```bash
HTTPS_PROXY=https://proxy.example.com:8080
ANTHROPIC_BASE_URL=https://your-llm-gateway.com
```

For detailed Claude Code docs: https://code.claude.com/docs/en/third-party-integrations

#### OpenCode Providers

OpenCode uses the `/connect` command to add most providers. API keys are stored in `~/.local/share/opencode/auth.json`.

**Amazon Bedrock:**
```bash
AWS_ACCESS_KEY_ID=xxxxx
AWS_SECRET_ACCESS_KEY=xxxxx
AWS_REGION=us-east-1
# Or use: AWS_PROFILE=my-profile
```

**Azure OpenAI:**
```bash
AZURE_RESOURCE_NAME=your-resource-name
# API key set via /connect command in OpenCode
```

**Google Vertex AI:**
```bash
GOOGLE_CLOUD_PROJECT=your-project-id
VERTEX_LOCATION=global
# GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
```

**Other providers (Anthropic, OpenAI, Groq, etc.):**
Use the `/connect` command inside OpenCode to add credentials.

For detailed OpenCode docs: https://opencode.ai/docs/providers/

### SSH Keys

To use your own SSH keys instead of password auth:

```bash
# Copy your public key
docker cp ~/.ssh/id_rsa.pub opencode:/root/.ssh/authorized_keys
```

## Ports

| Port | Service |
|------|---------|
| 2222 | SSH |

## Building Manually

```bash
docker build -f Dockerfile.opencode -t opencode-container .
docker run -d -p 2222:22 --name opencode opencode-container
```

## Troubleshooting

### OpenCode not found
```bash
# Use full path or reload shell
/root/.opencode/bin/opencode
# or
source ~/.bashrc
```

### PAI not working
```bash
# Ensure Bun is in path
export PATH="/root/.bun/bin:$PATH"
# Check PAI installation
ls -la /root/.claude/
```

## License

MIT
