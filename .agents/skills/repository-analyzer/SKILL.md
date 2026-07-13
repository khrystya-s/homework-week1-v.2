---
name: repository-analyzer

description: >
  Analyze a software repository using the repository context provided by the
  github-repository-reader skill. Generate a structured analysis, print a
  summary to the terminal, and save the analysis as both Markdown and JSON.
---

# Repository Analyzer

## Purpose

Analyze a software repository using the repository context provided by the
`github-repository-reader` skill.

This skill is responsible only for repository analysis.

It never retrieves repository data directly.

---

# Dependency

Before starting any analysis, always invoke the
`github-repository-reader` skill.

Wait until the repository context has been successfully retrieved.

The repository context must include:

- repository metadata;
- repository description;
- default branch;
- recursive directory tree;
- complete file list;
- contents of important files when available.

Never begin analysis before the repository context is available.

The `github-repository-reader` skill is the only source of repository information.

---

# Mandatory Rules

This skill MUST NOT:

- access GitHub directly;
- clone repositories;
- execute Git commands;
- retrieve repository contents independently;
- inspect repositories on its own.

Always analyze only the repository context received from the
`github-repository-reader` skill.

Never fabricate repository information.

---

# When to use

Use this skill whenever the user asks to:

- analyze a repository;
- summarize a software project;
- describe repository architecture;
- identify technologies;
- evaluate repository quality;
- generate repository documentation.

---

# Workflow

## Step 1

Invoke the `github-repository-reader` skill.

Wait until the repository context has been successfully retrieved.

Do not skip this step.

---

## Step 2

Receive the repository context.

The context should include:

- repository metadata;
- repository description;
- recursive directory tree;
- recursive file list;
- contents of important files.

---

## Step 3

Analyze the repository.

Determine:

- project summary;
- technologies;
- project structure;
- strengths;
- potential issues;
- recommendations.

Use only the provided repository context.

Never assume information that is not present.

---

## Step 4

Print a concise repository analysis directly to the terminal.

Include:

- repository name;
- project summary;
- technologies;
- strengths;
- potential issues;
- recommendations.

Console output is the primary result.

---

## Step 5

If the directory

```
output/
```

does not exist, create it.

Generate:

```
output/report.md
```

and

```
output/report.json
```

---

# Analysis Tasks

## Project Summary

Write a concise technical description of the repository.

---

## Technologies

Identify technologies explicitly supported by the available repository context.

Examples:

- programming languages;
- frameworks;
- libraries;
- package managers;
- build tools.

Never assume technologies that are not present.

---

## Project Structure

Describe:

- overall repository organization;
- important directories;
- main modules;
- project layout.

---

## Strengths

Identify positive aspects supported by the repository context.

Examples:

- clear documentation;
- modular architecture;
- readable code;
- logical project structure;
- consistent naming;
- maintainable organization.

---

## Potential Issues

Report only issues supported by the available repository context.

Examples:

- missing documentation;
- inconsistent naming;
- missing tests;
- weak dependency management;
- oversized source files;
- poor project organization.

Never invent problems.

---

## Recommendations

Provide practical recommendations based on the repository analysis.

Examples:

- improve documentation;
- increase test coverage;
- reorganize modules;
- improve naming consistency;
- simplify project structure;
- improve maintainability.

Clearly distinguish recommendations from observations.

---

# Output

## Console Output

Print a concise repository analysis to the terminal.

Include:

- repository name;
- project summary;
- technologies;
- strengths;
- potential issues;
- recommendations.

---

## Markdown Report

Save the complete analysis as

```
output/report.md
```

using the following structure:

```md
# Repository Analysis

## Project Summary

...

## Technologies

...

## Project Structure

...

## Strengths

...

## Potential Issues

...

## Recommendations

...
```

---

## JSON Report

Save the same analysis as

```
output/report.json
```

using the following format:

```json
{
  "repository": "",
  "summary": "",
  "technologies": [],
  "project_structure": "",
  "strengths": [],
  "potential_issues": [],
  "recommendations": []
}
```

The JSON report must contain only information supported by the available repository context.

---

# Constraints

This skill analyzes only the repository context produced by the
`github-repository-reader` skill.

Never:

- access GitHub directly;
- clone repositories;
- execute Git commands;
- retrieve repository data independently;
- fabricate repository information;
- assume technologies that are not supported by the available repository context.

Always distinguish observations from recommendations.

Always use the `github-repository-reader` skill as the only source of repository information.

Always:

- print the analysis to the terminal;
- generate `output/report.md`;
- generate `output/report.json`.