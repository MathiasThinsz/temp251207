# Project Scaffolder Architecture Extension
## A Comprehensive Guide to Adding n8n, Biotech, and Extensible Project Types

**Author:** Claude (AI Architect)  
**Date:** December 6, 2025  
**Target Repository:** https://github.com/fltman/project-scaffolder

---

## Executive Summary

This document provides a comprehensive architectural blueprint for extending the `project-scaffolder` to support:

1. **n8n Automation Projects** - with n8n-mcp and n8n-skills integration
2. **Biotech/Life Sciences Projects** - clinical development, CRO management, etc.
3. **Miscellaneous Projects** - home improvement, personal projects
4. **Future-proof extensibility** - a plugin architecture for unknown project types

The key insight: the scaffolder should evolve from a "project type catalog" to a **pluggable skill system** where each project type is a self-contained plugin with its own analyzer, templates, and AI guidance.

---

## Part 1: Current Architecture Analysis

Based on the visible repository structure:

```
project-scaffolder/
├── .claude/           # Claude configuration
├── docs/              # Documentation
├── skills/
│   └── project-setup/ # Core scaffolding skill
├── templates/         # Project templates
└── CLAUDE.md          # AI instructions
```

### Current Strengths
- Clean separation between skills and templates
- Leverages Claude's skill system for AI-guided scaffolding
- Supports established software project types (Node.js, Python, Rust, Go, Java, Ruby)

### Current Limitations
- Project types appear hardcoded
- No clear extension mechanism for non-code projects
- No MCP server integration for specialized domains

---

## Part 2: Proposed Architecture - The Plugin System

### 2.1 New Directory Structure

```
project-scaffolder/
├── .claude/
│   ├── commands/
│   │   ├── scaffold.md          # Main scaffolding command
│   │   ├── analyze.md           # Project analysis command
│   │   └── add-project-type.md  # Self-extension command
│   └── settings.json
│
├── core/
│   ├── analyzer/
│   │   ├── base-analyzer.ts     # Abstract base class
│   │   ├── analyzer-registry.ts # Plugin registration
│   │   └── interfaces.ts        # TypeScript interfaces
│   ├── scaffolder/
│   │   ├── template-engine.ts   # Template processing
│   │   └── file-generator.ts    # File generation
│   └── mcp/
│       └── mcp-bridge.ts        # MCP server integration
│
├── plugins/
│   ├── _plugin-template/        # Template for new plugins
│   │   ├── plugin.json          # Plugin manifest
│   │   ├── analyzer.ts          # Detection logic
│   │   ├── SKILL.md             # AI guidance
│   │   └── templates/           # Project templates
│   │
│   ├── software/                # Software project types
│   │   ├── nodejs/
│   │   ├── python/
│   │   ├── rust/
│   │   └── ...
│   │
│   ├── n8n/                     # n8n Automation (NEW)
│   │   ├── plugin.json
│   │   ├── analyzer.ts
│   │   ├── SKILL.md
│   │   ├── mcp-config.json      # n8n-mcp integration
│   │   └── templates/
│   │
│   ├── biotech/                 # Biotech/Life Sciences (NEW)
│   │   ├── plugin.json
│   │   ├── analyzer.ts
│   │   ├── SKILL.md
│   │   └── templates/
│   │       ├── clinical-development/
│   │       ├── cro-selection/
│   │       ├── patient-recruitment/
│   │       ├── data-management/
│   │       └── brand-launch/
│   │
│   └── misc/                    # Miscellaneous Projects (NEW)
│       ├── plugin.json
│       ├── analyzer.ts
│       ├── SKILL.md
│       └── templates/
│           ├── home-improvement/
│           └── personal-project/
│
├── skills/
│   └── project-setup/
│       └── SKILL.md             # Updated master skill
│
├── docs/
│   ├── README.md
│   ├── PLUGIN_DEVELOPMENT.md    # How to create plugins
│   ├── PROJECT_TYPES.md         # Available types
│   └── MCP_INTEGRATION.md       # MCP server guide
│
├── CLAUDE.md
└── package.json
```

### 2.2 Plugin Manifest Schema

Every project type plugin must have a `plugin.json`:

```json
{
  "name": "n8n",
  "version": "1.0.0",
  "displayName": "n8n Workflow Automation",
  "description": "Scaffolding for n8n workflow projects",
  "category": "automation",
  "tags": ["n8n", "workflow", "automation", "no-code"],
  
  "detection": {
    "files": ["*.n8n.json", "workflows/*.json"],
    "directories": [".n8n", "workflows"],
    "patterns": ["n8n-nodes-base", "n8n-workflow"],
    "keywords": ["n8n", "workflow automation"]
  },
  
  "mcp": {
    "servers": [
      {
        "name": "n8n-mcp",
        "package": "n8n-mcp",
        "required": true,
        "config": {
          "N8N_API_URL": "${env:N8N_API_URL}",
          "N8N_API_KEY": "${env:N8N_API_KEY}"
        }
      }
    ],
    "skills": [
      {
        "name": "n8n-skills",
        "repository": "https://github.com/czlonkowski/n8n-skills",
        "required": false
      }
    ]
  },
  
  "templates": [
    {
      "id": "basic-workflow",
      "name": "Basic Workflow Project",
      "description": "Simple n8n workflow with webhook trigger"
    },
    {
      "id": "ai-agent",
      "name": "AI Agent Workflow",
      "description": "n8n workflow with LangChain AI agents"
    }
  ],
  
  "requiredTools": ["node", "npm"],
  "optionalTools": ["docker", "docker-compose"]
}
```

### 2.3 Base Analyzer Interface

```typescript
// core/analyzer/interfaces.ts

export interface ProjectContext {
  path: string;
  files: string[];
  directories: string[];
  packageJson?: Record<string, any>;
  readme?: string;
  existingConfig?: Record<string, any>;
}

export interface AnalysisResult {
  projectType: string;
  confidence: number;  // 0-100
  subType?: string;
  detectedFeatures: string[];
  suggestedTemplates: string[];
  warnings: string[];
  mcpServersNeeded: string[];
}

export interface ScaffoldOptions {
  template: string;
  projectName: string;
  outputPath: string;
  variables: Record<string, any>;
  mcpConfig?: MCPConfig;
}

export interface ProjectAnalyzer {
  readonly pluginId: string;
  readonly category: string;
  
  // Detection
  canHandle(context: ProjectContext): boolean;
  analyze(context: ProjectContext): Promise<AnalysisResult>;
  
  // Scaffolding
  getTemplates(): TemplateInfo[];
  scaffold(options: ScaffoldOptions): Promise<void>;
  
  // AI Guidance
  getSkillPath(): string;
  getPromptContext(): string;
}
```

---

## Part 3: n8n Project Type Implementation

### 3.1 Plugin Structure

```
plugins/n8n/
├── plugin.json
├── analyzer.ts
├── SKILL.md
├── mcp-config.json
├── templates/
│   ├── basic-workflow/
│   │   ├── .n8n/
│   │   │   └── config.json
│   │   ├── workflows/
│   │   │   └── example-workflow.json
│   │   ├── .claude/
│   │   │   └── settings.local.json  # n8n-mcp pre-configured
│   │   ├── docker-compose.yml
│   │   ├── .env.example
│   │   └── README.md
│   │
│   ├── ai-agent-workflow/
│   │   ├── workflows/
│   │   │   └── ai-agent.json
│   │   ├── credentials/
│   │   │   └── .gitkeep
│   │   └── README.md
│   │
│   └── production-setup/
│       ├── docker/
│       ├── nginx/
│       ├── monitoring/
│       └── README.md
│
└── evaluations/
    ├── basic-workflow.eval.md
    └── ai-agent.eval.md
```

### 3.2 n8n SKILL.md

```markdown
---
name: n8n-project-scaffolder
description: "AI-guided scaffolding for n8n workflow automation projects"
activation:
  - "n8n project"
  - "workflow automation"
  - "n8n workflow"
  - "create n8n"
dependencies:
  mcp_servers:
    - n8n-mcp
  skills:
    - n8n-skills (optional)
---

# n8n Project Scaffolding Skill

## Overview
This skill guides Claude in creating well-structured n8n workflow projects
with proper MCP integration for AI-assisted workflow development.

## Prerequisites Check

Before scaffolding, verify:
1. n8n-mcp server is configured (check `.mcp.json` or Claude settings)
2. n8n instance is accessible (if deploying workflows)
3. Required credentials are available

## Project Types

### 1. Basic Workflow Project
**Use when:** Starting with n8n, simple automation needs
**Includes:**
- Pre-configured docker-compose.yml
- Example webhook workflow
- Environment template
- Claude settings with n8n-mcp

### 2. AI Agent Workflow Project  
**Use when:** Building AI-powered automations
**Includes:**
- LangChain node configurations
- AI tool integration patterns
- Memory and context management
- Streaming configuration

### 3. Production Setup
**Use when:** Deploying n8n at scale
**Includes:**
- Nginx reverse proxy
- SSL configuration
- Monitoring stack (Prometheus/Grafana)
- Backup scripts
- High-availability patterns

## MCP Integration

### Required: n8n-mcp Server
The scaffolder automatically configures n8n-mcp in the project's
`.claude/settings.local.json`:

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "npx",
      "args": ["n8n-mcp"],
      "env": {
        "MCP_MODE": "stdio",
        "N8N_API_URL": "${N8N_API_URL}",
        "N8N_API_KEY": "${N8N_API_KEY}"
      }
    }
  }
}
```

### Optional: n8n-skills Integration
For enhanced AI workflow building, install n8n-skills:
```bash
/plugin install czlonkowski/n8n-skills
```

## Scaffolding Workflow

1. **Detection Phase**
   - Check for existing n8n configurations
   - Identify project sub-type
   - Detect existing workflows

2. **Configuration Phase**  
   - Prompt for n8n instance URL
   - Verify API key availability
   - Select template variant

3. **Generation Phase**
   - Create directory structure
   - Generate workflow templates
   - Configure MCP servers
   - Set up Docker environment

4. **Validation Phase**
   - Test n8n-mcp connectivity
   - Validate workflow JSON syntax
   - Check credential placeholders

## Template Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PROJECT_NAME` | Project directory name | Required |
| `N8N_VERSION` | n8n Docker image tag | `latest` |
| `N8N_PORT` | Local n8n port | `5678` |
| `N8N_PROTOCOL` | http or https | `http` |
| `ENABLE_MONITORING` | Include Prometheus/Grafana | `false` |

## Common Patterns

### Webhook + Processing + Notification
```
Webhook Trigger → Code Node → IF Node → Slack/Email
```

### AI Agent Pattern
```
Chat Trigger → AI Agent → [Tool 1, Tool 2, ...] → Response
```

### Scheduled ETL
```
Schedule Trigger → HTTP Request → Transform → Database
```

## Error Handling

If n8n-mcp is not available:
1. Scaffold with documentation-only mode
2. Include manual setup instructions
3. Provide curl commands for testing
```

### 3.3 n8n Analyzer Implementation

```typescript
// plugins/n8n/analyzer.ts

import { ProjectAnalyzer, ProjectContext, AnalysisResult, ScaffoldOptions } from '../../core/analyzer/interfaces';

export class N8nAnalyzer implements ProjectAnalyzer {
  readonly pluginId = 'n8n';
  readonly category = 'automation';

  canHandle(context: ProjectContext): boolean {
    // Check for n8n-specific files
    const n8nIndicators = [
      context.files.some(f => f.endsWith('.n8n.json')),
      context.directories.includes('.n8n'),
      context.directories.includes('workflows'),
      this.hasN8nWorkflowContent(context)
    ];
    
    return n8nIndicators.some(Boolean);
  }

  async analyze(context: ProjectContext): Promise<AnalysisResult> {
    const features: string[] = [];
    const warnings: string[] = [];
    let subType = 'basic';
    
    // Detect existing workflows
    const workflowFiles = context.files.filter(f => 
      f.includes('workflow') && f.endsWith('.json')
    );
    
    if (workflowFiles.length > 0) {
      features.push(`${workflowFiles.length} existing workflows`);
    }
    
    // Detect AI/LangChain usage
    if (this.hasLangChainNodes(context)) {
      subType = 'ai-agent';
      features.push('AI Agent workflows detected');
    }
    
    // Detect Docker setup
    if (context.files.includes('docker-compose.yml')) {
      features.push('Docker configuration present');
    } else {
      warnings.push('No Docker setup - recommend adding docker-compose.yml');
    }
    
    // Check for credentials
    if (!context.files.some(f => f.includes('.env'))) {
      warnings.push('No .env file - credentials not configured');
    }

    return {
      projectType: 'n8n',
      confidence: this.calculateConfidence(context),
      subType,
      detectedFeatures: features,
      suggestedTemplates: this.suggestTemplates(subType),
      warnings,
      mcpServersNeeded: ['n8n-mcp']
    };
  }

  private hasN8nWorkflowContent(context: ProjectContext): boolean {
    // Check package.json or other files for n8n references
    if (context.packageJson) {
      const deps = {
        ...context.packageJson.dependencies,
        ...context.packageJson.devDependencies
      };
      return Object.keys(deps).some(d => d.includes('n8n'));
    }
    return false;
  }

  private hasLangChainNodes(context: ProjectContext): boolean {
    // Would parse workflow JSON files to detect LangChain nodes
    // Simplified check here
    return context.files.some(f => f.includes('ai-agent'));
  }

  private calculateConfidence(context: ProjectContext): number {
    let confidence = 0;
    
    if (context.directories.includes('.n8n')) confidence += 40;
    if (context.directories.includes('workflows')) confidence += 30;
    if (context.files.some(f => f.endsWith('.n8n.json'))) confidence += 30;
    
    return Math.min(confidence, 100);
  }

  private suggestTemplates(subType: string): string[] {
    switch (subType) {
      case 'ai-agent':
        return ['ai-agent-workflow', 'production-setup'];
      default:
        return ['basic-workflow', 'production-setup'];
    }
  }

  getTemplates() {
    return [
      {
        id: 'basic-workflow',
        name: 'Basic Workflow',
        description: 'Simple n8n project with webhook example'
      },
      {
        id: 'ai-agent-workflow',
        name: 'AI Agent Workflow',
        description: 'n8n with LangChain AI agents'
      },
      {
        id: 'production-setup',
        name: 'Production Setup',
        description: 'Full production n8n environment'
      }
    ];
  }

  async scaffold(options: ScaffoldOptions): Promise<void> {
    // Implementation would copy templates and process variables
  }

  getSkillPath(): string {
    return './SKILL.md';
  }

  getPromptContext(): string {
    return `
This is an n8n workflow automation project.
Use n8n-mcp tools for workflow management.
Key nodes: webhook, httpRequest, code, if, switch.
Always validate workflows before deployment.
    `;
  }
}
```

---

## Part 4: Biotech Project Type Implementation

### 4.1 Plugin Structure

```
plugins/biotech/
├── plugin.json
├── analyzer.ts
├── SKILL.md
├── templates/
│   ├── clinical-development/
│   │   ├── program-plan/
│   │   │   ├── README.md
│   │   │   ├── templates/
│   │   │   │   ├── clinical-development-plan.md
│   │   │   │   ├── study-synopsis-template.md
│   │   │   │   ├── timeline-gantt.xlsx
│   │   │   │   └── budget-template.xlsx
│   │   │   └── checklists/
│   │   │       ├── ind-submission.md
│   │   │       ├── phase-transition.md
│   │   │       └── regulatory-milestones.md
│   │   │
│   │   └── regulatory/
│   │       ├── ind-package/
│   │       ├── nda-package/
│   │       └── amendments/
│   │
│   ├── cro-selection/
│   │   ├── rfp-template.md
│   │   ├── vendor-scorecard.xlsx
│   │   ├── contract-checklist.md
│   │   ├── site-selection-matrix.xlsx
│   │   └── oversight-plan.md
│   │
│   ├── patient-recruitment/
│   │   ├── recruitment-strategy.md
│   │   ├── site-materials/
│   │   │   ├── informed-consent-template.md
│   │   │   ├── patient-brochure.md
│   │   │   └── screening-log.xlsx
│   │   ├── digital-strategy/
│   │   │   ├── social-media-plan.md
│   │   │   └── landing-page-content.md
│   │   └── retention-plan.md
│   │
│   ├── data-management/
│   │   ├── data-management-plan.md
│   │   ├── crf-design-guidelines.md
│   │   ├── edit-check-specifications.md
│   │   ├── sap-template.md
│   │   └── database-lock-checklist.md
│   │
│   └── brand-launch/
│       ├── brand-strategy.md
│       ├── positioning-statement.md
│       ├── key-messages.md
│       ├── launch-timeline.xlsx
│       └── kol-engagement-plan.md
│
└── knowledge/
    ├── ich-guidelines-reference.md
    ├── fda-regulations-overview.md
    └── glossary.md
```

### 4.2 Biotech SKILL.md

```markdown
---
name: biotech-project-scaffolder
description: "AI-guided scaffolding for biotech and life sciences projects"
activation:
  - "clinical development"
  - "biotech project"
  - "pharmaceutical"
  - "clinical trial"
  - "drug development"
  - "CRO selection"
  - "patient recruitment"
  - "brand launch"
---

# Biotech/Life Sciences Project Scaffolding

## Overview
This skill provides structured templates and guidance for pharmaceutical
and biotech project planning, following ICH guidelines and industry
best practices.

## ⚠️ Important Disclaimer
These templates are starting points for planning purposes only.
All documents must be reviewed by qualified regulatory, legal, and
medical professionals before use in actual clinical development.

## Project Types

### 1. Clinical Development Program Planning
**Use for:** IND preparation, Phase I-III planning, regulatory strategy

**Key Documents:**
- Clinical Development Plan (CDP)
- Study Synopsis Templates
- Regulatory Milestone Tracker
- Budget & Resource Planning

**Regulatory Framework:**
- ICH E6(R2) - Good Clinical Practice
- ICH E8(R1) - General Considerations for Clinical Studies
- 21 CFR Parts 11, 50, 56, 312

### 2. CRO Selection & Study Execution
**Use for:** Vendor selection, study start-up, site management

**Key Documents:**
- Request for Proposal (RFP) Template
- Vendor Scorecard & Evaluation Matrix
- Contract Negotiation Checklist
- Site Selection Criteria
- Study Oversight Plan

**Considerations:**
- Therapeutic area expertise
- Geographic coverage
- Technology platforms
- Quality track record
- Cost structure

### 3. Patient Recruitment & Retention
**Use for:** Enrollment optimization, retention strategies

**Key Documents:**
- Recruitment Strategy Template
- Digital Marketing Plan
- Site Support Materials
- Informed Consent Templates
- Retention Program Design

**Key Metrics:**
- Screen-to-enroll ratio
- Enrollment velocity
- Retention rate
- Site activation timeline

### 4. Data Management & Biostatistics
**Use for:** Study data planning, analysis strategy

**Key Documents:**
- Data Management Plan (DMP)
- CRF Design Guidelines
- Edit Check Specifications
- Statistical Analysis Plan (SAP)
- Database Lock Procedures

**Standards:**
- CDISC CDASH/SDTM
- ADaM datasets
- Define.xml specifications

### 5. Brand Strategy & Launch Planning
**Use for:** Commercial preparation, launch excellence

**Key Documents:**
- Brand Strategy Framework
- Positioning & Messaging
- Launch Timeline
- KOL Engagement Plan
- Market Access Strategy

## Template Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `COMPOUND_NAME` | Drug/compound identifier | ABC-123 |
| `INDICATION` | Target disease/condition | Type 2 Diabetes |
| `PHASE` | Development phase | Phase 2b |
| `SPONSOR` | Company name | Acme Pharma |
| `THERAPEUTIC_AREA` | TA classification | Metabolic |

## Scaffolding Workflow

1. **Project Definition**
   - Identify project type
   - Define scope and phase
   - Establish timeline constraints

2. **Template Selection**
   - Choose relevant templates
   - Customize for indication
   - Add company-specific sections

3. **Document Generation**
   - Create folder structure
   - Generate document shells
   - Include reference materials

4. **Integration Points**
   - Link related documents
   - Set up cross-references
   - Configure review workflows

## Quality Considerations

### Document Control
- Version control required
- Change history tracking
- Approval workflows
- Archive requirements

### Compliance
- 21 CFR Part 11 (electronic records)
- GDPR (patient data)
- HIPAA (US patient privacy)
- ICH guidelines
```

### 4.3 Biotech Analyzer

```typescript
// plugins/biotech/analyzer.ts

export class BiotechAnalyzer implements ProjectAnalyzer {
  readonly pluginId = 'biotech';
  readonly category = 'life-sciences';

  private readonly BIOTECH_KEYWORDS = [
    'clinical', 'trial', 'protocol', 'ind', 'nda', 'fda', 'ema',
    'cro', 'patient', 'recruitment', 'enrollment', 'regulatory',
    'pharmaceutical', 'biotech', 'drug', 'compound', 'therapeutic',
    'cdisc', 'sdtm', 'adam', 'sap', 'crf', 'informed consent'
  ];

  private readonly SUBTYPE_INDICATORS: Record<string, string[]> = {
    'clinical-development': ['protocol', 'ind', 'cdp', 'clinical development'],
    'cro-selection': ['cro', 'vendor', 'rfp', 'site selection'],
    'patient-recruitment': ['recruitment', 'enrollment', 'retention', 'patient'],
    'data-management': ['data management', 'crf', 'sdtm', 'sap', 'biostatistics'],
    'brand-launch': ['brand', 'launch', 'commercial', 'marketing', 'positioning']
  };

  canHandle(context: ProjectContext): boolean {
    const textContent = this.getSearchableContent(context);
    return this.BIOTECH_KEYWORDS.some(kw => 
      textContent.toLowerCase().includes(kw)
    );
  }

  async analyze(context: ProjectContext): Promise<AnalysisResult> {
    const textContent = this.getSearchableContent(context);
    const detectedSubTypes = this.detectSubTypes(textContent);
    
    return {
      projectType: 'biotech',
      confidence: this.calculateConfidence(textContent),
      subType: detectedSubTypes[0] || 'clinical-development',
      detectedFeatures: this.detectFeatures(context),
      suggestedTemplates: detectedSubTypes,
      warnings: this.generateWarnings(context),
      mcpServersNeeded: [] // Biotech typically doesn't need MCP
    };
  }

  private detectSubTypes(content: string): string[] {
    const detected: string[] = [];
    const lowerContent = content.toLowerCase();
    
    for (const [subType, indicators] of Object.entries(this.SUBTYPE_INDICATORS)) {
      if (indicators.some(ind => lowerContent.includes(ind))) {
        detected.push(subType);
      }
    }
    
    return detected.length > 0 ? detected : ['clinical-development'];
  }

  private getSearchableContent(context: ProjectContext): string {
    return [
      context.readme || '',
      context.files.join(' '),
      context.directories.join(' ')
    ].join(' ');
  }

  private calculateConfidence(content: string): number {
    const lowerContent = content.toLowerCase();
    const matchCount = this.BIOTECH_KEYWORDS.filter(kw => 
      lowerContent.includes(kw)
    ).length;
    
    return Math.min(matchCount * 15, 100);
  }

  private detectFeatures(context: ProjectContext): string[] {
    const features: string[] = [];
    
    if (context.files.some(f => f.includes('protocol'))) {
      features.push('Protocol documents present');
    }
    if (context.directories.includes('regulatory')) {
      features.push('Regulatory folder structure');
    }
    
    return features;
  }

  private generateWarnings(context: ProjectContext): string[] {
    const warnings: string[] = [];
    
    warnings.push('⚠️ Templates require review by qualified professionals');
    warnings.push('⚠️ Ensure compliance with applicable regulations');
    
    return warnings;
  }

  getTemplates() {
    return [
      { id: 'clinical-development', name: 'Clinical Development', description: 'CDP, protocols, regulatory' },
      { id: 'cro-selection', name: 'CRO Selection', description: 'RFP, vendor management' },
      { id: 'patient-recruitment', name: 'Patient Recruitment', description: 'Enrollment strategies' },
      { id: 'data-management', name: 'Data Management', description: 'DMP, CRF, SAP' },
      { id: 'brand-launch', name: 'Brand Launch', description: 'Commercial strategy' }
    ];
  }

  getSkillPath(): string {
    return './SKILL.md';
  }

  getPromptContext(): string {
    return `
This is a biotech/pharmaceutical project.
Follow ICH guidelines and regulatory requirements.
All documents are templates requiring professional review.
Consider compliance, patient safety, and data integrity.
    `;
  }
}
```

---

## Part 5: Miscellaneous Projects Implementation

### 5.1 Plugin Structure

```
plugins/misc/
├── plugin.json
├── analyzer.ts
├── SKILL.md
├── templates/
│   ├── home-improvement/
│   │   ├── README.md
│   │   ├── project-plan.md
│   │   ├── budget-tracker.xlsx
│   │   ├── materials-list.md
│   │   ├── contractor-checklist.md
│   │   ├── permit-tracker.md
│   │   └── photo-log/
│   │       └── .gitkeep
│   │
│   ├── personal-project/
│   │   ├── README.md
│   │   ├── goals.md
│   │   ├── timeline.md
│   │   ├── resources.md
│   │   └── notes/
│   │
│   └── _custom/
│       ├── README.md
│       └── template-generator.md
│
└── schemas/
    └── custom-project-schema.json
```

### 5.2 Miscellaneous SKILL.md

```markdown
---
name: misc-project-scaffolder
description: "Flexible scaffolding for non-software projects"
activation:
  - "home improvement"
  - "renovation"
  - "personal project"
  - "planning"
  - "organize project"
---

# Miscellaneous Project Scaffolding

## Overview
This skill handles non-software projects that don't fit other categories.
It provides flexible templates that can be customized for any project type.

## Project Types

### 1. Home Improvement Projects
**Use for:** Renovations, repairs, upgrades, landscaping

**Standard Structure:**
```
project-name/
├── README.md           # Project overview
├── 01-planning/
│   ├── goals.md        # What you want to achieve
│   ├── budget.xlsx     # Cost tracking
│   └── timeline.md     # Schedule & milestones
├── 02-research/
│   ├── materials.md    # Materials research
│   ├── contractors.md  # Contractor info
│   └── permits.md      # Permit requirements
├── 03-execution/
│   ├── daily-log.md    # Progress tracking
│   └── issues.md       # Problems & solutions
├── 04-completion/
│   ├── final-costs.xlsx
│   ├── lessons-learned.md
│   └── warranty-info.md
└── photos/
    ├── before/
    ├── during/
    └── after/
```

### 2. Custom Projects
**Use for:** Anything else that needs structure

**Flexible Template Variables:**
| Variable | Description |
|----------|-------------|
| `PROJECT_NAME` | Project identifier |
| `PROJECT_TYPE` | Category description |
| `START_DATE` | Planned start |
| `END_DATE` | Target completion |
| `BUDGET` | Total budget |

## Scaffolding Workflow

1. **Project Discovery**
   - Ask about project type
   - Identify key phases
   - Determine tracking needs

2. **Structure Selection**
   - Use predefined template OR
   - Build custom structure

3. **Template Customization**
   - Adjust phases
   - Add/remove sections
   - Configure tracking

## Home Improvement Specific

### Budget Categories
- Materials
- Labor/Contractors
- Permits/Fees
- Tools/Equipment
- Contingency (10-20%)

### Timeline Phases
1. Planning & Design
2. Permits & Approvals
3. Material Procurement
4. Demolition (if needed)
5. Construction/Installation
6. Finishing
7. Inspection & Cleanup

### Documentation Best Practices
- Photo everything before starting
- Keep all receipts
- Document contractor agreements
- Track permit status
- Note warranty information
```

---

## Part 6: Self-Extension System (Future-Proofing)

### 6.1 Plugin Creation Command

Create `.claude/commands/add-project-type.md`:

```markdown
---
name: add-project-type
description: Create a new project type plugin for the scaffolder
---

# Add New Project Type

## Usage
```
/add-project-type [name] [category]
```

## Process

### Step 1: Gather Information
Ask the user:
1. What type of projects does this cover?
2. What files/directories indicate this project type?
3. What templates should be included?
4. Are there any MCP servers needed?
5. What's the typical workflow for this project type?

### Step 2: Create Plugin Structure
Generate in `plugins/[name]/`:

```
[name]/
├── plugin.json          # Plugin manifest
├── analyzer.ts          # Detection logic
├── SKILL.md             # AI guidance
└── templates/
    └── default/
        └── README.md
```

### Step 3: Generate Plugin Manifest
Create `plugin.json` with:
- Detection patterns from user input
- Template definitions
- Required tools/MCP servers
- Category and tags

### Step 4: Create SKILL.md
Generate skill document including:
- Project type overview
- Available templates
- Typical workflows
- Best practices
- Common pitfalls

### Step 5: Register Plugin
Add to `core/analyzer/analyzer-registry.ts`

### Step 6: Test
Run analyzer against sample project to verify detection.
```

### 6.2 Self-Documenting Plugin Schema

```json
// plugins/_plugin-template/plugin-schema.json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Project Scaffolder Plugin",
  "type": "object",
  "required": ["name", "version", "displayName", "detection"],
  "properties": {
    "name": {
      "type": "string",
      "description": "Unique plugin identifier (lowercase, hyphenated)"
    },
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "displayName": {
      "type": "string",
      "description": "Human-readable name"
    },
    "description": {
      "type": "string"
    },
    "category": {
      "type": "string",
      "enum": ["software", "automation", "life-sciences", "business", "personal", "other"]
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" }
    },
    "detection": {
      "type": "object",
      "properties": {
        "files": {
          "type": "array",
          "items": { "type": "string" },
          "description": "File patterns that indicate this project type"
        },
        "directories": {
          "type": "array",
          "items": { "type": "string" }
        },
        "patterns": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Text patterns to search for in files"
        },
        "keywords": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Keywords in project description/readme"
        }
      }
    },
    "mcp": {
      "type": "object",
      "properties": {
        "servers": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "name": { "type": "string" },
              "package": { "type": "string" },
              "required": { "type": "boolean" },
              "config": { "type": "object" }
            }
          }
        },
        "skills": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "name": { "type": "string" },
              "repository": { "type": "string" },
              "required": { "type": "boolean" }
            }
          }
        }
      }
    },
    "templates": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "name"],
        "properties": {
          "id": { "type": "string" },
          "name": { "type": "string" },
          "description": { "type": "string" }
        }
      }
    },
    "requiredTools": {
      "type": "array",
      "items": { "type": "string" }
    },
    "optionalTools": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

---

## Part 7: Updated Master SKILL.md

Update `skills/project-setup/SKILL.md`:

```markdown
---
name: project-scaffolder
description: "Intelligent project scaffolding with plugin-based extensibility"
activation:
  - "new project"
  - "scaffold"
  - "create project"
  - "setup project"
  - "initialize"
---

# Project Scaffolder

## Overview
An extensible project scaffolding system that automatically detects project 
types and provides appropriate templates, configuration, and AI guidance.

## Supported Project Types

### Software Development
| Type | Detection | MCP Integration |
|------|-----------|-----------------|
| Node.js | package.json, node_modules | - |
| Python | requirements.txt, setup.py, pyproject.toml | - |
| Rust | Cargo.toml | - |
| Go | go.mod | - |
| Java | pom.xml, build.gradle | - |
| Ruby | Gemfile | - |

### Automation
| Type | Detection | MCP Integration |
|------|-----------|-----------------|
| n8n | workflows/*.json, .n8n/ | n8n-mcp (required), n8n-skills (optional) |

### Life Sciences
| Type | Detection | MCP Integration |
|------|-----------|-----------------|
| Clinical Development | protocol, IND, CDP keywords | - |
| CRO Selection | RFP, vendor, CRO keywords | - |
| Patient Recruitment | recruitment, enrollment keywords | - |
| Data Management | DMP, CRF, SAP keywords | - |
| Brand Launch | brand, launch, commercial keywords | - |

### Personal/Miscellaneous
| Type | Detection | MCP Integration |
|------|-----------|-----------------|
| Home Improvement | renovation, contractor keywords | - |
| Custom | User-defined | Optional |

## Scaffolding Workflow

### 1. Analysis Phase
```
1. Scan project directory (or get user description for new project)
2. Run all registered analyzers
3. Rank results by confidence score
4. Present findings to user
```

### 2. Selection Phase
```
1. Confirm detected project type (or let user override)
2. Show available templates
3. Gather template variables
4. Check MCP server requirements
```

### 3. Generation Phase
```
1. Create directory structure
2. Process template files
3. Configure MCP servers (if needed)
4. Generate documentation
```

### 4. Validation Phase
```
1. Verify file generation
2. Test MCP connectivity (if applicable)
3. Run project-specific validations
4. Present summary
```

## Adding New Project Types

Use the `/add-project-type` command to create new plugins:

```
/add-project-type [name] [category]
```

This will guide you through creating:
- Plugin manifest (plugin.json)
- Analyzer implementation
- SKILL.md for AI guidance
- Template directory structure

## Plugin Architecture

Each project type is a self-contained plugin in `plugins/[name]/`:

```
[name]/
├── plugin.json      # Manifest with detection rules
├── analyzer.ts      # TypeScript analyzer
├── SKILL.md         # AI guidance document
└── templates/       # Project templates
```

## MCP Integration

Some project types benefit from MCP server integration:

### n8n Projects
Automatically configures n8n-mcp for workflow management:
- Workflow creation and validation
- Node documentation access
- Template deployment

### Custom MCP
Plugins can specify custom MCP servers in plugin.json.

## Best Practices

### For Users
1. Run scaffolder in target directory
2. Review detected project type
3. Choose appropriate template
4. Configure variables carefully
5. Review generated files before committing

### For Plugin Developers
1. Make detection specific (avoid false positives)
2. Provide clear SKILL.md documentation
3. Include sensible default templates
4. Document any MCP requirements
5. Add validation where appropriate
```

---

## Part 8: Implementation Roadmap

### Phase 1: Core Architecture (Week 1-2)
- [ ] Create plugin interface definitions
- [ ] Build analyzer registry
- [ ] Implement plugin loader
- [ ] Update master SKILL.md

### Phase 2: n8n Plugin (Week 2-3)
- [ ] Create n8n plugin structure
- [ ] Implement n8n analyzer
- [ ] Build n8n templates
- [ ] Test MCP integration
- [ ] Write SKILL.md

### Phase 3: Biotech Plugin (Week 3-4)
- [ ] Create biotech plugin structure
- [ ] Implement biotech analyzer
- [ ] Build all template categories
- [ ] Add regulatory references
- [ ] Write SKILL.md

### Phase 4: Misc Plugin (Week 4)
- [ ] Create misc plugin structure
- [ ] Implement flexible analyzer
- [ ] Build home improvement templates
- [ ] Create custom project generator

### Phase 5: Self-Extension System (Week 5)
- [ ] Create plugin template
- [ ] Build `/add-project-type` command
- [ ] Create plugin schema documentation
- [ ] Test plugin creation workflow

### Phase 6: Documentation & Testing (Week 6)
- [ ] Complete all documentation
- [ ] Create evaluation tests
- [ ] User testing
- [ ] Bug fixes and refinements

---

## Conclusion

This architecture transforms the project-scaffolder from a static tool into an 
extensible platform. Key benefits:

1. **Modularity** - Each project type is self-contained
2. **Extensibility** - Users can add new types without core changes
3. **AI-Native** - Each plugin has its own SKILL.md for optimal AI guidance
4. **MCP-Ready** - Built-in support for MCP server integration
5. **Future-Proof** - Schema-driven plugin system allows endless expansion

The n8n-mcp and n8n-skills integration provides a powerful example of how 
domain-specific tools can be seamlessly incorporated into the scaffolding 
workflow.
