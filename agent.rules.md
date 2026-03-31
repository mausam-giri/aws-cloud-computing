# Agent Rules

This file defines the guiding principles and standards that the agent must adhere to when contributing to this repository.

---

## 1. Structure and Organization

- All content must be placed in its appropriate, numbered folder (e.g., `01-getting-started/`, `08-advanced-networking/`).
- Each folder must contain a `README.md` as its primary entry point.
- New topic areas must follow the existing `##-topic-name/` naming convention.
- Loose files must not be left at the repository root; every `.md` guide belongs inside a folder.
- The root `README.md` must always reflect the current directory structure and link to every top-level section.

---

## 2. Code Quality

- All AWS CLI commands included in guides must be syntactically correct and use realistic placeholder values (e.g., `vpc-xxxxxxxxxxxxxxxxx`).
- Commands that span multiple lines must use the `\` line-continuation character for readability.
- Code blocks must specify the appropriate language identifier (e.g., ` ```bash `, ` ```json `).
- Never include real account IDs, credentials, or sensitive values — use placeholders only.

---

## 3. Documentation Standards

- Every guide must include a **Table of Contents** with anchor links when it has three or more sections.
- Section headings must follow a consistent hierarchy: `#` for the page title, `##` for major sections, `###` for sub-sections.
- Architecture diagrams using ASCII art are encouraged for networking and infrastructure guides.
- Every guide must end with a **Related Topics** section linking to relevant sibling guides.
- Navigation links (`← Back to Main Index`, `Next →`) must be included at the top of each module `README.md`.

---

## 4. Clarity and Consistency

- Use consistent terminology throughout: prefer "Transit Gateway" over "TGW" in prose (abbreviations may appear in tables and code).
- Tables must align columns and use `|---|---|` separators.
- Warning callouts must use the `> ⚠️` prefix to highlight easy-to-miss steps.
- CLI equivalent blocks must be clearly marked with a `> **CLI equivalent:**` heading before the code block.
- Avoid vague language; every step must be actionable and specific.

---

## 5. Accuracy

- All instructions must reflect the current AWS Management Console UI and CLI behavior.
- Pricing information must include a link to the official AWS pricing page and note that rates may change.
- Cleanup sections must be present in every guide that provisions billable AWS resources, and resources must be listed in the correct deletion order.

---

## 6. Maintainability

- Cross-references between files must use relative paths (e.g., `./transit-gateway.md`, `../README.md`).
- When a file is moved or renamed, all references to it across the repository must be updated in the same commit.
- Do not duplicate content between guides; link to the authoritative guide instead.

---

## 7. Commit Hygiene

- Commit messages must be concise and describe what changed (e.g., `Add VPC Endpoint guide`, `Move networking files to 08-advanced-networking/`).
- Each commit must represent a single logical change.
- Build artifacts, dependency directories, and temporary files must never be committed; add them to `.gitignore`.
