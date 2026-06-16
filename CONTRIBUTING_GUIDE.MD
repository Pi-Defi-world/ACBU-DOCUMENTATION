# Contributing to ACBU (African Currency Basket Unit)

Thank you for your interest in contributing to ACBU! This guide outlines the process for contributing to all ACBU repositories, including documentation, backend, frontend, and smart contracts.

---

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Project Structure](#project-structure)
- [How to Report Issues](#how-to-report-issues)
- [How to Submit Changes](#how-to-submit-changes)
- [Branch Strategy](#branch-strategy)
- [Commit Message Conventions](#commit-message-conventions)
- [Documentation Contributions](#documentation-contributions)
- [Code Contributions](#code-contributions)
  - [Backend (Node/Express)](#backend-nodeexpress)
  - [Frontend (Next.js)](#frontend-nextjs)
  - [Smart Contracts (Soroban/Rust)](#smart-contracts-sorobanrust)
- [Pull Request Process](#pull-request-process)
- [Review Guidelines](#review-guidelines)
- [Community & Communication](#community--communication)

---

## Code of Conduct

All contributors are expected to:

- Be respectful and inclusive in all interactions
- Provide constructive feedback focused on the work, not the person
- Respect differing viewpoints and experiences
- Accept constructive criticism gracefully
- Prioritize the project's mission of financial inclusion across Africa

Violations may result in removal from the project at the maintainers' discretion.

---

## Project Structure

ACBU spans multiple repositories:

| Repository | Description | Stack |
| :--- | :--- | :--- |
| `ACBU-DOCUMENTATION` | Project documentation, specs, and planning | Markdown |
| `acbu-backend` | Node/Express API server | TypeScript, Prisma, MongoDB, RabbitMQ |
| `acbu-frontend` | Next.js web application | TypeScript, React, TailwindCSS |
| `acbu-smart-contract` | Soroban smart contracts | Rust (no_std), Soroban SDK |

### Documentation Map

Key documents to read before contributing:

- **[README.MD](README.MD)** — Project overview and quick facts
- **[TECHNICAL/ARCHITECTURE.MD](TECHNICAL/ARCHITECTURE.MD)** — System architecture
- **[TECHNICAL/API_SPECIFICATION.MD](TECHNICAL/API_SPECIFICATION.MD)** — API endpoints
- **[PROJECT/SMART_CONTRACT_SPEC.MD](PROJECT/SMART_CONTRACT_SPEC.MD)** — Contract interfaces
- **[PROJECT/STAGES.MD](PROJECT/STAGES.MD)** — Development phases
- **[issues/MASTER_INDEX.md](issues/MASTER_INDEX.md)** — Known issues backlog (~200 items)

---

## How to Report Issues

### For Bug Reports

Create an issue with:

1. **Title:** Clear, descriptive summary (e.g., "B-001: Mint basket deposit limits use wrong USD proxy")
2. **Severity:** Critical / High / Medium / Low
3. **Area:** Backend / Frontend / Contracts / Documentation
4. **Evidence:** File path(s) and line(s) affected
5. **Impact:** What breaks, what's at risk
6. **Steps to Reproduce:** Clear, minimal reproduction steps
7. **Expected vs Actual Behavior**
8. **Environment:** OS, Node version, network (testnet/mainnet)

### For Feature Requests

Create an issue with:

1. **User Story:** As a [user type], I want [feature] so that [benefit]
2. **Acceptance Criteria:** Concrete, testable outcomes
3. **Priority:** P0 (ship-blocking) through P4 (nice-to-have)
4. **Relevant Specs:** Link to any related documentation

### Issue Labels

| Label | Meaning |
| :--- | :--- |
| `critical` | Loss of funds, auth bypass, or chain halt |
| `bug` | Incorrect behavior |
| `feature` | New capability |
| `documentation` | Docs-only change |
| `frontend` | UI/UX layer |
| `backend` | API/server layer |
| `contracts` | On-chain (Soroban) layer |
| `good first issue` | Suitable for new contributors |

---

## How to Submit Changes

### Step 1: Fork and Clone

```bash
# Fork the repository on GitHub
# Clone your fork
git clone https://github.com/<your-username>/ACBU-DOCUMENTATION.git
cd ACBU-DOCUMENTATION

# Add upstream remote
git remote add upstream https://github.com/Pi-Defi-world/ACBU-DOCUMENTATION.git
```

### Step 2: Create a Branch

```bash
git checkout -b <type>/<description>
```

Examples:
- `fix/auth-bypass-mint`
- `feat/savings-goal-create`
- `docs/contributing-guide`
- `test/escrow-lifecycle`

### Step 3: Make Your Changes

- Follow the code style of the surrounding files
- Keep changes focused and minimal — one concern per PR
- Add or update tests for your changes
- Update documentation if your change affects APIs or user flows

### Step 4: Test Locally

**Documentation only:**
```bash
# Proofread; check links with a Markdown linter
```

**Backend:**
```bash
cd acbu-backend
pnpm install
pnpm typecheck          # TypeScript compilation
pnpm test               # Run tests
pnpm prisma generate    # Regenerate Prisma client
```

**Frontend:**
```bash
cd acbu-frontend
pnpm install
pnpm typecheck          # TypeScript compilation
pnpm build              # Production build check
```

**Smart Contracts:**
```bash
cd acbu-smart-contract
cargo build --target wasm32-unknown-unknown --release
cargo test
cargo clippy -- -D warnings
```

### Step 5: Commit

Follow the [commit message conventions](#commit-message-conventions) below.

### Step 6: Push and Open a PR

```bash
git push origin <your-branch>
```

Then open a Pull Request against the `main` branch of the upstream repository.

---

## Branch Strategy

| Branch | Purpose |
| :--- | :--- |
| `main` | Production-ready, always deployable |
| `develop` | Integration branch (if used) |
| `feat/<name>` | New features |
| `fix/<name>` | Bug fixes |
| `docs/<name>` | Documentation updates |
| `test/<name>` | Test additions |
| `chore/<name>` | Maintenance, deps, config |

**Rules:**
- Never commit directly to `main`
- Always create a feature/fix branch from `main` (or `develop` if in use)
- Rebase onto `main` before opening a PR to avoid merge conflicts
- Squash commits before merging (maintainers' discretion)

---

## Commit Message Conventions

Follow the [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <short summary>

<optional body — explain what and why, not how>

<optional footer — references to issues>
```

### Types

| Type | Usage |
| :--- | :--- |
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, missing semicolons, etc. (no code change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `chore` | Maintenance, dependency updates, tooling |
| `perf` | Performance improvement |

### Scopes

Backend: `auth`, `mint`, `burn`, `transfer`, `savings`, `lending`, `kyc`, `webhook`, `limits`, `reserve`, `wallet`, `jobs`

Frontend: `auth`, `send`, `savings`, `mint`, `bills`, `lending`, `currency`, `wallet`, `settings`, `kyc`, `api`, `ui`

Contracts: `escrow`, `minting`, `burning`, `oracle`, `reserves`, `savings`, `lending`, `shared`

Documentation: `spec`, `architecture`, `business`, `ux`

### Examples

```
fix(backend/auth): use bcrypt.compare instead of hash for API key validation

The validateApiKey middleware was using bcrypt.hash() for lookup,
which produces a different hash each time due to random salt.
Switched to bcrypt.compare() to properly verify stored hashes.

Closes #42
```

```
feat(contracts/escrow): add require_auth to release function

Prevents unauthorized release of escrowed funds by requiring
the original payer's authorization.

Closes #1 (C-001)
```

```
docs: add contributing guide

Closes #1
```

---

## Documentation Contributions

### Style Guidelines

- Use **semantic headings** (H1 for title, H2 for sections, H3 for subsections)
- Use **tables** for structured data (fees, comparisons, matrices)
- Use **bold** for emphasis, not ALL CAPS
- Link to other documents using **relative paths** from repo root
- Keep files under ~500 lines; split into multiple files if needed
- End files with a metadata footer:

```
---
**Last Updated:** [Date]
**Version:** X.Y
**Status:** [Draft / Review / Approved]
```

### Documentation Review Checklist

- [ ] All internal links resolve
- [ ] Technical terms are defined or linked
- [ ] Examples are concrete and testable
- [ ] Diagrams have alt text descriptions
- [ ] Version and status footer is present
- [ ] No broken English or unclear phrasing

---

## Code Contributions

### Backend (Node/Express)

**Tech Stack:** TypeScript, Express, Prisma (PostgreSQL), MongoDB (cache/sessions), RabbitMQ (jobs)

**Key Conventions:**
- Use `Decimal` (via `decimal.js` or Prisma Decimal) for all monetary values — never `Number()` or `parseFloat()`
- Validate all inputs with **Zod** schemas in controllers before reaching services
- Use **idempotency keys** for all POST/PUT mutating endpoints
- Log structured JSON with `logger.info({ amount, currency, userId, idempotencyKey }, 'message')`
- Auth middleware (`validateApiKey`) must run before any money-moving handler
- Services return domain objects; controllers handle HTTP concerns
- Never swallow errors in catch blocks — always log and re-throw or return structured error

**Directory Pattern:**
```
src/
├── controllers/    # HTTP handlers (thin — delegate to services)
├── services/       # Business logic
├── middleware/      # Auth, rate limiting, validation
├── jobs/           # RabbitMQ consumers / scheduled jobs
├── config/         # Environment, limits, logger
├── routes/         # Express route definitions
└── utils/          # Shared helpers (JWT, cache, etc.)
```

### Frontend (Next.js)

**Tech Stack:** TypeScript, Next.js (App Router), React, TailwindCSS, shadcn/ui

**Key Conventions:**
- Use **Server Components** by default; `'use client'` only when needed
- Store **auth tokens** in httpOnly cookies, never in `sessionStorage` or `localStorage`
- Format all monetary values with a shared `formatAcbu()` helper
- All API calls go through `lib/api/client.ts` — never use raw `fetch`
- Use React Hook Form + Zod for form validation
- Always show loading skeletons, error states, and empty states
- Add `aria-label` to icon-only buttons and links
- Use semantic Tailwind classes; avoid hardcoded color values

**Directory Pattern:**
```
app/
├── (app)/           # Authenticated routes (dashboard, send, savings, etc.)
├── (public)/        # Public routes (auth, landing)
├── api/             # Next.js API routes (if any)
components/          # Shared UI components
contexts/            # React context providers
hooks/               # Custom React hooks
lib/                 # Utilities, API client, helpers
```

### Smart Contracts (Soroban/Rust)

**Tech Stack:** Rust (`no_std`), Soroban SDK, Stellar

**Key Conventions:**
- Every public `fn` that moves funds MUST call `require_auth()` on the relevant address
- Use `checked_*` math operations for all `i128` monetary arithmetic
- Follow **checks-effects-interactions** pattern to prevent reentrancy
- Import shared constants (`DECIMALS`, `BASIS_POINTS`) from `shared` crate — never redefine
- Emit structured events on every state change
- Avoid `.unwrap()` — use proper `Option`/`Result` handling with descriptive errors
- Pin WASM hashes; never use zeroed SHA256 placeholders in production
- Add integration tests for all entrypoints (happy + sad paths)

**Directory Pattern:**
```
acbu-smart-contract/
├── acbu_minting/
├── acbu_burning/
├── acbu_oracle/
├── acbu_reserve_tracker/
├── acbu_savings_vault/
├── acbu_lending_pool/
├── acbu_escrow/
└── shared/           # Shared constants, utilities, types
```

---

## Pull Request Process

### Before Opening a PR

1. **Review your diff.** Remove debugging code, console.logs, commented-out blocks.
2. **Run all tests.** Ensure nothing is broken.
3. **Run typecheck/lint.** `pnpm typecheck` (backend/frontend) or `cargo clippy` (contracts).
4. **Rebase onto main.** `git rebase upstream/main` to keep history clean.
5. **Write a clear PR description** (see template below).

### PR Description Template

```markdown
## Summary
Brief description of the change (1-2 sentences).

## Motivation
Why is this needed? Link the issue being resolved.

## Changes
- Bullet list of what changed
- Group by file/module if helpful

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing steps completed
- [ ] TypeScript/Clippy clean

## Screenshots (if UI change)
Before / After images or videos.

## Related Issues
Closes #<issue-number>
```

### Review Process

1. PR author requests review from at least one maintainer
2. Reviewer checks for: correctness, security, style, test coverage
3. Address all review comments (either by changing code or explaining why not)
4. Maintainer approves and merges (squash merge preferred)
5. Delete the feature branch after merge

---

## Review Guidelines

### What Reviewers Look For

**Security (always check first):**
- Are auth checks present on all money-moving paths?
- Are inputs validated before use?
- Are secrets handled safely (no logging, no client-side storage)?
- Are there any IDOR / authorization bypass vectors?

**Correctness:**
- Does the code do what it claims to do?
- Are edge cases handled (empty input, zero amounts, large numbers)?
- Is idempotency preserved for mutating operations?

**Style:**
- Does it match the surrounding code conventions?
- Are types used properly (no `as any` without justification)?
- Are function/variable names clear and descriptive?

**Tests:**
- Are happy path and sad path covered?
- Do tests actually assert the right thing?
- Are there tests for the specific bug being fixed?

---

## Community & Communication

### Where to Ask Questions

- **Technical questions:** Open a GitHub Discussion (if enabled) or comment on the relevant issue
- **Bug reports:** Open an issue with the [bug report template](#for-bug-reports)
- **Feature ideas:** Open an issue with the [feature request template](#for-feature-requests)
- **Security vulnerabilities:** Email `security@acbu.io` — do NOT open a public issue

### Recognition

All contributors are recognized in our [README](README.MD) and release notes. Significant contributions may be eligible for rewards through our grant programs.

### License

By contributing, you agree that your contributions will be licensed under the project's [LICENSE](PROJECT/LICENSE).

---

## Quick Reference: Issue-to-PR Workflow

```
1. Find an issue → 2. Comment "I'm working on this" → 3. Fork + branch
                                                          ↓
6. Maintainer merges ← 5. Address review ← 4. Open PR with "Closes #N"
```

---

## Fixing Known Issues

See the issue backlog in [issues/MASTER_INDEX.md](issues/MASTER_INDEX.md) for ~200 documented MVP issues across:

- **[issues/backend.md](issues/backend.md)** — 75 backend issues
- **[issues/frontend.md](issues/frontend.md)** — 65 frontend issues
- **[issues/contracts.md](issues/contracts.md)** — 60 smart contract issues

Start with **Critical** severity items — these involve loss of funds, auth bypass, or false confirmations.

---

**Last Updated:** June 16, 2026
**Version:** 1.0
**Status:** Approved
