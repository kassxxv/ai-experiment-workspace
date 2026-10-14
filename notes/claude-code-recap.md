# Claude Code Recap: How to Build with AI Properly

> Researched October 2026 against the official Claude Code docs and current community sources (links at the bottom).

## Table of contents
1. [How Claude Code is put together](#part-1-how-claude-code-is-put-together)
2. [Working method that prevents spaghetti](#part-2-working-method-that-prevents-spaghetti)
3. [Git and branches with AI](#part-3-git-and-branches-with-ai)
4. [Context limits and long projects](#part-4-context-limits-and-long-projects)
5. [Design without AI slop](#part-5-design-without-ai-slop)
6. [Free stacks that cover most of the work](#part-6-free-stacks-that-cover-most-of-the-work)
7. [Claude + Gemini (and other tools)](#part-7-claude--gemini-and-other-tools)
8. [A starter workflow for a new app](#part-8-a-starter-workflow-for-a-new-app)

---

## Part 1: How Claude Code is put together

### The core idea
Claude Code is a loop: **model → tools (read, edit, bash, web) → results → model**, running inside a **context window**. That's everything Claude can "see" right now: your messages, the files it read, command output and loaded instructions.

**Context is the resource you're managing.** Quality drops as the window fills, and almost every best practice exists because of that.

### The extension layer: what each piece is

| Piece | What it is | When it loads | Is it enforced? | Use it for |
|---|---|---|---|---|
| **CLAUDE.md** | A markdown file of standing instructions | Every session, in full | No, it's advisory | "Use pnpm", build/test commands, project conventions |
| **`.claude/rules/*.md`** | CLAUDE.md split into pieces, optionally scoped by file path | At start, or when matching files are touched | No | "Rules for `*.tsx` files" |
| **Skill** (`SKILL.md`) | A packaged prompt plus optional scripts and reference files | Only the description at start. The full body loads when it's used | No | Reusable workflows (`/deploy`) and reference knowledge (API style guide) |
| **Subagent** (`.claude/agents/x.md`) | A separate Claude with its own context window and tool set | When spawned | n/a | Research or review that would flood your main context. Only a summary comes back |
| **MCP server** | A protocol that connects Claude to external systems (DB, Figma, browser, Supabase) | Tool names at start, full schemas when needed | n/a | Giving Claude **new abilities and data** |
| **Hook** | Your script that runs on a lifecycle event (`PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`…) | On the event | **Yes, it's deterministic** | "Always run lint after edits", "block edits to `.env`" |
| **Plugin** | A **package** that bundles skills, agents, hooks and MCP servers | When installed | n/a | Sharing a setup across repos and people (`/plugin`) |
| **Output style** | Changes Claude's tone and format for a session | Session start | No | "Explain more", "be concise" |

**The two rules to remember:**
1. **MCP gives Claude abilities. Skills give Claude know-how.** They work together: Supabase MCP connects to your DB, and a skill teaches your schema and query patterns.
2. **Instructions are requests. Hooks are guarantees.** If "never touch X" must always hold, make it a `PreToolUse` hook, not a line in CLAUDE.md.

### When to add which (Anthropic's own "triggers")

| Trigger | Add |
|---|---|
| Claude gets a convention or command wrong twice | A line in CLAUDE.md |
| You keep typing the same prompt to start a task | A user-invocable skill |
| You paste the same playbook into chat a third time | A skill |
| You keep copying data from a browser tab Claude can't see | An MCP server |
| A side task floods your conversation with output | A subagent |
| You want something to happen every time without asking | A hook |
| A second repository needs the same setup | Package it as a plugin |

### Skills, answered precisely
- **Is a skill just a prompt?** Mostly, yes. It's a markdown prompt with YAML frontmatter. It can also bundle scripts and reference files that Claude reads only if needed ("progressive disclosure"), and it can pre-run shell commands with `` !`git diff` ``.
- **Do I write my own instructions inline with it?** Yes, as arguments. `/fix-issue 1234 focus on the auth path`: everything after the name becomes `$ARGUMENTS`, and skills can also define `$0`, `$1` or named args. You can also just keep chatting normally after invoking it.
- **Whole conversation or one message?** **The whole conversation.** The docs say: *"the rendered SKILL.md content enters the conversation as a single message and stays there across later turns."* Two caveats:
  - **Tool permissions** granted by the skill (`allowed-tools`) **clear on your next message**.
  - The file isn't re-read, so write skills as standing instructions, not one-time steps. After `/clear` it's gone.
- **Auto vs manual:** Claude can trigger a skill when your task matches its `description`. `disable-model-invocation: true` makes it manual-only (`/name`), which is right for anything with side effects, like deploys.
- **`context: fork`** runs the skill in a fresh subagent that has **no** conversation history.

**Where skills live:**
- `~/.claude/skills/<name>/SKILL.md`: personal, all projects
- `.claude/skills/<name>/SKILL.md`: project, committed to git
- `<plugin>/skills/...`: called as `/plugin-name:skill-name`

**Minimal example:**
```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
Analyze and fix the GitHub issue: $ARGUMENTS.
1. Use `gh issue view` to get the details
2. Find the relevant files, implement the fix
3. Write and run tests, then commit
```

**Minimal subagent example** (`.claude/agents/security-reviewer.md`):
```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: Read, Grep, Glob, Bash
---
You are a senior security engineer. Review for injection, auth flaws,
secrets in code, insecure data handling. Give line references and fixes.
```

### The "*.md" zoo

| File | Who reads it | Purpose |
|---|---|---|
| `CLAUDE.md` | Claude Code | Project instructions. Keep it **under ~200 lines**. For each line, ask "would removing this cause a mistake?" If not, delete it |
| `CLAUDE.local.md` | Claude Code | Your private project notes (add it to .gitignore) |
| `~/.claude/CLAUDE.md` | Claude Code | Your personal rules for all projects |
| `AGENTS.md` | Codex, Cursor, Copilot, Windsurf, Zed… and **Claude Code now reads it too** | The cross-tool standard (Linux Foundation). **If you use several AI tools, put shared rules here** |
| `DESIGN.md` | Any agent | Google Stitch's open spec (Apr 2026): design tokens (colors, type, spacing) in YAML plus prose on *why*. It keeps UI consistent across sessions and tools |
| `SPEC.md` / `PLAN.md` | Convention, not magic | The feature spec Claude writes before building. This is how you survive context limits (Part 4) |
| `.claude/agents/*.md` | Claude Code | Subagent definitions (name, description, tools, model, system prompt) |
| `SKILL.md` | Claude Code | Skills, covered above |

**What belongs in CLAUDE.md:**

| ✅ Include | ❌ Exclude |
|---|---|
| Bash commands Claude can't guess | Anything Claude can figure out from the code |
| Code style rules that differ from defaults | Standard language conventions |
| Testing instructions, preferred test runner | Detailed API docs (link instead) |
| Branch naming, PR conventions | Info that changes often |
| Architecture decisions specific to the project | Long tutorials |
| Environment quirks (required env vars) | File-by-file descriptions |
| Non-obvious gotchas | "Write clean code" |

---

## Part 2: Working method that prevents spaghetti

From Anthropic's best-practices doc, condensed:

1. **Explore, then plan, then code, then commit.** Use plan mode (`Shift+Tab`) for anything touching several files. Skip it if you could describe the diff in one sentence.
2. **Have Claude interview you for big features.** Use this prompt:
   ```
   I want to build [X]. Interview me in detail using the AskUserQuestion tool.
   Ask about technical implementation, UI/UX, edge cases, concerns, and tradeoffs.
   Don't ask obvious questions, dig into the hard parts I might not have considered.
   Keep interviewing until we've covered everything, then write a complete spec to SPEC.md.
   ```
   Then **start a fresh session** to implement the spec.
3. **Always give Claude a way to verify its work.** Tests, a build, a linter, or a screenshot compared against a design. Without a check, "looks done" is the only signal Claude has, and that's how AI slop ships.
4. **Be specific.** Point to existing patterns (*"follow how `UserCard.tsx` does it"*), name files and constraints, and describe what "fixed" looks like.
5. **After two failed corrections, `/clear`** and write a better prompt. A polluted context only gets worse.
6. **Use a fresh-context review.** `/code-review`, or a subagent: *"review the diff against SPEC.md, report only correctness gaps"*. The reviewer isn't biased toward code it just wrote.
7. **Understand what got built.** Ask *"explain this flow like I'm onboarding: where does a request go, from button to DB?"* and *"why X instead of Y?"* If you can't explain your own app's data flow, stop adding features.

### Anti-spaghetti guardrails in practice
- Pick a mainstream, typed stack (TypeScript). Claude is best at what's popular.
- CLAUDE.md should name the architecture ("server code lives in `/lib/server`, never call the DB from components").
- Hooks should run typecheck and lint after edits, so errors come back to Claude automatically.
- Build **one feature at a time**, with a test for each, and commit when it's green.

### Common failure patterns
| Pattern | Fix |
|---|---|
| **Kitchen-sink session**: unrelated tasks mixed in one chat | `/clear` between unrelated tasks |
| **Correcting over and over** | After 2 failed corrections, `/clear` + a better prompt |
| **Over-specified CLAUDE.md**: rules get lost | Prune hard; convert must-haves into hooks |
| **Trust-then-verify gap**: plausible code, missed edge cases | Always provide tests, scripts or screenshots |
| **Infinite exploration**: "investigate X" reads 100s of files | Scope narrowly, or use a subagent |

---

## Part 3: Git and branches with AI

**Yes, use branches.** Claude's checkpoints (`Esc Esc` or `/rewind`) only track edits made through its file tools. Bash changes aren't captured, and the docs say *"This isn't a replacement for git."*

- **One feature = one branch.** For example: `git switch -c feat/login`, then plan, build and verify, then commit, then a PR or merge.
- **Commit small and often**, after each green step. These are your real save points.
- **Use worktrees for parallel work** (`claude --worktree` or `git worktree`). Two Claude sessions each get their own checkout, so they don't collide.
- Name sessions with `/rename feat-login` and resume with `claude --resume` or `claude --continue`. Treat sessions like branches.

---

## Part 4: Context limits and long projects

| Situation | Do this |
|---|---|
| Switching to an unrelated task | `/clear` |
| Long task, context filling | `/compact focus on the API changes and modified files` |
| Compact just part of the history | `Esc Esc` → pick a message → "Summarize from here" |
| Quick side question | `/btw ...`. The answer never enters your history |
| Heavy research | "Use a subagent to investigate X". Only a summary comes back |
| Check what's using space | `/context` |
| Undo Claude's edits / conversation | `Esc Esc` or `/rewind` |

You can also tell CLAUDE.md how to compact: *"When compacting, always preserve the list of modified files and test commands."*

### Files that make you context-proof
Knowledge that matters should **live in files, not in the chat**:
- `CLAUDE.md` / `AGENTS.md` for rules
- `SPEC.md` for what you're building
- `PLAN.md` or `PROGRESS.md` as a checklist that Claude updates as it goes ("✅ auth, ⏳ payments, next: webhook handler")
- `DESIGN.md` for the visual system
- Optionally `docs/decisions.md` for *why* you chose things

Then a brand-new session only needs: *"Read SPEC.md and PROGRESS.md, continue with the next item."*

---

## Part 5: Design without AI slop

**What "AI slop" UI looks like:** Inter or Roboto everywhere, purple-to-blue gradients, centered hero + 3 feature cards, emoji icons, glassmorphism on everything, every layout the same.

**What actually helps:**
1. **Anthropic's `frontend-design` skill** (free, official). It makes Claude commit to one aesthetic direction (editorial, brutalist, luxury, playful…) before writing code. It bans the generic defaults and pushes distinctive type, palettes and motion.
2. **Write a `DESIGN.md`** with your exact tokens (colors, fonts, spacing scale, radius) and the reasons behind them. Google ships a CLI that can export it to Tailwind.
3. **Give references, not adjectives.** Screenshots from Mobbin, Dribbble, or real sites you admire. "Make it look like this" beats "make it modern".
4. **Use a solid component base:** **shadcn/ui** + Tailwind. Then **customize the theme**. Unthemed shadcn is itself a recognizable look.
5. **Close the visual loop.** Have Claude screenshot the result (Playwright, or the Chrome extension) and compare it to the reference: *"list the differences and fix them."*
6. **Figma MCP.** Claude can read your designs into code and push code back into Figma.
7. **Google Stitch** to generate UI concepts, export DESIGN.md, then have Claude Code build them.

### Popular skills and plugins worth knowing
- **superpowers** (obra): enforces plan, then test, then code
- **frontend-design** (Anthropic): anti-slop UI
- **webapp / Playwright testing**: Claude verifies what it built
- **security-review** (built in): before shipping
- **skill-creator**: for writing your own skills
- Document skills: pdf, docx, xlsx, pptx

Rule of thumb: **install for the bottleneck you actually have, then prune.** Every model-invocable skill costs a little context.

---

## Part 6: Free stacks that cover most of the work

| Need | Pick | Free tier notes |
|---|---|---|
| **All-in-one backend** (DB, auth, storage, realtime) | **Supabase** | 2 projects, 500 MB DB, 1 GB files, 50k MAU. **Pauses after 7 days idle** (restorable). Has an official MCP |
| Reactive/realtime backend, TypeScript-native | **Convex** | Very AI-friendly: the schema and functions are just TS |
| Postgres only, branching | **Neon** | Scale-to-zero, many projects |
| Generous BaaS alternative | **Appwrite Cloud** | ~75k MAU |
| Self-hosted, zero cost | **PocketBase** | A single Go binary |
| Hosting (frontend) | **Vercel** / **Cloudflare Pages** / Netlify | Generous hobby tiers |
| Payments | Stripe / Polar / Lemon Squeezy | Free until you charge |
| Email | Resend | Free tier |

### Default stacks that AI handles well
- **Web app / SaaS:** Next.js + TypeScript + Tailwind + shadcn/ui + Supabase (or Convex) + Vercel
- **Simpler site or landing page:** Astro + Tailwind
- **Mobile:** Expo (React Native) + Supabase
- **Python/data/AI backend:** FastAPI + Postgres
- **Desktop:** Tauri or Electron

---

## Part 7: Claude + Gemini (and other tools)

**Important 2026 change:** Google **retired Gemini CLI for personal and free users on June 18, 2026**. Its replacement is **Antigravity CLI** (`agy`). Older guides that tell you to "pipe to the gemini CLI" are outdated.

Ways to combine them:
1. **A shared `AGENTS.md`**, the simplest. Both tools read the same project rules, so each stays consistent.
2. **Writer/reviewer across models.** Claude implements; the other model reviews the diff in a fresh context, or the reverse. Different models catch different bugs.
3. **Through MCP.** Community MCP servers (e.g. `gemini-mcp-tool`) let Claude call Gemini as a tool, typically for huge-context reads like a whole repo or long docs. Check that any server has been updated for the Antigravity switch.
4. **Split by strength.** Use one for UI concepts (Stitch, or Gemini for image-heavy work) and Claude Code for implementation and refactoring.

Keep **one source of truth**: the repo files (spec, progress, AGENTS.md). Never rely on one chat's memory when switching tools.

---

## Part 8: A starter workflow for a new app

1. `git init`, run `/init` to get a starter CLAUDE.md, then prune it and add an `AGENTS.md` if you use other tools.
2. Interview prompt → `SPEC.md`. Write `DESIGN.md` with references.
3. Install only `frontend-design`, plus a testing skill if needed. Connect the Supabase MCP.
4. Add a hook that runs typecheck and lint after edits.
5. For each feature: branch, plan mode, implement with tests, `/code-review`, commit, update PROGRESS.md.
6. `/clear` between features, and start each session with "read SPEC + PROGRESS".
7. Every few features, ask *"explain the architecture and data flow"* so you still understand it.

---

## Sources
- [Claude Code: Skills](https://code.claude.com/docs/en/skills)
- [Claude Code: Extend Claude Code (features overview)](https://code.claude.com/docs/en/features-overview)
- [Claude Code: Best practices](https://code.claude.com/docs/en/best-practices)
- [Claude Code: Memory / CLAUDE.md / AGENTS.md](https://code.claude.com/docs/en/memory)
- [Steering Claude Code (Anthropic blog)](https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more)
- [AGENTS.md vs CLAUDE.md (eesel)](https://www.eesel.ai/blog/claude-code-agents-md)
- [CLAUDE.md vs AGENTS.md vs .cursorrules (Unblocked)](https://getunblocked.com/blog/claude-md-vs-agents-md-vs-cursor-rules/)
- [Google Stitch open-sources DESIGN.md](https://the-decoder.com/googles-open-source-design-md-gives-ai-agents-a-prompt-ready-blueprint-for-brand-consistent-design/)
- [What is DESIGN.md (MindStudio)](https://www.mindstudio.ai/blog/what-is-design-md-google-stitch)
- [The Anti AI-Slop UI Design Skill](https://medium.com/@porter.nicholas/anthropic-skills-marketplace-the-anti-ai-slop-ui-design-skill-a572d0cfef4f)
- [Best Claude Code skills 2026 (Tella)](https://www.tella.com/blog/best-claude-code-skills)
- [Supabase free tier limits 2026](https://automationatlas.io/answers/supabase-free-tier-limits-2026/)
- [Database free tier comparison 2026](https://agentdeals.dev/database-free-tier-comparison-2026)
- [Best backend platforms for indie hackers](https://www.mindstudio.ai/blog/best-backend-platforms-indie-hackers)
- [Google: Transitioning Gemini CLI to Antigravity CLI](https://developers.googleblog.com/an-important-update-transitioning-gemini-cli-to-antigravity-cli/)
- [gemini-mcp-tool](https://mcpservers.org/id/servers/jamubc/gemini-mcp-tool)
