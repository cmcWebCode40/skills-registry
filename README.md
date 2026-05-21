# Mobile Skills Registry

A central, reusable skills library for React Native / Expo development. Share battle-tested patterns across all your projects and inject them into any AI tool's context.

## What Is This?

This is a curated collection of **24 production-ready skills** for building mobile apps with React Native and Expo. Each skill is:

- **Atomic** — One problem, one solution
- **Complete** — Full code, not just descriptions
- **Opinionated** — Based on real project patterns, not theory
- **AI-friendly** — Optimized for model consumption (markdown, structured format, examples)
- **Versioned** — Git-tracked, can evolve alongside your tech stack

## Use Cases

**For you:**
- No more "how did I solve pagination last time?" — it's documented once, reused everywhere
- Onboard new developers faster with working patterns
- Reduce boilerplate across projects
- Share knowledge without meetings

**For AI tools:**
- Claude Code, Copilot, Cursor can reference this registry in their system prompts
- Models inject relevant skills into context automatically
- "Build a feature" workflows become: identify needed skills → inject → implement

## Structure

```
skills/
├── INDEX.md                    # Quick-scan index of all skills
├── scaffold/                   # Project setup (10 skills)
│   ├── expo-app-config.md
│   ├── expo-router-navigation.md
│   ├── project-folder-structure.md
│   ├── axios-setup.md
│   ├── react-query-setup.md
│   ├── auth-context-setup.md
│   ├── theming-config.md
│   ├── key-storage-handler.md
│   ├── ui-button-component.md
│   └── ui-typography-component.md
├── api/                        # Data fetching & auth (4 skills)
│   ├── data-fetching-react-query.md
│   ├── api-services.md
│   ├── token-refresh.md
│   └── session-management.md
├── styling/                    # Design system (2 skills)
│   ├── nativewind-tailwind-config.md
│   └── theming-hooks.md
├── architecture/               # Code organization (1 skill)
│   └── feature-driven-folder-structure.md
├── utils/                      # Utilities (4 skills)
│   ├── image-handler.md
│   ├── file-downloader.md
│   ├── file-sharing.md
│   └── toast-config.md
└── hooks/                      # Custom hooks (3 skills)
    ├── use-notifications.md
    ├── use-pagination.md
    └── use-timer.md
```

## Quick Start

### 1. Browse Available Skills

```bash
# View the index (start here)
cat skills/INDEX.md

# Read a specific skill
cat skills/api/token-refresh.md
```

### 2. Reference a Skill in Your Project

When building a feature, search for relevant skills:

```bash
# Find skills by keyword
grep -r "token refresh" skills/
grep -r "pagination" skills/
```

### 3. Copy Code Into Your Project

Each skill includes:
- **Problem** — What it solves
- **When to Use** — When to apply it
- **Implementation** — Full, runnable code
- **Usage Example** — How to use it
- **Gotchas** — Non-obvious constraints

Copy the Implementation section into your project. Adapt as needed.

## Integration with AI Tools

### Claude Code

Add to your project's `CLAUDE.md`:

```markdown
## Skills Registry

Reusable mobile development patterns are documented in `/path/to/skills-registry/`. 
Before implementing auth, API layers, hooks, or UI components, check the registry for 
an existing skill. If found, reference it in your prompt:

"Build a login form using the react-query data fetching and axios setup skills."

Skills are organized by category: scaffold, api, styling, architecture, utils, hooks.
Full index: `/path/to/skills-registry/skills/INDEX.md`
```

### GitHub Copilot

Add to `.github/copilot-instructions.md`:

```markdown
## Skills Registry

Refer to `/path/to/skills-registry/` for standard patterns before implementing:
- Authentication & session management
- Data fetching with React Query
- Component styling with NativeWind/Tailwind
- Custom hooks and utilities

When implementing features, cite relevant skills if they apply.
```

### Cursor

Add to `.cursorrules`:

```
Reference the skills registry at /path/to/skills-registry/ for:
- Scaffolding new projects
- Auth flows
- API integration patterns
- Custom hooks and utilities

When implementing features, check INDEX.md for relevant skills first.
```

### Other AI Tools

Reference this registry in your system prompt or instructions for any AI tool:

```
Before implementing any feature, consult the skills registry at /path/to/skills-registry/.
Search INDEX.md for relevant patterns. If found, inject the skill's Implementation 
section into your context.
```

## Skill Format

Each skill file follows this structure:

```markdown
---
name: kebab-case-slug
category: scaffold | api | styling | architecture | utils | hooks
stack: [react-native, typescript, expo, ...]
keywords: [keyword1, keyword2, ...]
source-files: [relative/path/to/file.ts]
---

# Skill Title

## Problem
What problem does this solve?

## When to Use
When should you apply this skill?

## Implementation

### Dependencies
npm packages required

### Code
Full, runnable implementation

## Usage Example
How to use it in your code

## Gotchas
Non-obvious constraints, platform differences, gotchas
```

### Creating a Skill File

1. **Identify a pattern** — A solution you've implemented multiple times
2. **Extract the code** — Get the real, production implementation
3. **Write the sections** — Problem, When to Use, Implementation, Usage Example, Gotchas
4. **Add front matter** — name, category, stack, keywords, source-files
5. **Commit to git** — Version your skills

Example:

```bash
cat > skills/api/my-new-skill.md << 'EOF'
---
name: my-new-skill
category: api
stack: [react-native, typescript]
keywords: [keyword1, keyword2]
source-files: [path/to/original/file.ts]
---

# My New Skill

## Problem
...

## When to Use
...

## Implementation
...

## Usage Example
...

## Gotchas
...
EOF

git add skills/api/my-new-skill.md
git commit -m "feat(skills): add my-new-skill"
```

## Discovering Skills

### By Category
- **scaffold/** — Project setup, foundational components, configuration
- **api/** — Data fetching, API clients, auth, error handling
- **styling/** — Design tokens, theming, Tailwind/NativeWind
- **architecture/** — Folder structure, code organization
- **utils/** — Reusable utilities (image handling, file I/O, notifications)
- **hooks/** — Custom React hooks

### By Keyword
Use grep to find skills by topic:

```bash
# Find skills related to tokens
grep -r "token" skills/ --include="*.md"

# Find skills for images
grep -r "image" skills/ --include="*.md"

# Find skills about navigation
grep -r "navigation" skills/ --include="*.md"
```

### By Stack
All skills include a `stack` field in front matter. Search by technology:

```bash
# Find skills using React Query
grep -r "react-query" skills/ --include="*.md"

# Find skills using NativeWind
grep -r "nativewind" skills/ --include="*.md"
```

## Best Practices

### For Using Skills

1. **Read the entire skill file** — Not just the code
2. **Check the gotchas** — Avoid common pitfalls
3. **Adapt to your context** — Skills are templates, not copy-paste
4. **Test thoroughly** — Skills are based on one project; your context may differ

### For Adding Skills

1. **Extract only working patterns** — No speculative code
2. **Include gotchas from experience** — Non-obvious constraints are valuable
3. **Reference the actual source file** — Readers can verify against production code
4. **Keep skills focused** — One problem per skill. Don't merge unrelated patterns
5. **Use real code examples** — Not pseudocode

### For Maintaining Skills

1. **Update when patterns change** — Keep skills in sync with your tech stack
2. **Document breaking changes** — If a dependency updates, update the skill
3. **Deprecate, don't delete** — Mark deprecated skills with a note, keep them for reference
4. **Version alongside your projects** — Use git tags to track skill versions

## What's Included (24 Skills)

### Scaffold (10 skills)
Complete project setup from scratch:
- Expo app config, EAS builds, environment management
- File-based routing with Expo Router
- Folder structure and code organization
- Axios singleton with interceptors
- React Query setup and configuration
- Auth context with session management
- Theme configuration with dark mode
- Two-tier storage (SecureStore + MMKV)
- Button and Typography UI components

### API (4 skills)
Server communication and state management:
- Data fetching patterns with React Query
- Axios service setup
- Silent token refresh with race-condition prevention
- Session management with callbacks

### Styling (2 skills)
Design system and theming:
- NativeWind/Tailwind configuration
- Theme and theming hooks

### Architecture (1 skill)
Code organization:
- Feature-driven folder structure

### Utils (4 skills)
Reusable utilities:
- Image picker with iOS/Android normalization
- File downloader with redirect handling
- PDF generation and sharing
- Toast notification configuration

### Hooks (3 skills)
Custom React hooks:
- Push notification registration and deep linking
- Pagination with infinite scroll
- Countdown timer with app backgrounding support

## Contributing

To add a new skill:

1. Clone this repo
2. Create a file in the appropriate category: `skills/{category}/{skill-name}.md`
3. Follow the skill format (see **Skill Format** above)
4. Test the code in a real project
5. Update `skills/INDEX.md` with one line for your skill
6. Commit and push

## FAQ

### Q: Can I use these skills in production?
**A:** Yes. Each skill is extracted from production code. Test them in your context before shipping.

### Q: Do I need to use all skills?
**A:** No. Pick what you need. This is a menu, not a prescription.

### Q: Can I modify a skill for my project?
**A:** Absolutely. Skills are starting points. Adapt them to your tech stack and coding style.

### Q: What if a skill is outdated?
**A:** Open an issue or PR to update it. Skills should stay in sync with current best practices.

### Q: Can I use this registry for other frameworks (Vue, Flutter, etc.)?
**A:** This registry is React Native / Expo focused. You could fork it and adapt the skills for other stacks.

### Q: How do I reference skills in my prompts to AI tools?
**A:** Example:

```
"Build a login screen using:
1. The react-query data fetching skill (api/data-fetching-react-query.md)
2. The form validation patterns from modules/auth/schema/
3. The Button component from scaffold/ui-button-component.md"
```

The AI tool reads the skill files and applies the patterns.

## License

MIT — Use freely in your projects.

## Changelog

### v1.0.0 (2025-05-21)
- Initial release with 24 skills
- Categories: scaffold, api, styling, architecture, utils, hooks
- Markdown format optimized for AI consumption
- Integration guides for Claude Code, Copilot, Cursor

---

**Last updated:** 2025-05-21  
**Total skills:** 24  
**Last category added:** hooks (use-timer)

For questions or contributions, open an issue or PR on GitHub.
