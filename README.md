# CC-Skills

Claude Code Skills - A collection of specialized skills for Claude Code to enhance development workflows.

## 📌 Project Overview

CC-Skills is a repository containing specialized skills for Claude Code that automate and improve common development tasks. Currently includes two main skills: a Git Commit Assistant for streamlined git workflows and a README Generator for creating comprehensive project documentation.

## 🏗️ Overall Architecture

```
CC-Skills/
├── git-commit-assistant/
│   └── SKILL.md          # Git commit workflow automation
├── readme-generation/
│   └── SKILL.md          # README generation from templates
└── README.md             # Project documentation
```

### Components:
- **git-commit-assistant**: Automates the git commit process with safety checks, conventional commit formatting, and code review
- **readme-generation**: Generates project README files following standardized templates and best practices

## 🔄 Communication and Runtime Model

The skills in this repository operate as specialized tools within the Claude Code environment. Each skill defines specific triggers and workflows that Claude can execute when prompted by the user. The skills work independently but follow consistent patterns for user interaction and safety checks.

## 🧰 Environment and Dependencies

### Prerequisites
- Claude Code (latest version recommended)
- Git (for git-commit-assistant functionality)
- Access to Claude's skill execution environment

## 🛠️ Build

This repository doesn't require compilation or building. The skills are defined in markdown files that Claude Code can directly interpret and execute.

## 🚀 Deployment

1. Clone or download the repository
2. Place the skill files in your Claude Code skills directory
3. Restart Claude Code to load the new skills
4. The skills will be available for use based on their trigger conditions

## 🎮 Usage Guide

### Git Commit Assistant
**Overview**: Provides a safe, streamlined git commit workflow with code review and conventional commit formatting.

**Basic Syntax**:
```
git commit [options]
```

**When to Use**:
- When you want to commit changes with safety checks
- When you need conventional commit format generation
- When you want automated code review before committing

**Module Usage**:
- Triggers when user explicitly requests "commit", "git commit", "提交代码", etc.
- Performs safety checks for sensitive information
- Formats commit messages following repository conventions
- Requires final confirmation before executing commit

**Help**: The skill will activate when you ask Claude to commit changes or when you explicitly request git commit assistance.

**Typical Usage Scenario**:
1. Make changes to your code
2. Ask Claude "Please commit these changes"
3. The assistant will review changes, suggest commit message, and ask for confirmation
4. Upon confirmation, executes the git commit command

### README Generation Skill
**Overview**: Generates project README files based on templates and project structure analysis.

**Basic Syntax**:
```
Generate README for this project
```

**When to Use**:
- When starting a new project that needs documentation
- When updating existing documentation
- When following standardized README templates

**Module Usage**:
- Triggers when user requests README generation
- Analyzes project structure
- Generates README following standardized format
- Includes architecture, usage, and deployment information

**Typical Usage Scenario**:
1. Ask Claude "Generate a README for this project"
2. The skill analyzes the project structure
3. Creates a comprehensive README following best practices
4. Outputs the README content for review and confirmation

## 📝 License

This project is open source and available under the MIT License.
