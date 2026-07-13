# Homework Week 1 — GitHub Skills for Antigravity CLI

## Overview

This project implements a collection of custom Antigravity Skills for working with GitHub repositories.

The skills use the GitHub REST API to inspect repositories, analyze their contents, and automatically review Pull Requests using a Large Language Model.

## Implemented Skills

### 1. GitHub Repository Reader

Retrieves repository information directly from GitHub.

Features:

- retrieves repository metadata;
- detects the default branch;
- recursively lists directories and files;
- retrieves selected file contents when requested;
- prints repository structure to the console;
- saves the repository structure as a Markdown report in `output/`.

The skill never clones repositories or executes Git commands.

---

### 2. Repository Analyzer

Analyzes the structure and contents of a GitHub repository.

Before starting the analysis, it invokes the `github-repository-reader` skill to obtain repository context.

The analyzer evaluates:

- project structure;
- architecture;
- technologies;
- dependencies;
- code organization;
- potential improvements.

Reports are generated in both Markdown and JSON formats.

---

### 3. Pull Request Reviewer (Bonus)

Automatically reviews GitHub Pull Requests.

The reviewer:

- retrieves Pull Request information using the GitHub REST API;
- obtains repository context from the `github-repository-reader` skill;
- analyzes changed files using an LLM;
- detects:
  - potential bugs;
  - code style issues;
  - architectural problems;
  - performance concerns;
  - security issues;
  - missing edge cases;
- publishes review comments directly to GitHub using the Pull Request Review API;
- generates Markdown and JSON review reports.

## Project Structure

```text
.agents/
└── skills/
    ├── github-repository-reader/
    │   └── SKILL.md
    ├── repository-analyzer/
    │   └── SKILL.md
    └── pull-request-reviewer/
        └── SKILL.md

output/
```

## Technologies

- Antigravity CLI
- GitHub REST API
- Large Language Model
- Markdown
- JSON

## Authentication

The skills use a GitHub Personal Access Token when available.

Supported environment variables:

- `GITHUB_TOKEN`
- `GH_TOKEN`

If no token is provided, public repositories are accessed without authentication whenever GitHub allows it.

## Design

The skills are designed with separation of responsibilities:

- **GitHub Repository Reader** is responsible only for retrieving repository information.
- **Repository Analyzer** performs repository analysis using the Reader as its data source.
- **Pull Request Reviewer** reviews Pull Requests and also reuses the Reader to obtain repository context before generating review comments.

This modular design avoids duplicated logic and promotes skill reusability.

## Output

Depending on the executed skill, generated reports are stored in the `output/` directory.

Possible outputs include:

- `repository.md`
- `repository.json`
- `analysis.md`
- `analysis.json`
- `review.md`
- `review.json`
