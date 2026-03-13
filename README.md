# Agent Skills

Curated skills for AI coding agents (Claude Code, Cursor, Windsurf, etc.) that teach your agent **battle-tested patterns** instead of letting it guess.


## What's Inside

| Skill | Rules | Covers |
|-------|------:|--------|
| **[laravel-lucid-architecture](skills/laravel-lucid-architecture)** | 14 | Domains, Features, Operations, Jobs, Data Objects, Testing |
| **[nextjs-best-practices](skills/nextjs-best-practices)** | 11 | App Router, RSC, Server Actions, Caching, Streaming, Middleware |

Every rule includes:
- **Why** it matters (not just the what)
- Bad vs. good code examples
- Impact priority (Critical → Low)

## Quick Start

### Next.js (via [skills.sh](https://skills.sh/))

```bash
npx skills add adelstiti/agent-skills --skill nextjs-best-practices
```

### Laravel (via [Laravel Boost](https://laravelboost.com/))

```bash
php artisan boost:add-skill adelstiti/agent-skills --skill laravel-lucid-architecture
```

## Skills Breakdown

### Laravel Lucid Architecture

A comprehensive guide to building scalable Laravel 12 apps with the [Lucid Architecture](https://lucidarch.dev) methodology.

**Categories:** Foundation & Structure, Features & Operations, Domains & Services, Jobs & Units, Data & Validation, Testing, Advanced Patterns

**Stack:** PHP 8.3+ / Laravel 12.x

**Try it:** *"Structure this Laravel feature using Lucid Architecture"*

---

### Next.js Best Practices

Modern Next.js 15 patterns covering the full App Router mental model — from routing to production optimization.

**Categories:** Routing & Layout, Data Fetching & Caching, Server & Client Components, Server Actions, Performance, Metadata & SEO, Error Handling, Security & Middleware

**Stack:** Next.js 15.x / React 19.x / TypeScript

**Try it:** *"Review this Next.js page for App Router best practices"*

## How It Works

```
skills/
├── laravel-lucid-architecture/
│   ├── SKILL.md          # Main instructions (YAML frontmatter)
│   ├── metadata.json     # Version, categories, references
│   └── rules/            # Individual rules (loaded on demand)
│       ├── foundation-directory-structure.md
│       ├── feature-single-purpose.md
│       └── ...
└── nextjs-best-practices/
    ├── SKILL.md
    ├── metadata.json
    └── rules/
        ├── route-app-directory.md
        ├── comp-server-by-default.md
        └── ...
```

Each skill has a `SKILL.md` with frontmatter metadata and a `rules/` directory. Rules are loaded on demand so your agent only pulls in what's relevant to the current task.

## Contributing

1. Follow the existing rule template in `rules/_template.md`
2. Include incorrect and correct code examples
3. Explain the **why**, not just the what
4. Assign an appropriate priority level

## License

MIT
