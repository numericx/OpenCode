# OpenCode

OpenCode is an AI-powered coding assistant that helps developers with software engineering tasks through an interactive command-line interface.

## Overview

OpenCode provides intelligent assistance for software development tasks including:
- Bug fixing and code debugging
- Feature implementation and refactoring
- Code explanation and documentation
- Testing and quality assurance
- Project setup and configuration guidance

## Key Features

- **Intelligent Code Assistance**: Leverages advanced AI models to understand and generate code
- **Interactive CLI**: Provides a conversational interface for coding tasks
- **Multi-tool Support**: Integrates with bash, file system operations, and development workflows
- **Context-Aware**: Understands codebase structure and conventions to provide relevant suggestions
- **Security Focused**: Includes permission controls to protect sensitive files and operations

## Configuration

The repository includes configuration files that define:
- Permission controls for file access, editing, and bash commands (`opencode.json`)
- Behavioral guidelines for AI coding assistance (`AGENTS.md`)
- Python development best practices reference (`python-research.md`)
- Specialized subagents for Python development (`.opencode/agents/`)
- Frontend design skills for creating distinctive user interfaces

## Getting Started

1. Install dependencies:
   ```bash
   npm install @opencode-ai/plugin
   ```

2. Configure permissions in `opencode.json` according to your security requirements

3. Run OpenCode to start interacting with the AI coding assistant

## Python Development Agents

OpenCode ships with specialized Python subagents that encapsulate modern Python tooling (uv, ruff, mypy, pytest, mkdocs):

| Agent | Invoke with | Role |
|-------|-------------|------|
| **@ask** | `@ask how does this function work?` | Answer questions and explain code — strictly read-only, no changes ever |
| **@review** | `@review src/myproject` | Full-stack code review: lint, types, tests, security, architecture, and design — read-only |
| **@fix** | `@fix` (paste code or copy to clipboard) | Debug logic, syntax, and runtime errors — reads from clipboard or inline context, shows fixed code — read-only |
| **@scaffold** | `@scaffold set up a new project` | Initialize projects with uv, manage dependencies, create `src/` layout, configure `pyproject.toml` |
| **@quality** | `@quality fix linting issues` | Configure and run ruff (lint + format), mypy type checking, pre-commit hooks |
| **@test** | `@test write tests for the API` | Set up pytest, write tests, manage coverage, run deptry dependency checks |
| **@docs** | `@docs set up mkdocs` | Configure MkDocs Material or Sphinx, enforce docstring conventions, maintain README |
| **@cicd** | `@cicd add a CI pipeline` | Create GitHub Actions workflows, Dockerfiles, PyPI publish with trusted publishers |
| **@security** | `@security audit the codebase` | Run ruff S-rules, pip-audit, and bandit (read-only — reports findings, never edits) |

Each agent embeds the `python-research.md` guidelines and `AGENTS.md` behavioral rules directly in its prompt, and has per-role restricted permissions (e.g., @security is read-only, @docs only edits documentation files).

## Security

OpenCode includes granular permission controls to ensure safe operation:
- File read/write permissions with specific allow/deny patterns
- Controlled bash command execution with predefined rules
- External directory access restrictions
- Web access controls

These controls help prevent unauthorized access to sensitive files and system resources.

## Contributing

Contributions are welcome! Please read our guidelines in `AGENTS.md` which outline our approach to:
- Thoughtful coding practices
- Simplicity in implementation
- Surgical changes to codebases
- Goal-driven execution

## License

This project is licensed under the terms specified in the LICENSE file.