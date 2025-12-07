---
name: n8n-project-scaffolder
description: "AI-guided scaffolding for n8n workflow automation projects"
version: "1.0.0"
activation:
  - "n8n project"
  - "workflow automation"
  - "n8n workflow"
  - "create n8n"
  - "scaffold n8n"
  - "new automation"
dependencies:
  mcp_servers:
    required:
      - n8n-mcp
    optional: []
  skills:
    optional:
      - n8n-skills
---

# n8n Project Scaffolding Skill

## Overview

This skill guides Claude in creating well-structured n8n workflow automation projects with proper MCP integration for AI-assisted workflow development.

## Prerequisites Check

Before scaffolding, verify the following are available:

### Required
1. **Node.js 18+** - `node --version`
2. **npm 9+** - `npm --version`

### For MCP Integration (Highly Recommended)
3. **n8n-mcp server** configured in Claude settings or project `.mcp.json`
4. **n8n instance** accessible (local Docker or cloud)
5. **n8n API key** generated from n8n Settings → API

### Optional Enhancements
6. **n8n-skills** installed for enhanced workflow building
7. **Docker** for containerized deployment

## Project Templates

### 1. Basic Workflow Project (`basic-workflow`)

**Best for:** Getting started, simple automations, learning n8n

**Structure:**
```
{project-name}/
├── .claude/
│   └── settings.local.json    # Pre-configured n8n-mcp
├── workflows/
│   └── example-webhook.json   # Sample webhook workflow
├── .env.example               # Environment template
├── docker-compose.yml         # Local n8n setup
└── README.md                  # Project documentation
```

**Included Workflow:** Webhook → Process Data → HTTP Response

### 2. AI Agent Workflow (`ai-agent-workflow`)

**Best for:** AI-powered automations, chatbots, intelligent processing

**Structure:**
```
{project-name}/
├── .claude/
│   └── settings.local.json
├── workflows/
│   ├── ai-agent-basic.json    # Simple AI agent
│   └── ai-agent-tools.json    # Agent with custom tools
├── credentials/
│   └── .gitkeep
├── .env.example               # Including AI API keys
├── docker-compose.yml
└── README.md
```

**Included Patterns:**
- Chat Trigger → AI Agent → Response
- AI Agent with HTTP Tool, Code Tool
- Memory-enabled conversations

### 3. Production Docker Setup (`production-docker`)

**Best for:** Deploying n8n at scale, team environments

**Structure:**
```
{project-name}/
├── .claude/
│   └── settings.local.json
├── docker/
│   ├── nginx/
│   │   └── nginx.conf         # Reverse proxy config
│   └── ssl/
│       └── .gitkeep           # SSL certificates location
├── monitoring/
│   ├── prometheus/
│   │   └── prometheus.yml
│   └── grafana/
│       └── dashboards/
├── workflows/
│   └── .gitkeep
├── backups/
│   └── .gitkeep
├── .env.example
├── docker-compose.yml         # Full stack
├── docker-compose.prod.yml    # Production overrides
└── README.md
```

### 4. Multi-Workflow Project (`multi-workflow`)

**Best for:** Complex automation systems, interconnected workflows

**Structure:**
```
{project-name}/
├── workflows/
│   ├── triggers/              # Entry point workflows
│   │   ├── webhook-receiver.json
│   │   └── scheduled-job.json
│   ├── processors/            # Processing workflows
│   │   ├── data-transformer.json
│   │   └── enrichment.json
│   └── outputs/               # Output workflows
│       ├── notification.json
│       └── database-writer.json
├── shared/
│   └── constants.json         # Shared configuration
└── README.md
```

## MCP Integration

### Setting Up n8n-mcp

The scaffolder automatically creates `.claude/settings.local.json`:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "http://localhost:5678",
        "N8N_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

### Available MCP Tools

Once configured, these n8n-mcp tools become available:

**Discovery & Documentation:**
- `search_nodes` - Find n8n nodes by name or functionality
- `get_node` - Get detailed node information and examples
- `search_templates` - Browse 2,700+ workflow templates

**Validation:**
- `validate_node` - Validate node configuration
- `validate_workflow` - Validate complete workflow JSON

**Workflow Management (requires n8n API):**
- `n8n_create_workflow` - Deploy workflows to n8n
- `n8n_list_workflows` - List existing workflows
- `n8n_update_partial_workflow` - Update workflow sections
- `n8n_test_workflow` - Execute workflow tests

### n8n-skills Integration

For enhanced workflow building, install n8n-skills:

```bash
# In Claude Code
/plugin install czlonkowski/n8n-skills
```

This adds 7 specialized skills:
1. **n8n Expression Syntax** - Correct {{}} patterns
2. **n8n MCP Tools Expert** - Optimal tool usage
3. **n8n Workflow Patterns** - Proven architectures
4. **n8n Validation Expert** - Error resolution
5. **n8n Node Configuration** - Property dependencies
6. **n8n Code JavaScript** - Code node patterns
7. **n8n Code Python** - Python code patterns

## Scaffolding Workflow

### Phase 1: Detection
```
1. Check for existing n8n indicators:
   - .n8n/ directory
   - workflows/*.json files
   - docker-compose.yml with n8n service
   - package.json with n8n dependencies

2. Determine project sub-type:
   - basic: Simple automation
   - ai-agent: LangChain nodes detected
   - production: Docker/nginx config present
```

### Phase 2: Configuration
```
1. Prompt for required variables:
   - PROJECT_NAME (required)
   - N8N_PORT (default: 5678)
   - N8N_HOST (default: localhost)

2. Optional configurations:
   - ENABLE_AI_TOOLS (for AI agent template)
   - INCLUDE_MONITORING (for production template)

3. MCP configuration:
   - Prompt for N8N_API_URL
   - Request API key (or skip for documentation-only)
```

### Phase 3: Generation
```
1. Create directory structure
2. Process and copy template files
3. Substitute variables in templates
4. Configure MCP in .claude/settings.local.json
5. Generate customized README.md
```

### Phase 4: Validation
```
1. Verify all files created
2. Validate JSON syntax in workflow files
3. Test MCP connectivity (if configured)
4. Display post-setup instructions
```

## Template Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `PROJECT_NAME` | string | *required* | Project directory and display name |
| `N8N_VERSION` | string | `latest` | n8n Docker image tag |
| `N8N_PORT` | number | `5678` | Web interface port |
| `N8N_HOST` | string | `localhost` | n8n hostname |
| `N8N_PROTOCOL` | string | `http` | Protocol (http/https) |
| `N8N_API_URL` | string | - | Full URL to n8n API |
| `N8N_API_KEY` | string | - | n8n API authentication key |
| `ENABLE_AI_TOOLS` | boolean | `false` | Include AI node configurations |
| `INCLUDE_MONITORING` | boolean | `false` | Add Prometheus/Grafana |

## Common Workflow Patterns

### Pattern 1: Webhook Processing
```
Webhook Trigger
    ↓
Set Node (extract data)
    ↓
IF Node (validate)
    ↓ (true)          ↓ (false)
Process           Respond Error
    ↓
Respond Success
```

### Pattern 2: Scheduled ETL
```
Schedule Trigger (cron)
    ↓
HTTP Request (fetch data)
    ↓
Code Node (transform)
    ↓
Database/Sheet (store)
    ↓
Slack/Email (notify)
```

### Pattern 3: AI Agent
```
Chat Trigger
    ↓
AI Agent
    ├── LM Chat (OpenAI/Anthropic)
    ├── Tool: HTTP Request
    ├── Tool: Code
    └── Memory: Buffer Window
    ↓
Respond
```

### Pattern 4: Error Handling
```
Trigger
    ↓
Try (main workflow)
    ↓ (error)
Error Trigger
    ↓
Log Error
    ↓
Send Alert
```

## Post-Scaffolding Steps

### For Basic Projects
1. Copy `.env.example` to `.env`
2. Start n8n: `docker-compose up -d`
3. Access n8n at http://localhost:5678
4. Import example workflow from `workflows/`

### For AI Agent Projects
1. Copy `.env.example` to `.env`
2. Add your OpenAI/Anthropic API key to `.env`
3. Start n8n: `docker-compose up -d`
4. Configure credentials in n8n UI
5. Import and test AI agent workflow

### For Production Deployments
1. Configure SSL certificates in `docker/ssl/`
2. Update nginx.conf with your domain
3. Set secure passwords in `.env`
4. Deploy: `docker-compose -f docker-compose.prod.yml up -d`
5. Set up backup automation

## Troubleshooting

### MCP Not Connecting
```
1. Verify n8n is running: curl http://localhost:5678/healthz
2. Check API key is valid
3. Ensure MCP_MODE=stdio is set
4. Check Claude settings for typos
```

### Workflow Import Fails
```
1. Validate JSON syntax
2. Check n8n version compatibility
3. Verify referenced credentials exist
4. Check node type availability
```

### Docker Issues
```
1. Check Docker is running
2. Verify port 5678 is free
3. Check docker-compose logs
4. Ensure sufficient disk space
```

## Best Practices

### Project Organization
- Keep workflows in `workflows/` directory
- Use descriptive workflow names
- Document workflows with Sticky Notes
- Maintain consistent naming conventions

### Version Control
- Always include `.gitignore` (exclude .env, credentials)
- Commit workflow JSON files
- Use environment variables for secrets
- Tag releases with workflow versions

### Security
- Never commit API keys or passwords
- Use n8n's credential system
- Restrict n8n access in production
- Enable audit logging

### Testing
- Use n8n's test execution feature
- Create test workflows for validation
- Document expected inputs/outputs
- Test error scenarios
