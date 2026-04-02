# Proposal: Migrating Documentation from Confluence to GitHub

**Document Type:** Architecture Decision Proposal  
**Date:** April 2, 2026  
**Status:** Proposed  
**Audience:** Architecture Board

![Docs as Code](https://img.shields.io/badge/Docs--as--Code-GitHub%20Markdown-blue?logo=github)
![Status](https://img.shields.io/badge/Status-Proposed-yellow)
![Version](https://img.shields.io/badge/Version-1.0.0-green)

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Problem Statement](#problem-statement)
- [Current State Architecture](#current-state-architecture)
- [Proposed Solution](#proposed-solution)
- [Target State Architecture](#target-state-architecture)
- [Key Benefits](#key-benefits)
- [Risk Analysis](#risk-analysis)
- [Migration Approach](#migration-approach)
  - [Migration Timeline](#migration-timeline)
  - [Migration Workflow](#migration-workflow)
- [Pull Request Workflow for Docs](#pull-request-workflow-for-docs)
- [Repository Structure](#repository-structure)
- [Tooling Recommendations](#tooling-recommendations)
- [Success Metrics](#success-metrics)
- [Viewing This Document on GitHub](#viewing-this-document-on-github)
- [Conclusion](#conclusion)
- [Appendix: Comparison Table](#appendix-comparison-table)

---

## Executive Summary

This proposal recommends migrating our team documentation from Atlassian Confluence to GitHub. The move aligns documentation directly with source code, reduces tooling fragmentation, lowers costs, and enables the same engineering workflows (pull requests, code review, versioning) for documentation as we use for code.

---

## Current State Architecture

The diagram below shows the current fragmented documentation landscape, where documentation is disconnected from the codebase.

```mermaid
graph TD
    DEV[👨‍💻 Developer] -->|writes code| GH[GitHub Repository]
    DEV -->|writes docs separately| CF[Confluence]
    TL[Tech Lead] -->|reviews code| GH
    TL -->|reviews docs separately| CF
    ARC[Architect] -->|reads specs| CF
    NEW[New Joiner] -->|reads onboarding| CF
    NEW -->|reads code| GH

    GH -.->|docs drift out of sync| CF

    subgraph "Current State — Two Silos"
        GH
        CF
    end

    style CF fill:#FF5630,color:#fff
    style GH fill:#24292e,color:#fff
    style DEV fill:#0052CC,color:#fff
    style TL fill:#0052CC,color:#fff
    style ARC fill:#0052CC,color:#fff
    style NEW fill:#0052CC,color:#fff
```

**Pain points highlighted:** Developers maintain two separate systems. Documentation drifts away from code. No shared review process. Duplicated search effort.

---

## Problem Statement

Our current Confluence-based documentation suffers from several recurring issues:

- **Documentation drift:** Docs live separately from code, so they fall out of date when code changes.
- **No versioning alignment:** There is no native way to tie a doc page to a specific release or branch.
- **Review fatigue:** Confluence has no pull request model, so documentation changes receive little or no peer review.
- **Search fragmentation:** Engineers must search two separate tools — Confluence for docs, GitHub for code.
- **License cost:** Confluence Cloud pricing scales with user seats, adding recurring cost that GitHub (already licensed) does not.

---

## Target State Architecture

The diagram below shows the unified documentation architecture after migration to GitHub.

```mermaid
graph TD
    DEV[👨‍💻 Developer] -->|code + docs in same PR| GH[GitHub Repository]
    GH -->|GitHub Actions triggers| CI[CI/CD Pipeline]
    CI -->|lint, spell-check, link-check| QG{Quality Gate}
    QG -->|pass| PAGES[GitHub Pages / Static Site]
    QG -->|fail| DEV
    TL[Tech Lead] -->|single PR review| GH
    ARC[Architect] -->|reads ADRs in repo| GH
    NEW[New Joiner] -->|clones repo, reads docs offline| GH
    PAGES -->|published docs site| ALL[All Stakeholders]

    subgraph "Target State — Unified Docs-as-Code"
        GH
        CI
        QG
        PAGES
    end

    style GH fill:#24292e,color:#fff
    style CI fill:#2088FF,color:#fff
    style PAGES fill:#28a745,color:#fff
    style QG fill:#f6c90e,color:#000
    style DEV fill:#0052CC,color:#fff
    style TL fill:#0052CC,color:#fff
    style ARC fill:#0052CC,color:#fff
    style NEW fill:#0052CC,color:#fff
    style ALL fill:#6f42c1,color:#fff
```

---

## Proposed Solution

Move all documentation into Markdown files stored within GitHub repositories, following the **docs-as-code** methodology:

- Each repository owns its own documentation under a `/docs` folder or project wiki.
- A shared `docs` repository holds cross-cutting architecture decision records (ADRs), runbooks, and onboarding guides.
- GitHub Pages (or a static site generator such as MkDocs or Docusaurus) publishes content as a browsable site.

---

## Key Benefits

The mind map below summarises all key benefits at a glance:

```mermaid
mindmap
  root((GitHub Docs))
    Engineering
      Docs next to code
      Branching per release
      PR review workflow
      ADR traceability
    Cost
      No per-seat licence
      Existing GitHub licence
      Reduced tooling sprawl
    Quality
      CI linting & spell-check
      Broken link detection
      Mandatory peer review
    Access
      Offline via git clone
      Unified search
      Single onboarding tool
    Security
      RBAC via GitHub teams
      Audit trail in git log
      Signed commits
    Innovation
      Docs-as-Code standard
      Inner-source contribution
      Open-source alignment
```

### 1. Docs Live Next to Code

![Code and Docs Together](https://img.shields.io/badge/Code%20%2B%20Docs-Same%20PR-24292e?logo=github&logoColor=white)

Documentation lives in the same repository as the code it describes. When a developer changes an API, they update the docs in the same pull request. Reviewers see both changes together, making it nearly impossible for docs to silently fall out of sync.

### 2. Full Version History and Branching

![Git Version Control](https://img.shields.io/badge/Version%20Control-Git%20History-f05032?logo=git&logoColor=white)

Git gives every document a complete audit trail — who changed what, when, and why. Docs can be branched and merged alongside feature branches, enabling per-release documentation without any manual duplication.

### 3. Pull Request Workflow for Docs

![Pull Requests](https://img.shields.io/badge/Workflow-Pull%20Request%20Review-2088FF?logo=github-actions&logoColor=white)

All documentation changes go through the same pull request and code review process the team already uses. This enforces quality gates, enables inline comments, and creates an approval trail that satisfies governance requirements.


### 4. Architecture Decision Records (ADRs)

![ADR](https://img.shields.io/badge/ADR-Architecture%20Decision%20Records-6f42c1?logo=github&logoColor=white)

GitHub is the natural home for ADRs stored as Markdown files. ADRs can be linked directly to the commits or pull requests that implemented the decision, providing permanent traceability.

```mermaid
classDiagram
    class ADR {
        +int number
        +String title
        +String status
        +String date
        +String context
        +String decision
        +String consequences
        +link() GitCommit
    }

    class ADRStatus {
        <<enumeration>>
        PROPOSED
        ACCEPTED
        DEPRECATED
        SUPERSEDED
    }

    class GitCommit {
        +String sha
        +String message
        +String author
        +String timestamp
    }

    class PullRequest {
        +int number
        +String title
        +String[] reviewers
        +String mergedBy
    }

    ADR --> ADRStatus : has status
    ADR --> GitCommit : linked to
    ADR --> PullRequest : reviewed via
    GitCommit --> PullRequest : merged through
```

### 5. Unified Search and Discoverability

Engineers already search GitHub daily. Consolidating documentation there eliminates context switching and makes relevant docs surface naturally in code searches.

### 6. Cost Reduction

![Cost](https://img.shields.io/badge/Cost-Savings-28a745?logo=github&logoColor=white)

Confluence Cloud charges per user seat. GitHub is already licensed across the organisation. Eliminating Confluence reduces SaaS spend without introducing any new tooling.

```mermaid
pie title Annual Tooling Cost Distribution
    "GitHub (existing licence)" : 60
    "Confluence licence (to be eliminated)" : 25
    "Other tools" : 15
```

### 7. Automation and CI/CD Integration

![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white)

Markdown documentation can be validated, linted, and published automatically via GitHub Actions. Broken links, spelling errors, and formatting issues are caught in CI before merging — something Confluence cannot easily enforce.

### 8. Open-Source and Inner-Source Alignment

The docs-as-code model is the standard in open-source. Teams contributing to or consuming open-source software will find GitHub documentation immediately familiar. Inner-source initiatives benefit from the same contribution model.

### 9. Offline Access

Git repositories are fully cloned locally. Engineers have complete access to all documentation without a network connection or VPN — critical during incidents or travel.

### 10. Reduced Vendor Lock-in

Markdown is a plain-text, open format readable by any editor. Moving away from Confluence's proprietary storage format reduces vendor lock-in and preserves long-term readability of historical documentation.

---

## Risk Analysis

| Risk | Mitigation |
|------|-----------|
| Resistance to Markdown authoring | Provide onboarding sessions; VS Code previews and Copilot assist lower the barrier significantly |
| Loss of Confluence-specific features (macros, inline comments) | Evaluate GitHub Discussions for inline Q&A; static site generators replicate most macro functionality |
| Migration effort for existing content | Use automated Confluence-to-Markdown export tools (e.g., `confluence-to-markdown`); prioritise active pages and archive the rest |
| Search quality | GitHub search covers Markdown natively; a hosted static site adds full-text search |
| Access control | GitHub teams and repository visibility settings provide equivalent or finer-grained access control |

---

## Migration Approach

### Migration Timeline

```mermaid
gantt
    title Confluence to GitHub Migration Timeline
    dateFormat  YYYY-MM-DD
    section Phase 1 — Foundation
    Define repo structure & conventions     :p1a, 2026-04-06, 5d
    Set up GitHub Actions pipeline          :p1b, after p1a, 3d
    Configure MkDocs / static site          :p1c, after p1a, 4d
    Create ADR template & guidelines        :p1d, after p1b, 2d

    section Phase 2 — Pilot Migration
    Migrate onboarding / runbook            :p2a, after p1d, 5d
    Gather pilot team feedback              :p2b, after p2a, 3d
    Refine tooling and templates            :p2c, after p2b, 2d

    section Phase 3 — Full Migration
    Export all Confluence spaces            :p3a, after p2c, 5d
    Review and clean exported content       :p3b, after p3a, 7d
    Archive Confluence (read-only)          :p3c, after p3b, 2d
    Stakeholder communication               :p3d, after p3b, 3d

    section Phase 4 — Decommission
    90-day parallel run                     :p4a, after p3d, 30d
    Disable Confluence write access         :p4b, after p4a, 1d
    Cancel Confluence subscription          :p4c, after p4b, 1d
    Redirect Confluence URLs                :p4d, after p4b, 2d
```

### Migration Workflow

```mermaid
flowchart LR
    A([Start]) --> B[Audit Confluence Spaces]
    B --> C{Active or\nArchived?}
    C -->|Active| D[Export to Markdown\nconfluence-to-markdown CLI]
    C -->|Archived| E[Export to ZIP\nread-only archive]
    D --> F[Review & Clean\nMarkdown Content]
    F --> G[Add to GitHub\nRepository /docs]
    G --> H[Raise Pull Request]
    H --> I[Peer Review]
    I --> J{Approved?}
    J -->|Yes| K[Merge to main]
    J -->|No| F
    K --> L[GitHub Actions\nPublish to Pages]
    L --> M([Done])
    E --> N[Store in GitHub\nArchive Repo]
    N --> M

    style A fill:#28a745,color:#fff
    style M fill:#28a745,color:#fff
    style J fill:#f6c90e,color:#000
    style C fill:#f6c90e,color:#000
```

### Phase 1 — Foundation (Weeks 1–2)
- Define repository structure and folder conventions.
- Set up GitHub Actions pipeline for linting and publishing.
- Select and configure a static site generator (MkDocs Material recommended for enterprise use).
- Create ADR template and contribution guidelines.

### Phase 2 — Pilot Migration (Weeks 3–4)
- Migrate one high-traffic documentation area (e.g., onboarding guide or a core service's runbook).
- Gather feedback from pilot team.
- Refine tooling and templates.

### Phase 3 — Full Migration (Weeks 5–8)
- Export all active Confluence spaces to Markdown using automated tooling.
- Review and clean up exported content.
- Archive Confluence spaces (read-only) to preserve historical access during transition.
- Communicate to all stakeholders.

### Phase 4 — Decommission (After 90-day parallel run)
- Disable Confluence write access.
- Cancel Confluence subscription.
- Redirect Confluence URLs to the new documentation site.

---

## Pull Request Workflow for Docs

The sequence diagram below shows how a documentation change flows through the PR process — the same as any code change.

```mermaid
sequenceDiagram
    actor Dev as Developer
    participant GH as GitHub
    participant CI as GitHub Actions
    participant Rev as Reviewer / Tech Lead
    participant Pages as GitHub Pages

    Dev->>GH: git push (code + docs in same branch)
    GH->>CI: Trigger CI pipeline
    CI->>CI: Lint Markdown
    CI->>CI: Spell-check (cspell)
    CI->>CI: Check broken links (lychee)
    CI-->>GH: Report results (pass/fail)
    Dev->>GH: Open Pull Request
    GH-->>Rev: Notify reviewer
    Rev->>GH: Review code AND docs together
    Rev-->>Dev: Request changes (if any)
    Dev->>GH: Push fixes
    Rev->>GH: Approve PR
    GH->>GH: Merge to main
    GH->>CI: Trigger publish pipeline
    CI->>Pages: Build & deploy static site
    Pages-->>Rev: Updated docs live
```

---

## Repository Structure

The recommended folder layout for the shared documentation repository:

```mermaid
graph TD
    ROOT[📁 docs-repo] --> DOCS[📁 docs]
    ROOT --> ADR[📁 adr]
    ROOT --> RUNBOOKS[📁 runbooks]
    ROOT --> ONBOARD[📁 onboarding]
    ROOT --> MKDOCS[📄 mkdocs.yml]
    ROOT --> README[📄 README.md]
    ROOT --> GHA[📁 .github/workflows]

    DOCS --> ARCH[📄 architecture.md]
    DOCS --> API[📄 api-guidelines.md]
    DOCS --> SEC[📄 security.md]

    ADR --> ADR0[📄 0001-use-github-for-docs.md]
    ADR --> ADR1[📄 0002-mkdocs-static-site.md]

    RUNBOOKS --> RB1[📄 incident-response.md]
    RUNBOOKS --> RB2[📄 deployment-checklist.md]

    ONBOARD --> OB1[📄 getting-started.md]
    ONBOARD --> OB2[📄 dev-environment-setup.md]

    GHA --> LINT[📄 lint-docs.yml]
    GHA --> PUBLISH[📄 publish-pages.yml]

    style ROOT fill:#24292e,color:#fff
    style GHA fill:#2088FF,color:#fff
    style ADR fill:#6f42c1,color:#fff
```

---

## Tooling Recommendations

| Purpose | Tool |
|---------|------|
| Content format | Markdown (CommonMark) |
| Static site | MkDocs with Material theme |
| Hosting | GitHub Pages or Azure Static Web Apps |
| CI/CD | GitHub Actions |
| Link checking | `lychee` or `markdown-link-check` |
| Spell checking | `cspell` |
| Export/migration | `confluence-to-markdown` CLI |
| Diagrams | Mermaid (renders natively in GitHub Markdown) |

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Documentation coverage (services with up-to-date README/docs) | ≥ 90% within 6 months |
| Docs updated in same PR as code change | ≥ 80% of feature PRs |
| Time to find documentation (engineer survey) | Reduced by ≥ 30% |
| Confluence license cost | Eliminated within 6 months |
| Stale documentation incidents | Reduced by ≥ 50% |

---

## Conclusion

Migrating to GitHub-based documentation is a strategic investment in engineering efficiency and quality. It closes the gap between code and documentation, applies proven engineering discipline (version control, code review, CI/CD) to prose, and reduces tooling sprawl. The migration is low-risk when phased correctly and delivers compounding returns as teams build confidence with the docs-as-code model.

**Recommendation:** Approve the proposal and authorise a two-week pilot starting in the next sprint cycle.

---

## Viewing This Document on GitHub

This document is authored in **GitHub Flavored Markdown (GFM)**. When pushed to a GitHub repository, everything in this file — badges, diagrams, tables, and code blocks — renders natively in the browser with **zero configuration or plugins required**.

The table below explains what each content type looks like locally versus on GitHub:

| Content Type | In a plain text editor | On GitHub (browser) |
|---|---|---|
| Badges (`shields.io`) | Raw `![text](url)` syntax | Rendered colour badges |
| Mermaid diagrams | Raw ` ```mermaid ``` ` code block | Fully rendered interactive diagram |
| Tables | Pipe-separated text | Formatted grid table |
| Code blocks | Plain fenced text | Syntax-highlighted code |
| Anchor links (TOC) | Plain `[text](#anchor)` | Clickable jump links |
| Headings | `## text` | Formatted headings with auto anchor |

### How to Push This Document to GitHub

#### Step 1 — Create a repository

Go to [github.com/new](https://github.com/new) and create a new repository (e.g. `team-docs` or `migration-proposal`). Set visibility to **Private** for internal proposals.

#### Step 2 — Initialise git locally

Open a terminal in the folder containing this file and run:

```bash
git init
git add confluence-to-github-migration-proposal.md
git commit -m "docs: add Confluence to GitHub migration proposal"
```

#### Step 3 — Push to GitHub

```bash
git remote add origin https://github.com/<your-org>/<your-repo>.git
git branch -M main
git push -u origin main
```

#### Step 4 — Share the rendered URL

Once pushed, navigate to the file in your browser:

```
https://github.com/<your-org>/<your-repo>/blob/main/confluence-to-github-migration-proposal.md
```

Share this URL with the Architecture Board. They will see the full rendered document — badges, all Mermaid UML diagrams, Gantt chart, sequence diagram, mind map, pie chart, and tables — directly in the browser without installing anything.

### What Renders Where

```mermaid
flowchart LR
    FILE[📄 .md File] --> GH[Push to GitHub]
    FILE --> VSC[VS Code Preview]
    FILE --> MKDOCS[MkDocs Build]

    GH --> GH_RENDER[✅ Badges\n✅ Mermaid Diagrams\n✅ Tables\n✅ Code highlighting\n✅ Anchor links\nNo setup needed]

    VSC --> EXT{Extensions\nInstalled?}
    EXT -->|Yes — Mermaid Support| VSC_RENDER[✅ Mermaid Diagrams\n✅ Tables\n⚠️ Badges need setting]
    EXT -->|No| VSC_BASIC[✅ Tables\n❌ Mermaid Diagrams\n❌ Badges]

    MKDOCS --> MK_RENDER[✅ Badges\n✅ Mermaid Diagrams\n✅ Tables\n✅ Full static site\nRequires pip install]

    style GH_RENDER fill:#28a745,color:#fff
    style VSC_RENDER fill:#2088FF,color:#fff
    style VSC_BASIC fill:#dc3545,color:#fff
    style MK_RENDER fill:#6f42c1,color:#fff
    style EXT fill:#f6c90e,color:#000
```

> **Recommendation for this proposal:** Push to a private GitHub repository and share the rendered URL with architects. This requires no installation, proves the concept in practice, and delivers the best rendering experience.

---

## Appendix: Comparison Table

| Capability | Confluence | GitHub (Markdown + Pages) |
|------------|-----------|--------------------------|
| Version history | Page history only | Full Git history |
| Branch-aligned docs | Not supported | Native |
| Pull request review | Not supported | Native |
| CI/CD integration | Limited | Native (GitHub Actions) |
| Offline access | No | Yes (local clone) |
| Cost | Per-seat licence | Included in existing licence |
| Vendor lock-in | High (proprietary format) | Low (plain Markdown) |
| Diagram support | Paid macros | Mermaid (free, built-in) |
| Search | Confluence only | GitHub + static site |
| ADR traceability | Manual | Linked to commits/PRs |
