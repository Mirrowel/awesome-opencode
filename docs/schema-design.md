# OpenCode Extension Schema Design

> **Working Document** - This document serves as both a planning guide for implementation and a precursor to official documentation. Content may evolve as implementation progresses.

This document describes the YAML schema for the awesome-opencode registry, a data-driven catalog of OpenCode extensions. It covers extension types, configuration locations, schema fields, and downstream compatibility.

---

## 1. Overview

The awesome-opencode registry is a curated collection of extensions that enhance the OpenCode editor. The registry follows a data-driven approach where YAML files in `data/{category}/` directories serve as the source of truth.

### Design Principles

- **Human Readable**: YAML format ensures contributors can easily read and edit entries
- **Downstream Ready**: Schema designed for seamless consumption by platforms like opencode.cafe
- **Discoverable**: Tags and metadata support filtering and search
- **Scope-Aware**: Clear distinction between global and project-level installations

### Directory Structure

```
awesome-opencode/
├── data/
│   ├── plugins/
│   │   └── *.yaml
│   ├── themes/
│   │   └── *.yaml
│   ├── agents/
│   │   └── *.yaml       # Agent configurations and skills
│   ├── projects/
│   │   └── *.yaml
│   ├── resources/
│   │   └── *.yaml       # Guides, configs, MCP servers, tools, commands
│   └── forks/
│       └── *.yaml
└── docs/
    └── schema-design.md
```

---

## 2. OpenCode Extension Types

Extensions in awesome-opencode fall into distinct categories, each serving a specific purpose in the OpenCode ecosystem.

### 2.1 Plugin (`plugin`)

JavaScript or TypeScript modules that extend OpenCode's core functionality. Plugins can be installed via npm or as local files.

**Installation Methods:**
- NPM package: `npm install <plugin-name>`
- Local path: Reference to local `.js` or `.ts` file

**Example Use Cases:**
- Authentication providers
- Custom UI components and panels
- Integration with external services

### 2.2 Theme (`theme`)

Visual themes that customize the OpenCode editor appearance. Themes are distributed as JSON files containing color schemes and styling rules.

**Installation Methods:**
- JSON file placed in themes directory
- Referenced from remote URL

**Example Use Cases:**
- Dark/light mode variations
- Syntax highlighting for specific languages
- Custom UI color schemes

### 2.3 Agent (`agent`)

Custom AI agent configurations and skills that define behavior, tools, and instructions for specialized tasks. This category includes both agent configurations and reusable skill instruction sets.

**Includes:**
- Agent configurations (system prompts, available tools, behavior)
- Skills (reusable instruction sets in `SKILL.md` format)

**Example Use Cases:**
- Security-focused code reviewers
- Documentation generators
- Interview question generators
- Bug reproduction assistants

### 2.4 Project (`project`)

Standalone applications, utilities, and tools built for or with OpenCode. These are complete projects rather than extensions.

**Example Use Cases:**
- Web interfaces for OpenCode
- Session management tools
- Proxy servers and API wrappers
- Companion applications

### 2.5 Resource (`resource`)

Guides, configurations, MCP servers, tools, and commands that enhance the OpenCode experience without being installable plugins.

**Includes:**
- Configuration starters and examples
- MCP servers (Model Context Protocol servers)
- CLI tools and utilities
- Custom slash commands
- How-to guides and tutorials

**Example Use Cases:**
- Database access MCP servers
- API integrations (GitHub, Slack, etc.)
- Project-specific build commands
- Starter configuration templates

### 2.6 Fork (`fork`)

Modified builds of OpenCode itself with custom features or patches. Forks maintain compatibility while adding functionality.

**Example Use Cases:**
- Custom builds with enterprise features
- Experimental feature branches
- Distribution-specific variants

---

## 3. OpenCode Configuration Locations

OpenCode extensions can be installed in multiple locations depending on their scope and purpose. Understanding these locations is crucial for proper installation and management.

### 3.1 Global Configuration (User-Wide)

Global installations apply to all projects for the current user.

| Path | Purpose |
|------|---------|
| `~/.config/opencode/opencode.json` | Main configuration file |
| `~/.config/opencode/plugin/` | Plugin modules |
| `~/.config/opencode/themes/` | Theme JSON files |
| `~/.config/opencode/command/` | Custom slash commands |
| `~/.config/opencode/agent/` | Custom agent configurations |
| `~/.config/opencode/skill/<name>/SKILL.md` | Global skills |

**Note:** On Windows, the equivalent is `%APPDATA%/opencode/`.

### 3.2 Project-Level Configuration

Project-level installations are scoped to specific repositories.

| Path | Purpose |
|------|---------|
| `<project>/opencode.json` | Project-specific configuration |
| `<project>/.opencode/plugin/` | Project plugins |
| `<project>/.opencode/themes/` | Project themes |
| `<project>/.opencode/command/` | Project commands |
| `<project>/.opencode/agent/` | Project agents |
| `<project>/.opencode/skill/<name>/SKILL.md` | Project skills |

### 3.3 Claude-Compatible Paths

OpenCode maintains compatibility with Claude's skill system for portability.

| Path | Purpose |
|------|---------|
| `~/.claude/skills/<name>/SKILL.md` | Global Claude skills |
| `.claude/skills/<name>/SKILL.md` | Project Claude skills |

### 3.4 Remote and Custom Configurations

Extensions can also be fetched from remote sources or custom locations.

| Path/Variable | Purpose |
|---------------|---------|
| `.well-known/opencode` | Organizational defaults fetched remotely |
| `OPENCODE_CONFIG` | Environment variable pointing to custom config file |
| `OPENCODE_CONFIG_DIR` | Environment variable pointing to custom config directory |

### 3.5 Precedence Order

When multiple configuration sources exist, OpenCode follows this precedence (later sources override earlier ones):

```
1. Remote config (.well-known/opencode)
2. Global config (~/.config/opencode/)
3. Custom config (OPENCODE_CONFIG env var)
4. Project config (opencode.json)
5. Project local directories (.opencode/)
6. Inline config (environment variables)
```

**Resolution Example:**
If `scope` is set to `[global, project]` and both global and project configs define the same extension, the project-level configuration takes precedence for that project.

---

## 4. Current YAML Schema

The current awesome-opencode entries use a minimal schema focused on core metadata.

### Current Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Display name for the extension (human-readable) |
| `repo` | string | Repository URL (HTTPS format, e.g., `https://github.com/user/repo`) |
| `description` | string | Short description (max 100 characters) |

### Current Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `full_description` | string | Extended explanation of functionality |

### Example (Current Format)

```yaml
name: CodeGPT
repo: https://github.com/company/codegpt
tagline: AI-powered code completion and refactoring
description: |
  CodeGPT provides intelligent code completion using GPT models.
  Features include:
  - Real-time suggestions
  - Context-aware refactoring
  - Multi-language support
```

---

## 5. Proposed Schema Additions

The following fields are being added to enhance discoverability, installation guidance, and downstream compatibility.

### New Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | enum | Extension type: `plugin`, `theme`, `agent`, `project`, `resource`, or `fork` |
| `scope` | array | Installation scope: `[global]`, `[project]`, or `[global, project]` |

### New Optional Fields

| Field | Type | Description |
|-------|------|-------------|
| `tags` | array | Strings for filtering and discoverability (e.g., `["ai", "productivity"]`) |
| `min_version` | string | Minimum OpenCode version required (e.g., `"1.0.0"`) |
| `homepage` | string | Documentation URL if different from repository |
| `installation` | markdown | Detailed installation instructions |

### Example (Proposed Format)

```yaml
name: CodeGPT
repo: https://github.com/company/codegpt
type: plugin
scope: [global, project]
tagline: AI-powered code completion and refactoring
description: |
  CodeGPT provides intelligent code completion using GPT models.
  Features include:
  - Real-time suggestions
  - Context-aware refactoring
  - Multi-language support
tags:
  - ai
  - completion
  - productivity
min_version: "1.0.0"
homepage: https://codegpt.dev/docs
installation: |
  ## Prerequisites
  - OpenCode 1.0.0 or later
  - Node.js 18+ for local plugins

  ## Installation
  ### Global
  1. Run `opencode install codegpt`
  2. Restart OpenCode

  ## Configuration
  Add to `opencode.json`:
  ```json
  {
    "plugins": ["codegpt"]
  }
  ```
```

---

## 6. Installation Template

The `installation` field uses a structured template that contributors fill in. Not all sections are required—include what's relevant for the extension.

### Template Structure

```markdown
## Prerequisites
- OpenCode version requirements
- Other dependencies (Node.js, Python, CLI tools, etc.)

## Installation

### Global (all projects)
Steps for installing to ~/.config/opencode/

### Project-only
Steps for installing to .opencode/ within a project

## Configuration
Config options to add to opencode.json

## Files Modified
List of files created/modified (for backup purposes)
- ~/.config/opencode/opencode.json
- ~/.config/opencode/plugin/codegpt.js

## Removal
How to uninstall the extension
```

### Template Guidelines

1. **Prerequisites**: List minimum versions and dependencies
2. **Installation**: Provide both global and project-specific instructions if `scope` includes both
3. **Configuration**: Show exact JSON/config snippets needed
4. **Files Modified**: Help users understand what will change for backup/restore
5. **Removal**: Include cleanup steps for uninstallation

### Section Requirements by Type

| Extension Type | Key Sections |
|----------------|--------------|
| `plugin` | Installation, Configuration |
| `theme` | Installation, Files Modified |
| `agent` | Installation, Configuration |
| `project` | Prerequisites, Installation |
| `resource` | Prerequisites, Installation, Configuration |
| `fork` | Installation, Files Modified |

---

## 7. Downstream Compatibility

The awesome-opencode schema is designed for seamless conversion to downstream platforms like opencode.cafe.

### Field Mapping

| awesome-opencode YAML | opencode.cafe JSON | Notes |
|-----------------------|-------------------|-------|
| `name` | `displayName` | Display name for UI |
| `repo` | `repoUrl` | Repository URL |
| `description` | `description` | Short description |
| `type` | `type` | Mapped directly (see below) |
| `tags` | `tags` | Array of strings |
| `homepage` | `homepageUrl` | Documentation URL |
| `installation` | `installation` | Markdown content |
| (filename) | `productId` | Derived from filename |

### Type Mapping

| awesome-opencode Type | opencode.cafe Type |
|-----------------------|-------------------|
| `plugin` | `plugin` |
| `theme` | `theme` |
| `agent` | `agent` |
| `project` | `project` |
| `resource` | `resource` |
| `fork` | `fork` |

### Conversion Example

**YAML Input (`data/plugins/codegpt.yaml`):**

```yaml
name: CodeGPT
repo: https://github.com/company/codegpt
type: plugin
scope: [global, project]
tagline: AI-powered code completion
```

**JSON Output (opencode.cafe format):**

```json
{
  "productId": "codegpt",
  "displayName": "CodeGPT",
  "repoUrl": "https://github.com/company/codegpt",
  "type": "plugin",
  "tagline": "AI-powered code completion",
  "tags": [],
  "installation": ""
}
```

---

## 8. Future Vision

This section outlines planned features and improvements for the awesome-opencode ecosystem.

### 8.1 Agent-Powered Plugin Manager

A future plugin/agent that uses the `installation` markdown as context to assist with setup. The agent would:
- Parse the installation instructions
- Know which files will be modified (for backup purposes)
- Guide users through the installation interactively
- Handle scope selection based on user preference

This is **not** an automated CLI executor - it's an AI assistant with context about how to install the extension.

### 8.2 Interactive TUI with Scope Selection

A terminal UI that:
- Reads the `scope` field to present appropriate options
- Shows extension metadata and readme preview
- Handles multi-step installations interactively

```
┌─────────────────────────────────────────┐
│  Install Extension                      │
├─────────────────────────────────────────┤
│  Name:     CodeGPT                      │
│  Type:     plugin                       │
│  Scope:    [X] Global  [ ] Project      │
│                                         │
│  [Install]  [Cancel]  [View Readme]     │
└─────────────────────────────────────────┘
```

### 8.3 Automatic Backup and Restore

When an agent assists with installation, it can:
- Read the "Files Modified" section to know what to backup
- Store backups before making changes
- Enable clean removal with rollback capability

### 8.4 Export and Sync Scripts

Scripts to export the registry for downstream platforms:

```bash
# Export to opencode.cafe JSON format
./scripts/export-json.sh

# Export to custom format
./scripts/export.sh --format=custom --output=dist/
```

### 8.5 Contributing Workflow

Future improvements to the contribution process:

1. **Schema Validation**: CI checks YAML validity and required fields
2. **Preview Tool**: Generate opencode.cafe JSON preview for PRs
3. **Category Detection**: Auto-suggest category based on `type` field
4. **Link Checking**: Verify `repo` and `homepage` URLs are accessible

---

## Appendix A: Quick Reference

### Extension Type to Directory Mapping

| Type | Directory | File Extension |
|------|-----------|----------------|
| `plugin` | `data/plugins/` | `.yaml` |
| `theme` | `data/themes/` | `.yaml` |
| `agent` | `data/agents/` | `.yaml` |
| `project` | `data/projects/` | `.yaml` |
| `resource` | `data/resources/` | `.yaml` |
| `fork` | `data/forks/` | `.yaml` |

### Minimal Valid Entry

```yaml
name: My Extension
repo: https://github.com/user/my-extension
type: plugin
scope: [global]
tagline: A brief description
```

### Complete Entry (All Fields)

```yaml
name: My Extension
repo: https://github.com/user/my-extension
type: plugin
scope: [global, project]
tagline: A brief tagline (≤120 chars)
description: |
  A longer description explaining
  the extension in detail.
tags:
  - productivity
  - ai
min_version: "1.0.0"
homepage: https://user.dev/docs
installation: |
  ## Installation
  Run `opencode install my-extension`
```

---

## 9. Implementation Plan: Schema Additions

**Created**: 2026-01-11
**Last Updated**: 2026-01-11
**Status**: planning

This section provides a phased implementation plan for adding the new schema fields (`type`, `scope`, `tags`, `min_version`, `homepage`, `installation`) to the awesome-opencode registry.

---

### Current State Overview

| Component | Status |
|-----------|--------|
| YAML entries (61 files) | Existing, uses basic schema |
| JSON schema (`data/schema.json`) | Out of sync with proposed fields |
| Generation script (`scripts/generate-readme.js`) | Needs update for new fields |
| Validation script (`scripts/validate.js`) | Needs update for new fields |
| Contributing guide (`contributing.md`) | Needs documentation update |

---

### Phase 1: Schema Update
**Goal**: Update `data/schema.json` with new fields

#### Phase 1 Deliverables
- [ ] JSON schema validates new `type` field (required enum)
- [ ] JSON schema validates new `scope` field (required array)
- [ ] JSON schema validates new optional fields (`tags`, `min_version`, `homepage`, `installation`)
- [ ] Validation script runs without errors

#### Phase 1 Tasks

- [ ] **Task 1.1**: Add `type` field as required enum
  - [ ] Open `data/schema.json` and read current structure
  - [ ] Add `type` property with enum values: `plugin`, `theme`, `agent`, `project`, `resource`, `fork`
  - [ ] Set `type` to required in schema
  - [ ] Add description for each enum value

- [ ] **Task 1.2**: Add `scope` field as required array
  - [ ] Add `scope` property with items enum: `global`, `project`
  - [ ] Set `minItems: 1` (at least one scope required)
  - [ ] Set `uniqueItems: true` (no duplicates)
  - [ ] Set `scope` to required in schema

- [ ] **Task 1.3**: Add `tags` field as optional string array
  - [ ] Add `tags` property with type `array` of `string`
  - [ ] Leave `tags` as optional (not in required array)

- [ ] **Task 1.4**: Add `min_version` field as optional string
  - [ ] Add `min_version` property with type `string`
  - [ ] Add semver regex pattern for validation
  - [ ] Leave `min_version` as optional

- [ ] **Task 1.5**: Add `homepage` field as optional URL
  - [ ] Add `homepage` property with type `string`
  - [ ] Add URI format pattern (http/https)
  - [ ] Leave `homepage` as optional

- [ ] **Task 1.6**: Add `installation` field as optional markdown
  - [ ] Add `installation` property with type `string`
  - [ ] Add description indicating markdown format
  - [ ] Leave `installation` as optional

- [ ] **Task 1.7**: Validate schema structure
  - [ ] Run `node scripts/validate.js` to check schema validity
  - [ ] Fix any JSON syntax errors
  - [ ] Verify schema against JSON Schema specification

#### Phase 1 Manual Testing
**Testing Requirements**: User performs hands-on testing in terminal

- [ ] Run validation script: `node scripts/validate.js`
- [ ] Verify schema loads without errors
- [ ] Check that new fields are recognized in validation output
- [ ] Test with a sample YAML file containing new fields

#### Phase 1 Review Process
**User Review Required**: Do NOT proceed to Phase 2 without user sign-off

**Review Checklist**:
- [ ] Schema validates without errors
- [ ] All new field types are correct (required vs optional)
- [ ] Enum values match documented extension types
- [ ] User approves proceeding to next phase

**Go/No-Go Decision**: [_______] (User fills in date and sign-off)
- [ ] **GO** - Proceed to Phase 2
- [ ] **NO-GO** - Fix issues first

**Issues to Fix**:
- [ ] Issue 1: [Description]
- [ ] Issue 2: [Description]

---

### Phase 2: Migrate Existing Entries
**Goal**: Add new required fields (`type`, `scope`) to all 61 YAML entries

#### Phase 2 Deliverables
- [ ] All 61 YAML entries have `type` field
- [ ] All 61 YAML entries have `scope` field
- [ ] Validation passes for all entries

#### Phase 2 Tasks

- [ ] **Task 2.1**: Map category folders to extension types
  - [ ] `data/plugins/` → `type: plugin`
  - [ ] `data/themes/` → `type: theme`
  - [ ] `data/agents/` → `type: agent`
  - [ ] `data/projects/` → `type: project`
  - [ ] `data/resources/` → `type: resource`
  - [ ] `data/forks/` → `type: fork`

- [ ] **Task 2.2**: Add `type` field to all entries
  - [ ] Use `grep` to find all YAML files in `data/` subdirectories
  - [ ] For each file in `data/plugins/`, add `type: plugin` after `description`
  - [ ] For each file in `data/themes/`, add `type: theme`
  - [ ] Continue for all category folders (use edit with replaceAll for efficiency)
  - [ ] Verify files were modified correctly

- [ ] **Task 2.3**: Add `scope` field to all entries
  - [ ] Add `scope: [global, project]` to all entries (most extensions work both places)
  - [ ] Consider special cases:
    - Themes: May be `[global]` only
    - Resources: May vary based on content type
  - [ ] Add appropriate scope based on extension type analysis

- [ ] **Task 2.4**: Run validation on all entries
  - [ ] Execute `node scripts/validate.js`
  - [ ] Fix any entries that fail validation
  - [ ] Re-run validation until all 61 entries pass

#### Phase 2 Manual Testing
**Testing Requirements**: User verifies migration was successful

- [ ] Check 3-5 random YAML files for correct `type` field
- [ ] Check 3-5 random YAML files for correct `scope` field
- [ ] Run `node scripts/validate.js` and verify all entries pass
- [ ] Confirm count: 61 entries should be valid

#### Phase 2 Review Process
**User Review Required**: Do NOT proceed to Phase 3 without user sign-off

**Review Checklist**:
- [ ] All YAML files have `type` field matching their directory
- [ ] All YAML files have `scope` field
- [ ] Validation script shows 61 valid entries
- [ ] No schema validation errors
- [ ] User approves proceeding to next phase

**Go/No-Go Decision**: [_______] (User fills in date and sign-off)
- [ ] **GO** - Proceed to Phase 3
- [ ] **NO-GO** - Fix issues first

**Issues to Fix**:
- [ ] Issue 1: [Description]
- [ ] Issue 2: [Description]

---

### Phase 3: Update Generation Script
**Goal**: Update `scripts/generate-readme.js` to use new fields

#### Phase 3 Deliverables
- [ ] Template includes new fields in output
- [ ] Backwards compatibility maintained (optional fields can be missing)
- [ ] README regenerates successfully

#### Phase 3 Tasks

- [ ] **Task 3.1**: Update README template
  - [ ] Open `templates/README.template.md`
  - [ ] Add badges or labels for `type` field (e.g., "[plugin]", "[theme]")
  - [ ] Add tags display section if `tags` field is present
  - [ ] Ensure template handles missing optional fields gracefully

- [ ] **Task 3.2**: Update generation script logic
  - [ ] Open `scripts/generate-readme.js`
  - [ ] Ensure script reads new fields from YAML entries
  - [ ] Add logic to skip optional fields if empty/missing
  - [ ] Add type badges to extension listing output

- [ ] **Task 3.3**: Regenerate README
  - [ ] Run `node scripts/generate-readme.js`
  - [ ] Check `README.md` for new field displays
  - [ ] Verify backwards compatibility (entries without optional fields render correctly)
  - [ ] Check that type badges appear correctly

#### Phase 3 Manual Testing
**Testing Requirements**: User verifies README output

- [ ] Open `README.md` and verify type badges are visible
- [ ] Check that entries with tags display them
- [ ] Verify entries without optional fields render without errors
- [ ] Review README for formatting issues

#### Phase 3 Review Process
**User Review Required**: Do NOT proceed to Phase 4 without user sign-off

**Review Checklist**:
- [ ] New fields appear in generated README
- [ ] Backwards compatibility works (no breaking changes)
- [ ] Formatting looks correct (badges, tags, etc.)
- [ ] User approves proceeding to next phase

**Go/No-Go Decision**: [_______] (User fills in date and sign-off)
- [ ] **GO** - Proceed to Phase 4
- [ ] **NO-GO** - Fix issues first

**Issues to Fix**:
- [ ] Issue 1: [Description]
- [ ] Issue 2: [Description]

---

### Phase 4: Update Contributing Guide
**Goal**: Document new fields in `contributing.md`

#### Phase 4 Deliverables
- [ ] Contributing guide includes complete YAML format reference
- [ ] All new fields documented with examples
- [ ] Type and scope options documented

#### Phase 4 Tasks

- [ ] **Task 4.1**: Update `contributing.md` with new required fields
  - [ ] Open `contributing.md`
  - [ ] Add `type` field to YAML format reference
  - [ ] Add `scope` field to YAML format reference
  - [ ] Mark these as **required** fields

- [ ] **Task 4.2**: Add installation template example
  - [ ] Add `installation` field documentation
  - [ ] Include markdown example with all sections
  - [ ] Mark `installation` as **optional**

- [ ] **Task 4.3**: Document type and scope options
  - [ ] Add section listing all valid `type` values with descriptions
  - [ ] Add section explaining `scope` values (`global`, `project`, `[global, project]`)
  - [ ] Add `tags`, `min_version`, `homepage` documentation

#### Phase 4 Manual Testing
**Testing Requirements**: User reviews documentation

- [ ] Open `contributing.md` and verify new sections are present
- [ ] Check that YAML examples are valid
- [ ] Verify field descriptions match schema
- [ ] Test that documentation is clear for new contributors

#### Phase 4 Review Process
**User Review Required**: Do NOT proceed to Phase 5 without user sign-off

**Review Checklist**:
- [ ] All new fields documented in contributing guide
- [ ] Examples are accurate and complete
- [ ] Required vs optional fields clearly marked
- [ ] User approves proceeding to next phase

**Go/No-Go Decision**: [_______] (User fills in date and sign-off)
- [ ] **GO** - Proceed to Phase 5
- [ ] **NO-GO** - Fix issues first

**Issues to Fix**:
- [ ] Issue 1: [Description]
- [ ] Issue 2: [Description]

---

### Phase 5: Export Script (Future)
**Goal**: Create script for downstream JSON export

#### Phase 5 Deliverables
- [ ] `scripts/export-json.js` script created
- [ ] Script outputs opencode.cafe compatible JSON
- [ ] Field mapping documented

#### Phase 5 Tasks

- [ ] **Task 5.1**: Create export script
  - [ ] Create `scripts/export-json.js`
  - [ ] Load all YAML entries from `data/` directories
  - [ ] Map fields according to downstream compatibility table (Section 7):
    - `name` → `displayName`
    - `repo` → `repoUrl`
    - `type` → `type` (direct map)
    - `tags` → `tags`
    - `homepage` → `homepageUrl`
    - `installation` → `installation`
    - filename → `productId`

- [ ] **Task 5.2**: Generate productId from filename
  - [ ] Extract filename without extension (e.g., `codegpt.yaml` → `codegpt`)
  - [ ] Use as `productId` in output JSON

- [ ] **Task 5.3**: Test export output
  - [ ] Run `node scripts/export-json.js`
  - [ ] Verify output is valid JSON array
  - [ ] Check that field mapping is correct
  - [ ] Validate against opencode.cafe schema (if available)

#### Phase 5 Manual Testing
**Testing Requirements**: User verifies export script

- [ ] Run `node scripts/export-json.js` > dist/output.json
- [ ] Verify output is valid JSON
- [ ] Check 2-3 entries for correct field mapping
- [ ] Verify `productId` is derived from filename

#### Phase 5 Review Process
**User Review Required**: Phase 5 completion is optional (future enhancement)

**Review Checklist**:
- [ ] Export script runs successfully
- [ ] Output JSON is valid
- [ ] Field mapping matches specification
- [ ] User approves Phase 5 completion

**Go/No-Go Decision**: [_______] (User fills in date and sign-off)
- [ ] **GO** - Phase 5 complete
- [ ] **NO-GO** - Fix issues first

**Issues to Fix**:
- [ ] Issue 1: [Description]
- [ ] Issue 2: [Description]

---

### Important Notes

#### What NOT To Do
- **DO NOT run `npm install`** - The project may have existing dependencies; only run if explicitly instructed
- **DO NOT push changes to remote** - Always ask user before committing or pushing
- **DO NOT modify unrelated files** - Stay within the scope of this implementation plan
- **DO NOT delete existing data** - Back up or migrate data, never remove without confirmation
- **DO NOT change existing field behavior** - Only add new fields, don't modify existing ones

#### Commands to Use
```bash
# Validate schema and entries
node scripts/validate.js

# Regenerate README
node scripts/generate-readme.js

# Export JSON (Phase 5)
node scripts/export-json.js
```

#### File Locations
- Schema: `data/schema.json`
- YAML entries: `data/{category}/*.yaml`
- Generation script: `scripts/generate-readme.js`
- Validation script: `scripts/validate.js`
- Template: `templates/README.template.md`
- Contributing guide: `contributing.md`
- Export script: `scripts/export-json.js` (to be created)

---

## Success Criteria

### Schema Migration Complete When
- [ ] `data/schema.json` includes all new fields with correct types
- [ ] All 61 YAML entries have `type` and `scope` fields
- [ ] `node scripts/validate.js` passes without errors
- [ ] `node scripts/generate-readme.js` produces valid README with new fields
- [ ] `contributing.md` documents all new fields

### Quality Criteria
- [ ] Backwards compatibility maintained (optional fields can be missing)
- [ ] No breaking changes to existing data
- [ ] TypeScript compilation passes (if applicable)
- [ ] Linting passes (if applicable)
- [ ] All tests passing (if tests exist)

---

## Progress Log

**Started**: [Date]
**Last Updated**: 2026-01-11

### Completed Tasks
- [ ] [Date] Task completed - [Brief description]
- [ ] [Date] Another task completed - [Brief description]

### Issues Encountered
- [Date] Issue description and how it was resolved
- [Date] Another issue and resolution

### Phase Status
- Phase 1: [pending | in_progress | testing | review | completed]
- Phase 2: [pending | in_progress | testing | review | completed]
- Phase 3: [pending | in_progress | testing | review | completed]
- Phase 4: [pending | in_progress | testing | review | completed]
- Phase 5: [pending | in_progress | testing | review | completed] (future)

---

**End of Implementation Plan**

---

*Document Version: 1.1*  
*Last Updated: January 2026*
