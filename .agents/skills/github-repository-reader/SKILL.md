---
name: github-repository-reader

description: >
  Use this skill whenever the user asks to inspect a GitHub repository,
  list repository files, retrieve the repository structure, or read
  repository contents from GitHub. Always use the GitHub REST API.
---

# GitHub Repository Reader

## Purpose

Retrieve information from a remote GitHub repository.

This skill is responsible only for reading repository data.

Repository analysis is outside the scope of this skill.

---

# Mandatory Rules

This skill MUST always use the GitHub REST API.

This skill MUST NOT:

- clone repositories;
- execute git commands;
- execute Python scripts;
- execute Bash scripts;
- execute PowerShell scripts;
- execute Node.js scripts;
- create helper scripts;
- create temporary files;
- use command-line tools to parse JSON;
- use jq, Python, Node.js or any external parser;
- inspect the local filesystem.

Interpret API responses directly.

The JSON returned by GitHub must be read and interpreted without generating code.

---

# When to use

Use this skill whenever the user asks to:

- inspect a GitHub repository;
- list files;
- recursively list files;
- retrieve the repository tree;
- retrieve repository metadata;
- read repository contents.

Always prefer this skill over cloning repositories.

---

# Accepted Input

Accept:

```
owner/repository
```

or

```
https://github.com/owner/repository
```

or

```
https://github.com/owner/repository.git
```

Normalize every input to:

```
owner/repository
```

---

# Authentication

Look for credentials in the following order:

1. GITHUB_TOKEN
2. GH_TOKEN

If no token exists and the repository is public,
continue using unauthenticated GitHub API requests.

Never expose authentication credentials.

---

# Workflow

## Step 1

Retrieve repository metadata.

Endpoint:

```
GET /repos/{owner}/{repository}
```

Read directly from the API response:

- repository name
- owner
- description
- default branch

Do not execute any scripts.

---

## Step 2

Retrieve the complete repository tree.

Endpoint:

```
GET /repos/{owner}/{repository}/git/trees/{default_branch}?recursive=1
```

Read the JSON response directly.

Extract:

- directories
- files

Do not execute code to parse the response.

---

## Step 3

If explicitly requested, retrieve file contents.

Endpoint:

```
GET /repos/{owner}/{repository}/contents/{path}
```

Decode Base64 only using built-in capabilities.

Do not generate helper scripts.

---

# Console Output

The primary result of this skill is a recursive list of repository files.

Print every discovered file path directly to the terminal.

Print one file per line.

Example:

README.md
LICENSE
pyproject.toml
docs/index.md
src/main.py
src/utils.py
tests/test_main.py

The default behavior is to display the recursive file list in the terminal.

# Output

If the directory output/ does not exist, create it.

Save the Markdown file as output/file-list.md

Return structured information.

Example:

```json
{
  "repository": "",
  "owner": "",
  "description": "",
  "default_branch": "",
  "directories": [],
  "files": []
}
```

If file contents are requested:

```json
{
  "path": "",
  "content": ""
}
```

---

# Error Handling

Handle:

- repository not found;
- authentication failed;
- insufficient permissions;
- GitHub API unavailable;
- GitHub API rate limit exceeded.

Return GitHub error messages unchanged.

Never fabricate repository information.

---

# Final Constraints

This skill performs only repository retrieval.

Never:

- analyze code;
- evaluate architecture;
- generate recommendations;
- execute scripts;
- create temporary programs;
- clone repositories;
- execute git;
- execute Python;
- execute PowerShell;
- execute Bash;
- execute Node.js.

Always retrieve and interpret GitHub REST API responses directly.

Always print the recursive repository file list to the terminal.

Additionally:

- create the directory output if it does not exist;
- save the same file list into output/file-list.md.

Do not replace console output with the Markdown file.

The terminal output is mandatory.