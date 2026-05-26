# Skills Registry Project — Execution Tracker

## Project Goal

Build a central, model-agnostic registry of reusable React Native / Expo skills that can be injected into any AI tool's context (Claude Code, Copilot, Cursor, etc.). The goal is to avoid re-deriving the same patterns across projects.

**Scope:** Mobile development ecosystem (React Native / Expo). Start with skills from the current project's `.copilot/` docs and codebase.

---

## Phases

### Phase 1: Markdown Registry (Current)
**Status:** In Progress  
**Goal:** Build the raw Markdown registry with 24 skill files organized by category.  
**Location:** `skills/` (temporary; will move outside project after testing)  
**Deliverables:**
- [ ] `SKILLS_REGISTRY_PROJECT.md` — this file
- [ ] `skills/INDEX.md` — one-line-per-skill scannable index
- [ ] `skills/scaffold/` — 10 sub-skills
- [ ] `skills/api/` — 4 skills
- [ ] `skills/styling/` — 2 skills
- [ ] `skills/architecture/` — 1 skill
- [ ] `skills/utils/` — 4 skills
- [ ] `skills/hooks/` — 3 skills

**Timeline:** ~2 days (extracting from existing code and docs)

**Success Criteria:**
- All 24 skill files exist under `skills/`
- Each skill file has front matter (name, category, stack, keywords, source-files)
- Each skill has 5 sections: Problem, When to Use, Implementation, Usage Example, Gotchas
- INDEX.md is complete and scannable in <30 seconds

---

### Phase 2: CLI Tool (Next)
**Status:** Not Started  
**Goal:** Build a Node.js/Python CLI to manage skills (list, search, show, inject, add, push).  
**Expected Timeline:** ~1 week  
**Why CLI first (not GUI)?**
- AI tools are terminal-native; CLI can integrate into scripts and CI pipelines
- Composable: output can be piped to clipboard, used in prompts, etc.
- Faster to build and test than a GUI

**Minimum Viable CLI:**
```bash
skills list                                # show all skills
skills search "token refresh"             # search by keyword
skills show auth/silent-token-refresh     # print to stdout
skills inject auth/silent-token-refresh   # copy to clipboard (macOS) or stdout
skills add                                # interactive: create new skill
skills push                               # git commit + push to GitHub
```

---

### Phase 3: MCP Server (After Phase 2)
**Status:** Backlog  
**Goal:** Convert CLI backend into an MCP (Model Context Protocol) server.  
**Why?**
- Enables AI models to query skills **autonomously** without manual injection
- Claude Code, Cursor, and future tools can call it directly
- Becomes the context provider for workflow orchestration

**Integration Points:**
- Configure in `~/.claude/settings.json` for Claude Code
- Configure in `.cursorrules` for Cursor
- Configure in GitHub Copilot settings

---

### Phase 4: UI/Dashboard (Future)
**Status:** Backlog  
**Goal:** Optional web UI for exploring, searching, and managing skills.  
**Note:** Only valuable after Phase 3, when the data model is stable. Don't build this before feeling the friction.

---

### Phase 5: Workflow Orchestration (Future)
**Status:** Backlog  
**Goal:** Use the skills registry + CLI/MCP as a context manager in mobile app development workflows.  
**Example:** "Build a feature" workflow automatically injects relevant skills (auth, API, forms, etc.) between steps.

---

## Skills Extracted (24 Total)

### Category: Scaffold (10 skills)
Project setup and foundational patterns. Each is a focused sub-skill, not a monolithic "setup" guide.

| # | Skill | Source | Status |
|---|-------|--------|--------|
| 1 | Expo App Config | `.copilot/PROJECT_CONTEXT.md` + `app.config.ts` | Pending |
| 2 | Expo Router Navigation | `.copilot/PROJECT_CONTEXT.md` + `app/(auth)/_layout.tsx` | Pending |
| 3 | Project Folder Structure | `.copilot/PROJECT_CONTEXT.md` | Pending |
| 4 | Axios Setup | `libs/services/index.ts` | Pending |
| 5 | React Query Setup | `.copilot/ARCHITECTURE.md` + `app/_layout.tsx` | Pending |
| 6 | Auth Context Setup | `.copilot/ARCHITECTURE.md` + `libs/context/AuthContext.tsx` | Pending |
| 7 | Theming Config | `libs/constants/theme.ts` + `.copilot/DESIGN_SYSTEM.md` | Pending |
| 8 | Key Storage Handler | `libs/utils/keyStorage.ts` | Pending |
| 9 | Button Component | `components/ui/button/Button.tsx` + `.copilot/COMPONENT_CATALOG.md` | Pending |
| 10 | Typography Component | `components/ui/heading/Heading.tsx` + `.copilot/COMPONENT_CATALOG.md` | Pending |

### Category: API (4 skills)
Data fetching, error handling, token management.

| # | Skill | Source | Status |
|---|-------|--------|--------|
| 11 | Data Fetching with React Query | `.copilot/ARCHITECTURE.md` | Pending |
| 12 | API Services Pattern | `libs/services/index.ts` | Pending |
| 13 | Token Refresh | `libs/services/index.ts` (interceptor + session manager) | Pending |
| 14 | Session Management | `libs/services/sessionManager.ts` | Pending |

### Category: Styling (2 skills)
Design system, theming, responsive scaling.

| # | Skill | Source | Status |
|---|-------|--------|--------|
| 15 | NativeWind & Tailwind Config | `tailwind.config.js` | Pending |
| 16 | Theming Hooks | `libs/hooks/useTheme.ts` + `useThemedStyles.ts` | Pending |

### Category: Architecture (1 skill)
Folder structure and code organization patterns.

| # | Skill | Source | Status |
|---|-------|--------|--------|
| 17 | Feature-Driven Folder Structure | `.copilot/PROJECT_CONTEXT.md` | Pending |

### Category: Utils (4 skills)
Reusable utility functions extracted from the codebase.

| # | Skill | Source | Status |
|---|-------|--------|--------|
| 18 | Image Handler | `libs/utils/imageHandlers.ts` | Pending |
| 19 | File Downloader | `libs/utils/fileDownloader.ts` | Pending |
| 20 | File Sharing | `libs/utils/shareReceipt.ts` | Pending |
| 21 | Toast Config | `libs/utils/ToastConfig.tsx` | Pending |

### Category: Hooks (3 skills)
Custom React hooks for common patterns.

| # | Skill | Source | Status |
|---|-------|--------|--------|
| 22 | useNotifications | `libs/hooks/useNotifications.ts` | Pending |
| 23 | usePagination | `libs/hooks/usePagination.ts` | Pending |
| 24 | useTimer | `libs/hooks/useTimer.ts` | Pending |

---

## Next Steps

1. Create `skills/INDEX.md`
2. Create all 24 skill files in their respective directories
3. Test the registry by referencing skills in a new project
4. Move the registry outside the project to a central location
5. Begin Phase 2: CLI tool

---

## Notes

- **Starting location:** This project root (`skills/`), temporary for testing
- **Final location:** `~/skills-registry/` (git repo, referenced by all projects)
- **Skill file format:** Markdown with YAML front matter + 5 required sections
- **Audience:** AI tools and developers building React Native / Expo apps