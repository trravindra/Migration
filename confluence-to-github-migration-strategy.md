# Migrate HLD Documentation from Confluence to GitHub

---

## Table of Contents

- [1. WHY Documentation in GitHub?](#1-why-documentation-in-github)
- [2. Pros and Cons](#2-pros-and-cons)
  - [✅ Pros — Migrating to GitHub](#-pros--migrating-to-github)
  - [⚠️ Cons / Considerations](#️-cons--considerations)
- [3. Strategy](#3-strategy)
  - [a. Migrating Existing Documentation](#a-migrating-existing-documentation)
    - [Migration Process Overview](#migration-process-overview)
    - [Migration Toolchain Summary](#migration-toolchain-summary)
    - [1. Export from Confluence](#1-export-from-confluence)
    - [2. Parse HTML with BeautifulSoup](#2-parse-html-with-beautifulsoup)
    - [3. Convert to Markdown with Pandoc](#3-convert-to-markdown-with-pandoc)
    - [4. Render UML Diagrams with Kroki](#4-render-uml-diagrams-with-kroki)
    - [5. Restructure](#5-restructure)
    - [6. Upload to GitHub](#6-upload-to-github)
    - [7. Validate & Review](#7-validate--review)
  - [b. Templates for New Documentation](#b-templates-for-new-documentation)
- [4. Demo](#4-demo)
- [5. Questions](#5-questions)

---
 
## 1. WHY Documentation in GitHub?
 
- **Single Source of Truth** — Keep code and its design documentation in the same repository, eliminating context-switching between tools.
- **Version Control** — Every change to a document is tracked with full commit history, diffs, and blame.
- **Review Process** — Leverage Pull Requests for document reviews, approvals, and inline comments — the same workflow used for code.
- **Docs-as-Code** — Treat documentation with the same rigor as source code: linting, CI checks, and automated publishing.
- **Cost Optimization** — Reduce Confluence licensing costs by consolidating onto a platform the engineering team already pays for.
 
---
 
## 2. Pros and Cons
 
 
### ✅ Pros — Migrating to GitHub
 
| # | Topic | Detail |
|---|-------|--------|
| 1 | **Docs next to code** | Documentation and code changes ship in the same pull request, eliminating drift between the two. |
| 2 | **Full Git history** | Every change is versioned with author, timestamp, and commit message — a complete, tamper-evident audit trail. |
| 3 | **Pull request review** | Documentation goes through the same PR approval workflow as code, enforcing quality gates and governance. |
| 4 | **Native CI/CD** | GitHub Actions lints, spell-checks, and link-checks every Markdown file automatically before merge. |
| 5 | **Zero additional licence cost** | GitHub is already licensed across the organisation; Confluence's per-seat fee is eliminated entirely. |
| 6 | **Offline access** | A `git clone` gives engineers full documentation access without a VPN or network connection. |
| 7 | **Open, portable format** | Plain Markdown files are readable in any editor|
| 8 | **Native Mermaid diagrams** | GitHub renders Mermaid diagrams natively in the browser at no cost, replacing paid Confluence diagram macros. |
| 9 | **Consistent developer tooling** | VS Code, GitHub Copilot, and all standard dev tools work natively with Markdown, lowering the barrier to contribution. |
 
### ⚠️ Cons / Considerations
 
| # | Topic | Detail | Mitigation |
|---|-------|--------|------------|
| 1 | **Markdown learning curve** | Non-technical writers unfamiliar with Markdown need initial onboarding. | Short workshop + VS Code preview + GitHub Copilot assist eliminates most friction within days. |
| 2 | **Loss of Confluence macros** | Rich macros (status badges, roadmap timelines, inline tasks) have no 1-to-1 Markdown equivalent. | GitHub Discussions covers inline Q&A; MkDocs plugins and shields.io badges replicate most visual macros. |
| 3 | **No native inline page comments** | Confluence supports inline paragraph-level comments; GitHub does not replicate this outside of PRs. | GitHub Discussions threads can anchor to specific sections; PR comments serve this purpose during the review cycle. |
 
 
---
 
## 3. Strategy
 
### a. Migrating Existing Documentation

#### Migration Process Overview

```mermaid
flowchart TD
    A[Confluence Pages] --> B[Step 1: Export from Confluence]
    B --> |HTML Content| C[Step 2: Parse HTML with BeautifulSoup]
    C --> |Cleaned HTML| D[Step 3: Convert to Markdown with Pandoc]
    C --> |Extract Diagrams| E[Step 4: Render UML Diagrams with Kroki]
    D --> |Draft Markdown| F[Step 5: Restructure Content]
    E --> |SVG/PNG Images| F
    F --> |Organized Files| G[Step 6: Upload to GitHub with requests]
    G --> |Create Branch & PR| H[Step 7: Validate & Review]
    H --> |CI Checks| I{Quality Gates Pass?}
    I -->|Yes| J[Merge to Main]
    I -->|No| K[Fix Issues]
    K --> H
    J --> L[Documentation Live on GitHub]
    
    style A fill:#0288d1,stroke:#01579b,stroke-width:2px,color:#fff
    style B fill:#f57c00,stroke:#e65100,stroke-width:2px,color:#fff
    style C fill:#7b1fa2,stroke:#4a148c,stroke-width:2px,color:#fff
    style D fill:#388e3c,stroke:#1b5e20,stroke-width:2px,color:#fff
    style E fill:#fbc02d,stroke:#f57f17,stroke-width:2px,color:#000
    style F fill:#c2185b,stroke:#880e4f,stroke-width:2px,color:#fff
    style G fill:#00796b,stroke:#004d40,stroke-width:2px,color:#fff
    style H fill:#689f38,stroke:#33691e,stroke-width:2px,color:#fff
    style I fill:#ffa726,stroke:#f57c00,stroke-width:3px,color:#000
    style J fill:#66bb6a,stroke:#2e7d32,stroke-width:3px,color:#fff
    style K fill:#ef5350,stroke:#c62828,stroke-width:2px,color:#fff
    style L fill:#42a5f5,stroke:#1565c0,stroke-width:3px,color:#fff
```

**Migration Flow Table**

| Step | Phase | Tool | Input → Output |
|:----:|-------|------|----------------|
| **1** | **Export from Confluence** | `requests` + Confluence REST API | Confluence Pages → HTML Content |
| **2** | **Parse HTML with BeautifulSoup** | `BeautifulSoup` | HTML Content → Cleaned HTML + Extracted Elements |
| **3** | **Convert to Markdown with Pandoc** | `pandoc` (GFM) | Cleaned HTML → Draft Markdown |
| **4** | **Render UML Diagrams with Kroki** | Kroki API | Diagram Source → SVG/PNG Images |
| **5** | **Restructure** | Manual / Script | Draft Markdown + Images → Organized File Structure |
| **6** | **Upload to GitHub** | `git` | Organized Files → GitHub Branch |
| **7** | **Validate & Review** | PR + CI (link-check, linting) | GitHub PR → Approved Documentation |

---
 
#### 1. Export from Confluence
   - Export Confluence pages as HTML via the Confluence REST API.
   - Use Confluence REST API using Atlassian Bot account.
 
#### 2. Parse HTML with BeautifulSoup
   - Use `BeautifulSoup` to parse the exported HTML content.
   - Extract and clean the document body — strip Confluence-specific macros, inline styles, and wrapper `<div>` elements.
   - Identify and extract embedded image URLs (`<img>` tags), table structures, and code blocks for proper downstream conversion.
   - Handle Confluence-specific elements (status macros, user mentions, page links) and map them to Markdown equivalents or plain text.
 
#### 3. Convert to Markdown with Pandoc
   - Pipe the cleaned HTML through `pandoc` to convert to well-structured Markdown:
     ```bash
     pandoc -f html -t gfm -o output.md input.html
     ```
   - Use GitHub Flavored Markdown (`gfm`) as the target format for native GitHub rendering.
   - Post-process the Pandoc output to fix any remaining formatting artifacts (broken tables, excess whitespace, heading levels).
 
#### 4. Render UML Diagrams with Kroki
   - Extract UML diagram source from Confluence pages (PlantUML, draw.io XML, Mermaid, etc.).
   - Send diagram source to the **Kroki** API to render as SVG/PNG:
     ```
     POST https://kroki.io/{diagram_type}/svg
     ```
   - Supported diagram types: PlantUML, Mermaid, and many more.
   - For PlantUML and other formats not natively supported by GitHub, render via Kroki API and save images to `docs/hld/{feature}/images/`.
   - For Mermaid diagrams, keep them as fenced code blocks since GitHub renders them natively.
   - Optionally preserve PlantUML source in collapsible sections for future re-rendering.
 
#### 5. Restructure
   - Organize documents in a consistent folder structure within the repo:
     ```
     docs/
     ├── hld/
     │   ├── feature-a/
     │   │   ├── hld-feature-a.md
     │   │   └── images/
     │   ├── feature-b/
     │   │   ├── hld-feature-b.md
     │   │   └── images/
     │   └── _templates/
     │       └── hld-template.md
     ```
 
#### 6. Upload to GitHub
   - Commit the converted Markdown files and images to a new branch:
     ```bash
     git checkout -b docs/hld-migration
     git add docs/
     git commit -m "docs: migrate HLD for Feature A from Confluence"
     git push origin docs/hld-migration
     ```
   - Open a Pull Request on GitHub for review.
 
#### 7. Validate & Review
   - Open a Pull Request for each migrated document.
   - Run link-check and Markdown linting CI jobs.
   - Get sign-off from document owners / mergers.
 
---

### b. Templates for New Documentation
 
Provide a standardized HLD template so all future documents are consistent. Below is a comprehensive template with examples:

---

# [Service / Feature Name] — High-Level Design

## Table of Contents
- [1. Preface](#1-preface)
  - [1.1 References](#11-references) - mandatory
    - [1.1.1 Document references](#111-document-references)
    - [1.1.2 Creation and reviews](#112-creation-and-reviews)
- [2. Executive Summary](#2-executive-summary) - mandatory
- [3. Context](#3-context)
  - [3.1 Architecture principles](#31-architecture-principles)
  - [3.2 Technical Constraints](#32-technical-constraints)
  - [3.3 Technical Assumptions](#33-technical-assumptions)
  - [3.4 Discarded solutions](#34-discarded-solutions)
- [4. Technical Overview](#4-technical-overview) - mandatory
  - [4.1 Flow/Sequence Diagram](#41-flowsequence-diagram) - mandatory
  - [4.2 Class Diagram (detail design)](#42-class-diagram-detail-design) - optional
  - [4.3 Impacted Deployment units and Operational Details](#43-impacted-deployment-units-and-operational-details) - mandatory
  - [4.4 Impacted Fields/Methods (detail design)](#44-impacted-fieldsmethods-detail-design) - optional
  - [4.5 Introduced behavior](#45-introduced-behavior) - mandatory
  - [4.6 API updates](#46-api-updates) - mandatory
  - [4.7 Monitoring](#47-monitoring) - mandatory
- [5. Database Models](#5-database-models) - mandatory (explicit if not used)
  - [5.1 Example: Oracle/Couchbase/mongo](#51-example-oraclecouchbasemongo)
- [6. Non-functional Requirements Fulfillment](#6-non-functional-requirements-fulfillment)
  - [6.1 Application Design requirements](#61-application-design-requirements)
  - [6.2 Application Security requirements](#62-application-security-requirements)
  - [6.3 Data Management requirements](#63-data-management-requirements)
  - [6.4 Dev Life Cycle requirements](#64-dev-life-cycle-requirements)
- [7. Glossary](#7-glossary)
  - [7.1 Functional terms / acronyms](#71-functional-terms--acronyms)
  - [7.2 Project and technical terms / acronyms](#72-project-and-technical-terms--acronyms)
- [8. Appendices](#8-appendices)

---

## Document Metadata
 
| Field           | Value                        |
|-----------------|------------------------------|
| **Author(s)**   | John Doe, Jane Smith         |
| **Reviewer(s)** | Tech Lead, Solution Architect |
| **Status**      | Draft / In Review / Approved |
| **Created**     | 2026-04-08                   |
| **Last Updated**| 2026-04-08                   |
| **Version**     | 1.0                          |

---
 
## 1. Preface

This document should be reviewed in the Architecture Corner and presented in the DEV Forum.

### 1.1 References - mandatory

#### 1.1.1 Document references

| Ref | Title | Version | Link |
|-----|-------|---------|------|
| HLS Mandatory | | | |
| HANDOVER PAGE Mandatory | | | |
| Spec Overview Doc | | | |
| Assessment Page | | | |
| Test plan | | | |

#### 1.1.2 Creation and reviews

| Date | Description | Author |
|------|-------------|--------|
| | Written by | |
| | Reviewed by | |
| | Approved by | |

---

## 2. Executive Summary - mandatory

[Provide a brief executive summary of the high-level design. This should be a high-level overview that can be understood by non-technical stakeholders, covering the purpose, scope, and key decisions made in the design.]

---

## 3. Context

### 3.1 Architecture principles

Application Architecture Principles can be referred for applicable general principles to mention they are respected. If you wish you can list explicitly the principle you respect and those non applicable.

But what is more important is to mention if a principle is not going to be followed (quite unlikely though) to justify it (would need approval by Architect Lead).

This section is not only for these general principles but allow additional more detailed architecture principles defined at this application level to be listed.

**Applicable Principles:**
- [List architecture principles being followed]
- 

**Non-Applicable Principles:**
- [List principles not applicable with justification]
- 

**Application-Specific Principles:**
- [List any additional principles defined for this application]
- 

### 3.2 Technical Constraints

[Document technical constraints that impact the design, such as:]
- Technology stack limitations
- Infrastructure constraints
- Third-party integration requirements
- Compliance and regulatory requirements
- Performance requirements
- Budget constraints

### 3.3 Technical Assumptions

[Document key technical assumptions made during the design, such as:]
- Availability of certain services or APIs
- Data volume and growth projections
- User load and traffic patterns
- Network bandwidth and latency
- Team skill sets and availability

### 3.4 Discarded solutions

Details on the process that gave the chosen solution.

| Alternative Solution | Description | Reason for Rejection |
|---------------------|-------------|---------------------|
| [Solution 1] | [Brief description] | [Why it was discarded] |
| [Solution 2] | [Brief description] | [Why it was discarded] |

---

## 4. Technical Overview - mandatory

This chapter gives an overview of the application's main functionalities and the data entities with which the application interacts.

[Provide a high-level overview of the solution, including:]
- Main functionalities and features
- Key components and their interactions
- Data entities and their relationships
- Integration points with other systems

### 4.1 Flow/Sequence Diagram - mandatory

The diagram representing the detailed flow:

```mermaid
sequenceDiagram
    participant Client
    participant API Gateway
    participant Service
    participant Database
    
    Client->>API Gateway: Request
    API Gateway->>Service: Forward Request
    Service->>Database: Query Data
    Database-->>Service: Return Data
    Service-->>API Gateway: Response
    API Gateway-->>Client: Return Response
```

> **Note:** For PlantUML diagrams not natively supported by GitHub, render via Kroki API and save images to `docs/hld/{feature}/images/`. For Mermaid diagrams, keep them as fenced code blocks since GitHub renders them natively.

### 4.2 Class Diagram (detail design) - optional

Detailing the impacted Class diagram:

```mermaid
classDiagram
    class ComponentA {
        +String id
        +String name
        +processRequest()
        +validateData()
    }
    class ComponentB {
        +String id
        +getData()
        +updateData()
    }
    class DataModel {
        +String field1
        +Integer field2
        +save()
        +load()
    }
    
    ComponentA --> ComponentB : uses
    ComponentB --> DataModel : manages
```

[Provide detailed class diagram showing impacted classes, their attributes, methods, and relationships]

### 4.3 Impacted Deployment units and Operational Details - mandatory

Details on the OBEs impacted and operational specificity:

| Unit | Details |
|------|---------|
| BMS component | [Specify BMS component details] |
| Backend Type | [Specify backend type: Stateless/Stateful/Batch] |
| Load Item | [Specify load item configuration] |
| OBE | [Specify OBE (Operating Business Entity) impacted] |

**Operational Details:**
- Deployment approach: [Blue-Green / Rolling / Canary]
- Rollback strategy: [Details]
- Monitoring requirements: [Specific metrics to track]
- Configuration changes: [Environment variables, feature flags, etc.]

### 4.4 Impacted Fields/Methods (detail design) - optional

Details on the Methods and fields touched.

| Class/Component | Method/Field | Type | Change Description |
|----------------|--------------|------|-------------------|
| [ClassName] | [methodName()] | Method | [Description of change] |
| [ClassName] | [fieldName] | Field | [Description of change] |
| [ClassName] | [methodName()] | Method | [Description of change] |

**Code Impact Summary:**
- New methods added: [count]
- Modified methods: [count]
- Deprecated methods: [count]
- New fields added: [count]
- Modified fields: [count]

### 4.5 Introduced behavior - mandatory

Details and explanations on the old and new behaviors. Also the algorithms description detailed in the diagram below.

#### Old Behavior

[Describe the current/old behavior of the system]

**Flow:**
1. [Step 1 of old behavior]
2. [Step 2 of old behavior]
3. [Step 3 of old behavior]

**Limitations:**
- [Limitation 1]
- [Limitation 2]

#### New Behavior

[Describe the new behavior being introduced]

**Flow:**
1. [Step 1 of new behavior]
2. [Step 2 of new behavior]
3. [Step 3 of new behavior]

**Improvements:**
- [Improvement 1]
- [Improvement 2]

#### Behavior Comparison

| Aspect | Old Behavior | New Behavior |
|--------|-------------|--------------|
| Performance | [Details] | [Details] |
| Data Flow | [Details] | [Details] |
| Error Handling | [Details] | [Details] |
| User Experience | [Details] | [Details] |

#### Algorithm Description

```mermaid
flowchart TD
    A[Start] --> B{Condition Check}
    B -->|Yes| C[Process Path A]
    B -->|No| D[Process Path B]
    C --> E[Validate Result]
    D --> E
    E --> F{Valid?}
    F -->|Yes| G[Success]
    F -->|No| H[Error Handler]
    H --> I[Log Error]
    I --> J[End]
    G --> J
```

[Provide detailed explanation of the algorithm, including:]
- Input parameters and validation
- Processing logic and decision points
- Output format and structure
- Error handling and edge cases

### 4.6 API updates - mandatory

Details on the API changes: low/high level API updates (i.e., Services/messages updates) detailed if any.

#### Current API Interface (Before)

**Interface Diagram:**

```mermaid
classDiagram
    class IResourceService {
        <<Interface>>
        +getResource(id: String): Resource
        +createResource(data: ResourceData): Resource
        +updateResource(id: String, data: ResourceData): Resource
        +listResources(): List~Resource~
    }
    
    class IValidationService {
        <<Interface>>
        +validate(input: Object): ValidationResult
        +sanitize(input: String): String
    }
    
    class ResourceServiceImpl {
        -database: Database
        +getResource(id: String): Resource
        +createResource(data: ResourceData): Resource
        +updateResource(id: String, data: ResourceData): Resource
        +listResources(): List~Resource~
    }
    
    IResourceService <|.. ResourceServiceImpl
    ResourceServiceImpl ..> IValidationService
```

---

#### New API Interface (After)

**Interface Diagram:**

```mermaid
classDiagram
    class IResourceService {
        <<Interface>>
        +getResource(id: String, options: QueryOptions): CompletableFuture~Resource~
        +createResource(data: ResourceData, context: RequestContext): CompletableFuture~Resource~
        +updateResource(id: String, data: ResourceData, context: RequestContext): CompletableFuture~Resource~
        +patchResource(id: String, updates: Map~String,Object~, context: RequestContext): CompletableFuture~Resource~
        +deleteResource(id: String, context: RequestContext): CompletableFuture~Boolean~
        +listResources(filter: FilterCriteria, pagination: PaginationParams): CompletableFuture~PagedResult~Resource~~
        +subscribeToEvents(callback: EventCallback): Subscription
    }
    
    class IValidationService {
        <<Interface>>
        +validate(input: Object, schema: Schema): ValidationResult
        +validateAsync(input: Object, schema: Schema): CompletableFuture~ValidationResult~
        +sanitize(input: String, options: SanitizeOptions): String
        +validateBulk(inputs: List~Object~, schema: Schema): List~ValidationResult~
    }
    
    class ICacheService {
        <<Interface>>
        +get(key: String): Optional~Object~
        +set(key: String, value: Object, ttl: Duration): Boolean
        +invalidate(key: String): Boolean
        +invalidatePattern(pattern: String): Integer
    }
    
    class IEventPublisher {
        <<Interface>>
        +publish(event: Event): CompletableFuture~PublishResult~
        +publishBatch(events: List~Event~): CompletableFuture~BatchPublishResult~
    }
    
    class ResourceServiceImpl {
        -database: Database
        -cache: ICacheService
        -eventPublisher: IEventPublisher
        +getResource(id: String, options: QueryOptions): CompletableFuture~Resource~
        +createResource(data: ResourceData, context: RequestContext): CompletableFuture~Resource~
        +updateResource(id: String, data: ResourceData, context: RequestContext): CompletableFuture~Resource~
        +patchResource(id: String, updates: Map~String,Object~, context: RequestContext): CompletableFuture~Resource~
        +deleteResource(id: String, context: RequestContext): CompletableFuture~Boolean~
        +listResources(filter: FilterCriteria, pagination: PaginationParams): CompletableFuture~PagedResult~Resource~~
        +subscribeToEvents(callback: EventCallback): Subscription
    }
    
    IResourceService <|.. ResourceServiceImpl
    ResourceServiceImpl ..> IValidationService
    ResourceServiceImpl ..> ICacheService
    ResourceServiceImpl ..> IEventPublisher
```

---

### 4.7 Monitoring - mandatory

Details what need to be implemented to monitor the functionality (Kibana, Argos, Splunk, Sentinel, Error Viewer).

---

## 5. Database models - mandatory (explicit if not used)

Details of the data model changes if any. Link to seating data model here: **TOADD**

### 5.1 Example: Oracle/Couchbase/MongoDB

#### Database Model Diagram

**Figure 3 – Database model for Oracle**

*[Add ER diagram here using Mermaid or image reference]*

```mermaid
erDiagram
    TABLE_NAME_1 ||--o{ TABLE_NAME_2 : "relationship"
    TABLE_NAME_1 {
        string column1 PK
        string column2
        int column3
    }
    TABLE_NAME_2 {
        string column1 PK
        string column2 FK
        timestamp created_at
    }
```

#### Table Definitions

| Table | Purpose |
|-------|---------|
| *[Table Name 1]* | *[Describe in a few sentences the purpose and specific characteristics of this table]* |
| *[Table Name 2]* | *[Describe in a few sentences the purpose and specific characteristics of this table]* |
| *[Table Name 3]* | *[Describe in a few sentences the purpose and specific characteristics of this table]* |

---

## 6. Non-functional requirements fulfillment

This chapter describes in summary how the major corporate non-functional requirements are addressed (refer to Nano).

While filling the HLD, it is the right time to log they have been reviewed and keep a trace of the work done in Nano in the application assessment section.

If the currently designed application is new and not yet listed in the reference list of application, it is the perfect time to request it's addition by pressing on the + button : 

**Tip 1:** In Nano, assessment can be done per NFR (no more need to do a full application assessment) so you can do this step by step or share the work among various stakeholders of the design.  

**Tip 2:** If this design relates to an existing application, previous Nano assessment answer can be re-used. Although critical eyes is required to review then if they need to be updated

### 6.1 Application Design requirements
 
### 6.2 Application Security requirements
 
### 6.3 Data Management requirements
 
### 6.4 Dev Life Cycle requirements
 
---

## 7. Glossary

Overall R&D Glossary : https://intranet.amadeus.com/glossary/Pages/default.aspx

### 7.1 Functional terms / acronyms

| Term / acronym | Definition |
|----------------|------------|
|  |  |
|  |  |
|  |  |

### 7.2 Project and technical terms / acronyms

| Term / acronym | Definition | Term / acronym | Definition |
|----------------|------------|----------------|------------|
|  |  |  |  |
|  |  |  |  |
|  |  |  |  |

---

## 8. Appendices

---

**Document History:**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-04-08 | John Doe | Initial draft |
| 1.1 | 2026-04-10 | Jane Smith | Added security section |

---.2 | 2026-04-12 | John Doe | Updated based on review feedback |
```
 
> **Tip:** Store the template at `docs/hld/_templates/hld-template.md` and reference it in your `CONTRIBUTING.md`.
