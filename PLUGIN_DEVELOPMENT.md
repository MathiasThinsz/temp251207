# Adding New Project Types to the Scaffolder

## A Guide for Future Extensions

This guide explains how to add support for project types that don't exist yet - ones you haven't even thought of.

---

## Understanding the Plugin System

Every project type in the scaffolder is a **plugin** - a self-contained package that includes:

1. **Detection Logic** - How to recognize this project type
2. **Templates** - Starter files and folder structures
3. **AI Guidance** - Instructions for Claude via SKILL.md
4. **Configuration** - Variables, tools, and MCP integrations

---

## Quick Start: Creating a New Plugin

### Step 1: Copy the Template

```bash
cp -r plugins/_plugin-template plugins/your-project-type
```

### Step 2: Edit the Manifest

Edit `plugins/your-project-type/plugin.json`:

```json
{
  "name": "your-project-type",
  "version": "1.0.0",
  "displayName": "Your Project Type",
  "description": "What this project type is for",
  "category": "choose-category",
  "tags": ["relevant", "keywords"],
  
  "detection": {
    "files": ["patterns-that-identify-this-type"],
    "directories": ["folder-names-to-look-for"],
    "keywords": ["words-in-readme-or-description"]
  },
  
  "templates": [
    {
      "id": "default",
      "name": "Default Template",
      "description": "Standard starting point"
    }
  ]
}
```

### Step 3: Create the SKILL.md

Create `plugins/your-project-type/SKILL.md`:

```markdown
---
name: your-project-type-scaffolder
description: "Brief description of what this skill does"
activation:
  - "trigger phrase 1"
  - "trigger phrase 2"
---

# Your Project Type Scaffolding

## Overview
Explain what this project type is and when to use it.

## Templates
List and describe available templates.

## Workflow
Explain the typical scaffolding workflow.

## Best Practices
Share domain-specific best practices.
```

### Step 4: Add Templates

Create template directories:

```
plugins/your-project-type/templates/
└── default/
    ├── README.md
    ├── {{PROJECT_NAME}}/        # Variable substitution
    │   └── ...
    └── ...
```

### Step 5: Register the Plugin

The plugin loader will automatically discover plugins in the `plugins/` directory.

---

## Detailed Plugin Schema

### Categories

Choose from these categories or add new ones:

| Category | For |
|----------|-----|
| `software` | Programming/development projects |
| `automation` | Workflow automation, integrations |
| `life-sciences` | Biotech, pharma, clinical |
| `business` | Business planning, strategy |
| `personal` | Personal projects, hobbies |
| `creative` | Writing, design, media |
| `education` | Learning, courses, research |
| `other` | Anything else |

### Detection Strategies

**File-based detection:**
```json
"detection": {
  "files": [
    "package.json",           // Exact filename
    "*.py",                   // Glob pattern
    "src/**/*.tsx"           // Deep glob
  ]
}
```

**Directory-based detection:**
```json
"detection": {
  "directories": [
    "node_modules",
    ".git",
    "src"
  ]
}
```

**Content pattern detection:**
```json
"detection": {
  "patterns": [
    "from fastapi import",    // Code patterns
    "\"dependencies\":",      // JSON patterns
    "## Installation"         // Markdown patterns
  ]
}
```

**Keyword detection (for new projects):**
```json
"detection": {
  "keywords": [
    "react app",
    "web application",
    "frontend"
  ]
}
```

### Template Variables

Define variables users can customize:

```json
"variables": {
  "PROJECT_NAME": {
    "type": "string",
    "description": "Name of the project",
    "required": true,
    "pattern": "^[a-z][a-z0-9-]*$"
  },
  "USE_TYPESCRIPT": {
    "type": "boolean",
    "description": "Use TypeScript instead of JavaScript",
    "default": true
  },
  "FRAMEWORK": {
    "type": "string",
    "description": "Which framework to use",
    "enum": ["react", "vue", "svelte"],
    "default": "react"
  },
  "PORT": {
    "type": "number",
    "description": "Development server port",
    "default": 3000,
    "min": 1024,
    "max": 65535
  }
}
```

### MCP Integration

If your project type benefits from AI tools, add MCP configuration:

```json
"mcp": {
  "servers": [
    {
      "name": "your-mcp-server",
      "package": "your-mcp-package",
      "required": false,
      "description": "What this MCP server provides",
      "config": {
        "API_URL": "${env:YOUR_API_URL}",
        "API_KEY": "${env:YOUR_API_KEY}"
      }
    }
  ],
  "skills": [
    {
      "name": "related-skills",
      "repository": "https://github.com/owner/skills-repo",
      "required": false
    }
  ]
}
```

---

## Writing Good SKILL.md Files

### Structure

```markdown
---
name: plugin-name-scaffolder
description: "One-line description"
activation:
  - "phrase that should trigger this skill"
  - "another trigger phrase"
dependencies:
  mcp_servers:
    required: []
    optional: []
---

# Title

## Overview
What is this project type? When should someone use it?

## Templates
What templates are available? What's in each one?

## Variables
What can be customized? What do the options mean?

## Workflow
Step-by-step scaffolding process.

## Best Practices
Domain-specific advice and recommendations.

## Common Patterns
Typical structures or approaches.

## Troubleshooting
How to resolve common issues.
```

### Writing Tips

1. **Be specific** - Don't write generic advice; make it relevant to this project type
2. **Include examples** - Show directory structures, code snippets, configurations
3. **Anticipate questions** - What will users wonder about?
4. **Link to resources** - Point to external documentation when appropriate
5. **Keep it scannable** - Use headers, tables, and code blocks

---

## Examples of Different Project Types

### Example 1: Event Planning Plugin

```json
{
  "name": "event-planning",
  "displayName": "Event Planning",
  "category": "business",
  "detection": {
    "keywords": ["event", "conference", "wedding", "party", "planning"]
  },
  "templates": [
    {"id": "corporate-event", "name": "Corporate Event"},
    {"id": "wedding", "name": "Wedding Planning"},
    {"id": "conference", "name": "Conference"}
  ],
  "variables": {
    "EVENT_NAME": {"type": "string", "required": true},
    "EVENT_DATE": {"type": "string", "required": false},
    "EXPECTED_ATTENDEES": {"type": "number", "default": 100}
  }
}
```

### Example 2: Research Paper Plugin

```json
{
  "name": "research-paper",
  "displayName": "Research Paper",
  "category": "education",
  "detection": {
    "files": ["*.bib", "*.tex"],
    "keywords": ["research", "paper", "thesis", "dissertation"]
  },
  "templates": [
    {"id": "ieee", "name": "IEEE Format"},
    {"id": "apa", "name": "APA Format"},
    {"id": "latex-thesis", "name": "LaTeX Thesis"}
  ],
  "optionalTools": [
    {"name": "latex", "checkCommand": "pdflatex --version"},
    {"name": "bibtex", "checkCommand": "bibtex --version"}
  ]
}
```

### Example 3: Mobile App Plugin

```json
{
  "name": "mobile-app",
  "displayName": "Mobile Application",
  "category": "software",
  "detection": {
    "files": ["app.json", "*.xcodeproj", "*.gradle"],
    "directories": ["ios", "android"],
    "keywords": ["mobile app", "react native", "flutter", "ios", "android"]
  },
  "templates": [
    {"id": "react-native", "name": "React Native"},
    {"id": "flutter", "name": "Flutter"},
    {"id": "native-ios", "name": "Native iOS"}
  ],
  "variables": {
    "APP_NAME": {"type": "string", "required": true},
    "BUNDLE_ID": {"type": "string", "pattern": "^[a-z]+\\.[a-z]+\\.[a-z]+$"}
  }
}
```

---

## Using the /add-project-type Command

For interactive plugin creation, use:

```
/add-project-type [name] [category]
```

This will prompt you through:

1. **Basic Information**
   - Project type name
   - Description
   - Category

2. **Detection Rules**
   - Files that indicate this type
   - Directory patterns
   - Keywords for new projects

3. **Templates**
   - How many templates?
   - What's in each one?

4. **Variables**
   - What should be customizable?
   - Types and defaults

5. **MCP Integration**
   - Any relevant MCP servers?
   - Related skills?

---

## Testing Your Plugin

### Manual Testing

1. Create a test directory matching your detection rules
2. Run the scaffolder and verify detection
3. Test each template generation
4. Verify variable substitution works

### Automated Testing

Create `plugins/your-type/tests/`:

```
tests/
├── detection.test.ts      # Test detection logic
├── templates.test.ts      # Test template generation
└── fixtures/              # Test data
    ├── should-match/
    └── should-not-match/
```

---

## Contributing Back

If you create a useful plugin, consider:

1. **Documenting thoroughly** - Help others understand and use it
2. **Testing completely** - Ensure it works reliably
3. **Submitting a PR** - Share with the community

Your plugin could help thousands of people scaffold projects in a domain you understand well!
