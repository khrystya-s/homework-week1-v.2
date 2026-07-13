---
name: pull-request-reviewer

description: >
  Automatically review a GitHub Pull Request using the GitHub REST API and a
  Large Language Model. Before reviewing the Pull Request, obtain repository
  context using the github-repository-reader skill. Analyze changed files,
  detect potential issues, and publish review comments directly to GitHub.
---

# Pull Request Reviewer

## Purpose

Review a GitHub Pull Request.

Retrieve repository context using the `github-repository-reader` skill.

Retrieve the Pull Request diff using the GitHub REST API.

Analyze the modified code using a Large Language Model.

Publish meaningful review comments directly to the Pull Request.

---

# Dependency

# Dependency

Before reviewing a Pull Request, check whether repository context is already available.

If repository context is not available, invoke the
`github-repository-reader` skill.

Wait until the repository context has been successfully retrieved.

The repository context should include:

- repository metadata;
- repository description;
- default branch;
- recursive directory tree;
- complete file list;
- important repository files when available.

Use this context to better understand the project structure before reviewing the Pull Request.

Do not retrieve repository information independently when the
`github-repository-reader` skill has already provided it.

The repository context provides architectural information that improves the quality of the review.

Never begin reviewing a Pull Request before the repository context is available.

---

# When to use

Use this skill whenever the user asks to:

- review a Pull Request;
- analyze Pull Request changes;
- detect bugs in a Pull Request;
- perform code review;
- automatically review GitHub Pull Requests;
- publish review comments.

---

# Input

Accept one of the following formats:

```
https://github.com/{owner}/{repository}/pull/{number}
```

or

```
owner/repository
Pull Request Number
```

Normalize the repository identifier to:

```
owner/repository
```

---

# Authentication

Authenticate using a GitHub Personal Access Token.

Look for credentials in the following order:

1. GITHUB_TOKEN
2. GH_TOKEN

The token must allow:

- reading Pull Requests;
- reading repository contents;
- creating Pull Request reviews;
- publishing review comments.

Never expose authentication credentials.

---

# Responsibilities

Retrieve:

- repository context;
- Pull Request metadata;
- changed files;
- Pull Request diff.

Analyze every changed file independently.

Generate review comments only when meaningful issues are detected.

Publish review comments through the GitHub REST API.

---

# Workflow

## Step 1

Check whether repository context is already available.

If repository context is unavailable:

- invoke the `github-repository-reader` skill;
- wait until it finishes successfully;
- use the retrieved repository context for the review.

If repository context is already available, reuse it.

Do not retrieve repository information twice.

---

## Step 2

Retrieve Pull Request metadata.

Endpoint:

```
GET /repos/{owner}/{repository}/pulls/{pull_number}
```

Extract:

- repository;
- base branch;
- head branch;
- latest commit SHA.

---

## Step 3

Retrieve Pull Request files.

Endpoint:

```
GET /repos/{owner}/{repository}/pulls/{pull_number}/files
```

For every changed file retrieve:

- filename;
- status;
- additions;
- deletions;
- patch (diff).

---

## Step 4

Review each changed file independently.

For every changed file:

1. Read the complete diff.
2. Analyze only the modified code.
3. Generate review comments only when meaningful issues are found.
4. Continue with the next file.

After all files have been analyzed, generate the final review summary.

Ignore unchanged code.

---

# Review Categories

Evaluate every changed file for:

## Potential Bugs

Look for:

- incorrect logic;
- missing validation;
- null reference issues;
- race conditions;
- incorrect API usage.

---

## Code Style

Check:

- readability;
- naming consistency;
- formatting;
- unnecessary complexity.

---

## Architecture

Evaluate:

- modularity;
- separation of concerns;
- duplicated logic;
- maintainability.

---

## Performance

Identify:

- inefficient algorithms;
- redundant computations;
- unnecessary allocations;
- expensive operations.

---

## Security

Look for:

- exposed secrets;
- insecure API usage;
- missing input validation;
- injection risks;
- unsafe authentication.

---

## Edge Cases

Identify:

- missing boundary checks;
- missing exception handling;
- unsupported inputs;
- unexpected runtime scenarios.

---

## Recommendations

Provide practical recommendations.

Every recommendation should explain:

- what should change;
- why it should change;
- expected benefit.

---

# Severity Levels

Classify every issue as:

- HIGH
- MEDIUM
- LOW

Use:

HIGH

- bugs
- crashes
- incorrect algorithms
- security vulnerabilities
- data corruption

MEDIUM

- architectural issues
- duplicated logic
- maintainability
- performance problems

LOW

- style
- readability
- formatting
- naming consistency

---

# Review Rules

Generate review comments only when a meaningful issue is found.

Never publish comments merely to increase the number of comments.

Do not praise correct code.

Avoid duplicate comments.

If multiple issues describe the same problem, publish only one review comment.

Ignore generated files such as:

- package-lock.json
- yarn.lock
- pnpm-lock.yaml
- dist/*
- build/*
- vendor/*
- *.min.js

Ignore documentation files such as:

- README.md
- LICENSE
- CHANGELOG.md

unless the user explicitly requests documentation review.

If no meaningful issues are found:

Do not create inline review comments.

Instead publish a single Pull Request review stating that no significant issues were detected.

---

# Publishing Review

Publish review comments using the GitHub Pull Request Review API.

Endpoint:

```
POST /repos/{owner}/{repository}/pulls/{pull_number}/reviews
```

Create inline review comments whenever the changed line is available.

Do not use Issue Comments unless explicitly requested.

---

# Output

## Console Output

Print:

- repository name;
- Pull Request number;
- reviewed files;
- detected issues;
- HIGH issues;
- MEDIUM issues;
- LOW issues;
- overall summary.

---

## Markdown Report

If the directory

```
output/
```

does not exist, create it.

Save

```
output/review.md
```

Example structure:

```md
# Pull Request Review

## Summary

...

## Reviewed Files

...

## HIGH Issues

...

## MEDIUM Issues

...

## LOW Issues

...

## Recommendations

...
```

---

## JSON Report

Save

```
output/review.json
```

Format:

```json
{
  "repository": "",
  "pull_request": 0,
  "reviewed_files": 0,
  "summary": "",
  "issues": [
    {
      "severity": "HIGH",
      "category": "Potential Bug",
      "file": "",
      "line": 0,
      "comment": "",
      "recommendation": ""
    }
  ]
}
```

---

# Constraints

Always use repository context provided by the
`github-repository-reader` skill whenever available.

Only invoke the `github-repository-reader` skill when repository context has not already been retrieved.

Always retrieve Pull Request information using the GitHub REST API.

Always review only modified code.

Never review unchanged files.

Never clone repositories.

Never execute Git commands.

Never fabricate issues.

Only publish comments supported by the analyzed code.

Always:

- print a review summary to the terminal;
- generate `output/review.md`;
- generate `output/review.json`;
- publish review comments using the GitHub Pull Request Review API.