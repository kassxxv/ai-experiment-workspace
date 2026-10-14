# Resources for Building with AI

> Researched October 2026. Everything listed is free. Companion to [claude-code-recap.md](claude-code-recap.md).

## Table of contents
1. [Skills and plugins for Claude Code](#1-skills-and-plugins-for-claude-code)
2. [MCP servers (connections to other apps)](#2-mcp-servers-connections-to-other-apps)
3. [Building AI apps (an app that has AI inside)](#3-building-ai-apps-an-app-that-has-ai-inside)
4. [UI components (free, shadcn ecosystem)](#4-ui-components-free-shadcn-ecosystem)
5. [UX and design learning](#5-ux-and-design-learning)
6. [Starter kit](#6-starter-kit)

---

## 1. Skills and plugins for Claude Code

### Install first (official marketplace, no extra setup needed)
| Plugin | What it does | Install |
|---|---|---|
| **frontend-design** (Anthropic) | The most-installed plugin. Stops generic "AI look" UIs | `/plugin install frontend-design@claude-plugins-official` |
| **superpowers** (obra) | 14 skills that enforce brainstorm → plan → test → code → debug. Fixes "Claude jumps straight into code" | `/plugin install superpowers@claude-plugins-official` |

### Strong second picks
| Skill set | What it does | Install |
|---|---|---|
| **Impeccable** (pbakaus) | Anti-slop design: 23 commands plus 27 automatic checks (cheesy gradients, cramped spacing, Inter everywhere, cards inside cards). Run `/teach-impeccable` once to save your brand context | `npx skills add pbakaus/impeccable` |
| **Vercel agent-skills** | React/Next.js best practices (70 performance rules), **web-design-guidelines** (audits 100+ accessibility/UX/performance rules), composition patterns, deploy-to-vercel | `npx skills add vercel-labs/agent-skills` |
| **Supabase agent-skills** | Postgres/Supabase rules: schema design, RLS security, query performance | `/plugin marketplace add supabase/agent-skills` → `/plugin install supabase@supabase-agent-skills` |
| **Anthropic skills** | Official document skills (pdf, docx, xlsx, pptx) plus example skills to learn from | `/plugin marketplace add anthropics/skills` |

> Pick **one** design skill (frontend-design *or* Impeccable). Two design skills with different taste give Claude conflicting instructions.

### Where to discover more
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code): the main curated list (~47k stars) of skills, hooks, commands, status lines and tools
- [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills): focused on skills only, well curated
- ComposioHQ/awesome-claude-skills: huge (1000+), quantity over quality
- [skills.sh](https://skills.sh): a searchable directory that works with the `npx skills` installer

> ⚠️ A skill is instructions Claude follows, and many can run scripts. Before installing anything from an unknown author, skim its `SKILL.md`. Avoid low-star repos that ask for broad permissions.

---

## 2. MCP servers (connections to other apps)

| MCP | Why | Notes |
|---|---|---|
| **Context7** | Feeds Claude **current** docs for Next.js, Tailwind, Supabase etc., so it stops using outdated APIs | The most useful one for a vibe coder |
| **Playwright** | Claude opens your app in a real browser, clicks through it and takes screenshots to **check its own work** | Closes the "looks done" gap |
| **Supabase** | Claude sees your actual DB schema, writes migrations and queries | Use it on a dev project, not production data |
| **GitHub** | Issues, PRs, CI | Or simply install the `gh` CLI. Claude uses it well and it costs less context |
| **Vercel** | Deployments, env vars, logs | |
| **Figma** | Design ↔ code | Already connected in this environment |
| **Sentry** | Production errors → Claude fixes them | Once you have real users |

Add one with `claude mcp add ...`, or use the plugin version where one exists. Only keep connected what you use (check with `/mcp` and `/context`).

---

## 3. Building AI apps (an app that has AI inside)

| Tool | What it does |
|---|---|
| **[Vercel AI SDK](https://ai-sdk.dev)** | The standard TypeScript library for calling Claude, GPT or Gemini, with streaming, tool calling and `useChat`. Switching providers is a one-line change |
| **[AI Elements](https://vercel.com/blog/introducing-ai-elements)** | 45+ ready React components for AI apps: chat threads, prompt input, reasoning panel, tool-call display, code blocks, artifacts. Built on shadcn, copied into your project so you can restyle them. `npx ai-elements@latest` |
| **Anthropic docs + cookbook** | Prompting, tool use, prompt caching, agents. In Claude Code, the built-in `/claude-api` skill gives Claude up-to-date API knowledge |
| **v0.dev** (Vercel) | Generate a UI from a prompt, then pull the code into your repo for Claude to refine |

**A good default AI-app stack:** Next.js + AI SDK + AI Elements + Supabase (auth + DB, plus pgvector if you need "chat with your documents") + Vercel.

---

## 4. UI components (free, shadcn ecosystem)

| Library | Best for |
|---|---|
| **[shadcn/ui](https://ui.shadcn.com)** | The base layer. Accessible primitives you own. **Customize the theme**, because the default look is recognizable |
| **Magic UI** | 150+ animated components: marquees, bento grids, text effects. Great for landing pages |
| **Aceternity UI** | Flashy marketing and motion effects. Use sparingly |
| **Origin UI** | Polished practical components: inputs, tables, forms |
| **[21st.dev](https://21st.dev)** | A community marketplace of shadcn-style components, plus an MCP that lets Claude pull components in |
| **Tailark / ReUI** | Ready marketing sections and blocks |

> Rule: one base (shadcn) + one accent library at most. Mixing three animated libraries is exactly what makes a site look AI-generated.

---

## 5. UX and design learning
Use these to judge what Claude makes.

| Resource | Why |
|---|---|
| **[Laws of UX](https://lawsofux.com)** | Short, visual psychology principles (Hick's law, Fitts's law…). Free, about an hour to read |
| **Refactoring UI** (Tailwind's creators) | The best design book for developers: spacing, hierarchy, color. Paid, but worth it. Their free tips are on X/blog |
| **[Mobbin](https://mobbin.com)** | Real screenshots from real apps (free tier). Find a flow you like and give Claude the screenshot as a reference |
| **"Don't Make Me Think"** (Steve Krug) | Short classic on usability and navigation |
| **Figma Community** | Free UI kits and design systems |
| **Google Stitch** | Generates UI concepts and exports a `DESIGN.md` for Claude |
| **Fonts:** Google Fonts, Fontshare | Free, distinctive fonts, an easy way to stop looking like Inter-everywhere |

---

## 6. Starter kit
1. `frontend-design` **or** Impeccable, plus `superpowers`
2. Context7 + Playwright MCP (+ Figma, already connected)
3. Vercel `web-design-guidelines` + Supabase skills when you start the backend
4. shadcn/ui + one accent library, plus Mobbin screenshots as references
5. AI SDK + AI Elements if the app has AI features

That's about 6 things. Use them for a project or two before adding more, since every extra plugin adds context noise.

---

## Sources
- [anthropics/skills (GitHub)](https://github.com/anthropics/skills)
- [Best Claude Code plugins 2026 (Claude Market)](https://claudemarket.ai/blog/best-claude-code-plugins-2026)
- [Superpowers plugin guide (Developers Digest)](https://www.developersdigest.tech/blog/claude-code-superpowers-plugin-guide)
- [Impeccable intro (Mintlify)](https://www.mintlify.com/pbakaus/impeccable/introduction)
- [Impeccable guide (Emelia)](https://emelia.io/hub/impeccable-ai-design-skill)
- [Vercel agent-skills guide (Skillselion)](https://skillselion.com/guides/vercel-labs-skills-claude-code-guide)
- [Supabase agent skills](https://supabase.com/ui/docs/ai-editors-rules/skills)
- [Awesome Claude Code resources 2026 (Fast.io)](https://fast.io/resources/awesome-claude-code-resources-2026/)
- [7 Best free Claude skill repos (Ryan Doser)](https://ryandoser.com/best-free-claude-skill-repos/)
- [Best MCP servers for Claude Code (Composio)](https://composio.dev/blog/best-mcp-servers-claude-code-codex)
- [Best MCP servers for Claude Code (DesignRevision)](https://designrevision.com/blog/best-mcp-servers-for-claude-code)
- [Introducing AI Elements (Vercel)](https://vercel.com/blog/introducing-ai-elements)
- [Vercel AI Elements (LogRocket)](https://blog.logrocket.com/vercel-ai-elements/)
- [shadcn block libraries (AdminLTE)](https://adminlte.io/blog/shadcn-ui-block-libraries/)
- [Magic UI alternatives (DesignRevision)](https://designrevision.com/alternatives/magic-ui)
- [Free UI/UX resources for developers (Indie Hackers)](https://www.indiehackers.com/post/best-free-ui-ux-resources-for-developers-in-2026-YyfFKXCLyWfyVcu0ueET)
