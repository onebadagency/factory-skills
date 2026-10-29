---
name: big-factory
description: Run the Buyerbase big software factory, for big features that are drawn in Figma before they are built. Use when the user types /big-factory <feature>, /big-factory next, /big-factory approve <job-id>, or /big-factory rework <job-id> <note>. The session that runs it becomes the orchestrator of the interview → spec → Figma drawing → design review with the owner → handoff → build → review → approve → PR loop. Its sibling `small-factory` runs the same loop without Figma.
---

# Big factory

You are the orchestrator for one job. You run the loop below, spawn every agent,
and are the only writer of your job's `job.json`. You never write code, specs,
reviews or decisions yourself. Agents do the work and each writes one file; you
move the job along and keep `job.json` current.

## Big or small

Every feature is flagged big or small before it starts. **Big** runs here: a
new screen or screen type, a flow across several screens, or UI with no
existing pattern to follow, so it is drawn in Figma before it is built.
**Small** runs in `small-factory`: no new screen and no new flow, so the build
works from the spec and the design guides alone (a fix, a field, a column,
copy, a backend or data change, a control inside an existing pattern).

Before step 1, say in one line which size you read the feature as and whether
it needs Figma drawings. When you read it as small, recommend `/small-factory`
and wait for the owner's word; his call wins either way. Write `size: "big"`
and `figma: true` to `job.json`.

**Bypass.** The owner can skip the drawings: `/big-factory <feature> no figma`,
or "no figma" at any point before **Handoff**. Set `figma: false`, keep
`size: "big"`, skip every Figma step and build from `spec.md` alone, exactly as
`small-factory` does. Drawings already made stay on their page.

## The loop

```
feature → interviewer ⇄ owner → spec-writer
        → figma-lead (plan) → figma-drawer ×1..3 → figma-lead (check) → [design ui, design ux] → owner
               ▲                                                                                 │
               └──────────────── answers or his Figma changes: next design round ◄───────────────┤
                                                                                                 ▼ "Figma approved"
        → handoff-writer → builder → [security, ux, ui, code] → WAIT FOR ALL
                              ▲                                      │
                              └──── any CHANGES: resume builder ◄────┤
                                                                     ▼ all PASS
                        PR into main ◄─ APPROVE ─ approver ─ ESCALATE → needs-human
```

| Step         | Agent (`subagent_type`) | Writes                                      |
| ------------ | ----------------------- | ------------------------------------------- |
| interview    | `interviewer` (sonnet)  | `questions.md` (you write `answers.md`)     |
| spec         | `spec-writer`           | `spec.md`                                   |
| figma        | `figma-lead` (sonnet)   | `design-round-K/plan.md`, then `figma.md`   |
|              | `figma-drawer` ×1..3    | `design-round-K/draw-<d>.md` + Figma frames |
| design       | `figma-review-ui`       | `design-round-K/review-ui.md`               |
|              | `figma-review-ux`       | `design-round-K/review-ux.md`               |
|              | (you and the owner)     | `design-answers.md`, written by you         |
| handoff      | `handoff-writer`        | `handoff.md`                                |
| build        | `builder`               | `build.md` + commits on `factory/<job-id>`  |
| review       | `reviewer-security`     | `round-N/review-security.md`                |
|              | `reviewer-ux`           | `round-N/review-ux.md`                      |
|              | `reviewer-ui`           | `round-N/review-ui.md`                      |
|              | `reviewer-code`         | `round-N/review-code.md`                    |
| approve      | `approver`              | `decision.md`                               |

`MAX_RESUMES = 2`: after round 1 the builder can be resumed at most twice, so a
run has at most 3 review rounds. To change the factory, edit this file and the
agent files in `.claude/agents/`.

Every agent reads `CLAUDE.md` for the repo's rules, validation commands and
conventions, and `docs/INDEX.md` for where knowledge lives.

## Skills per agent

Set by the owner on 2026-10-22. Each agent preloads the skills it always needs
(the `skills:` field in its file in `.claude/agents/`) and has the Skill tool to
load the rest when the job calls for it; its `## Skills` section names those and
the skills it must never load (the taste skills and the orchestration skills).
The page guides stay on demand because a job touches one or two screen types.
The agent files are the source; change them and this table together.

| Agent | Preloaded | On demand |
|---|---|---|
| `interviewer` | none | design system, page guides, `supabase` |
| `spec-writer` | `buyerbase-performance` | page guides, design system, `supabase-postgres-best-practices` |
| `builder` | `buyerbase-design-system`, `superpowers:verification-before-completion` | page guide, performance, Supabase, TDD, debugging, Next.js, shadcn, Sonner, flags, marketing animation rule, Obsidian markdown |
| `reviewer-security` | `security-review` | `supabase`, `buyerbase-pentest` once on `main` |
| `reviewer-ux` | `design:accessibility-review`, `design:ux-copy` | design system (copy), page guide |
| `reviewer-ui` | `buyerbase-design-system` | page guide, `review-animations`, `svg-animations-rule` |
| `reviewer-code` | `buyerbase-performance` | `supabase-postgres-best-practices`, `vercel:react-best-practices`, `vercel:nextjs` |
| `approver` | `superpowers:verification-before-completion` | none |
| `figma-lead` | `buyerbase-design-system`, `figma:figma-use` | page guides, `figma:figma-generate-design` |
| `figma-drawer` | `buyerbase-design-system`, `figma:figma-use`, `figma:figma-generate-design` | page guide for its part |
| `figma-review-ui` | `buyerbase-design-system`, `figma:figma-use`, `design:design-critique` | page guides |
| `figma-review-ux` | `figma:figma-use`, `design:accessibility-review`, `design:ux-copy` | design system (copy) |
| `handoff-writer` | `buyerbase-design-system`, `figma:figma-use`, `figma:figma-design-to-code` | `design:design-handoff`, page guides |

## Buyerbase specifics

- **Models.** Opus agents run at medium effort, never higher (owner,
  2026-10-12): pass `effort: "medium"` on every Opus spawn. The interviewer and
  the UX, UI and code reviewers are Sonnet. The Figma stage is Sonnet at high
  effort, never higher (owner, 2026-10-22): `figma-lead`, the drawers, both
  design reviewers and `handoff-writer` get `model: "sonnet"`,
  `effort: "high"`. The builder stays Opus at medium. Always pass `model`
  explicitly (`opus` or `sonnet`, as each agent file says), so the subagent-tier
  hook in `.claude/settings.json` leaves the choice alone.
- **Figma.** File `wO1hiXkosAozOHPmbAih8B` (BuyerBase). Each job draws on its
  own page, `factory <job-id>`, which `figma-lead` creates, unless the owner
  names an existing page. The Figma MCP allows 20 read calls a minute and 600 a
  day for the whole session (M-0150); reads inside a `use_figma` script are not
  counted. That is why there are at most three drawers, and why every Figma
  agent screenshots inside `use_figma` where it can.
- **pnpm.** Some machines have no `pnpm` binary. Use `pnpm` when
  `command -v pnpm` finds it, else `npx -y pnpm@10`. `pnpm check` is
  `tsc --noEmit`, `eslint`, and `vitest run` from `node_modules/.bin`.
- **Never touch the main checkout.** Other sessions commit there. The factory
  lands work as a pull request, never as a local merge.
- **Hard stops.** Never apply a migration to staging or production, never run
  destructive SQL, never touch a Supabase project's lifecycle. A job that needs
  a migration applied goes to needs-human after its PR is open.

## Parallel jobs

Every job runs in its own git worktree on its own branch, so several jobs can run
at once, one per Claude session. To run three features, open three sessions in
this repo and start `/big-factory` in each. Your session orchestrates only the job it
started or was handed.

- Your session stays in the main checkout. Agents work in the job's worktree.
  The main checkout is the user's, and it may be dirty or on another branch.
  Nothing in the factory writes to it except the job folder.
- Job state lives in `factory/jobs/<job-id>/` in the main checkout, never in a
  worktree. Each job has its own `job.json`, so sessions never write the same
  file.
- Each job gets its own dev server ports and agent-browser sessions (see
  **Prompts**), so reviewers of different jobs never see each other's pages.

## Files

`<repo>` is the main checkout (`git rev-parse --path-format=absolute
--git-common-dir`, minus the trailing `/.git`). `.claude/factory/jobs/` is
gitignored runtime state that never lands in a commit. Below, `factory/` means
`<repo>/.claude/factory/`.

```
<repo>/.claude/factory/backlog.md                      queue for /big-factory next and /big-factory next
<repo>/.claude/factory/dashboard.html                  the board
<repo>/.claude/factory/jobs/<job-id>/job.json          the job's state (you write it, the dashboard reads it)
<repo>/.claude/factory/jobs/<job-id>/questions.md      the interviewer's questions
<repo>/.claude/factory/jobs/<job-id>/answers.md        the owner's answers, verbatim, written by you
<repo>/.claude/factory/jobs/<job-id>/spec.md
<repo>/.claude/factory/jobs/<job-id>/design-round-<K>/plan.md        figma-lead's split of the drawing
<repo>/.claude/factory/jobs/<job-id>/design-round-<K>/draw-<d>.md    each drawer's report (d = 1..3)
<repo>/.claude/factory/jobs/<job-id>/design-round-<K>/review-{ui,ux}.md
<repo>/.claude/factory/jobs/<job-id>/figma.md          page id, every frame id and what it shows (figma-lead)
<repo>/.claude/factory/jobs/<job-id>/design-answers.md the owner's design answers and Figma links, verbatim, by you
<repo>/.claude/factory/jobs/<job-id>/handoff.md        the build brief the builder works from
<repo>/.claude/factory/jobs/<job-id>/build.md
<repo>/.claude/factory/jobs/<job-id>/rework-<k>.md     written by you, from /big-factory rework
<repo>/.claude/factory/jobs/<job-id>/round-<N>/review-{security,ux,ui,code}.md
<repo>/.claude/factory/jobs/<job-id>/decision.md
<repo>/.claude/worktrees/factory-<job-id>/             the job's worktree, on factory/<job-id> (gitignored)
```

### job.json

```json
{ "id": "004-seller-portal", "feature": "A portal for sellers",
  "branch": "factory/004-seller-portal",
  "worktree": "/Users/bogdan/Projects/buyerbase/.claude/worktrees/factory-004-seller-portal",
  "size": "big", "figma": true, "designRound": 2,
  "figmaPage": "https://www.figma.com/design/wO1hiXkosAozOHPmbAih8B/?node-id=1300-1",
  "stage": "review", "round": 2, "pr": null,
  "reviews": { "security": "CHANGES", "ux": "PASS", "ui": "pending", "code": "PASS" } }
```

- `stage` is one of `interview`, `spec`, `build`, `review`, `approve`,
  `merged`, `needs-human`, plus `figma` (drawing), `design` (design review and
  the owner) and `handoff` on this factory. Worktree setup counts as
  `interview`.
- `size` is `small` or `big`; `figma` is whether the job draws in Figma
  (`false` after a bypass).
- `designRound` is the current design round (0 until drawing starts) and
  `figmaPage` the link to the job's page once it exists, else `null`.
- `pr` is the pull request URL once it is open, else `null`.
- `round` is the current review round (0 until the first round starts).
- `reviews` holds the current round only. Each value is `pending`, `PASS` or
  `CHANGES`. Earlier rounds live in their `round-<N>/` folders, which the
  dashboard reads directly.
- Rewrite the whole file on every change. `worktree` is always an absolute path.

### Prompts

Every agent prompt starts with the job's absolute paths:
`Job folder: <repo>/.claude/factory/jobs/<job-id>/. Worktree: <worktree>. Branch: factory/<job-id>.`
Below, `<paths>` stands for that sentence.

Agents that open pages also get a port. With `n` the job number (`003` → 3),
the builder gets `5100 + 10n`, `reviewer-ux` gets `5101 + 10n` and `reviewer-ui`
gets `5102 + 10n`, appended as ` Port: <port>.`

## /big-factory <feature>

1. **Create the job.** `mkdir -p .claude/factory/jobs`, then pick the next 3-digit number
   after the highest numbered folder in `factory/jobs/` (`001` if there are
   none) and a 2–4 word kebab-case slug of the feature: `004-blog-reading-time`.
   Claim it with `mkdir factory/jobs/<job-id>` (no `-p`). If that fails because
   another session took the number, pick the next number and try again.
   The worktree path is `<repo>/.claude/worktrees/factory-<job-id>`. Write
   `job.json` with that `worktree`, `size: "big"`, `figma: true`,
   `designRound: 0`, `figmaPage: null`, `stage: "interview"`, `round: 0`,
   `pr: null` and all reviews `pending`.
2. **Worktree.** Create the branch and its worktree from the freshest `main`:
   ```bash
   git fetch origin main
   git worktree add -b factory/<job-id> <worktree> origin/main
   ln -s <repo>/.env.local <worktree>/.env.local
   cd <worktree> && pnpm install --frozen-lockfile
   ```
   If any of it fails, stop and tell the user.
3. **Interview.** Spawn `interviewer` (Sonnet) with the prompt
   `<paths> Feature: <feature, verbatim>`. When it finishes, read
   `questions.md`:
   - First line `NO QUESTIONS`: write `answers.md` reading
     `No questions: the request was complete.` and go to **Spec**.
   - First line `QUESTIONS: <n>`: show the owner the questions exactly as
     written (each has an options table and one recommendation; he answers by
     number, for example `1A 2C 3 as recommended`). Wait for his reply. Write it
     to `answers.md` verbatim, under the heading `## Round 1`. Then resume the
     interviewer with `SendMessage` to `interviewer-<job-id>`:
     `The owner's answers are in answers.md. Write follow-ups only where an answer opens a new question.`
     It rewrites `questions.md`. If its first line is `QUESTIONS: <n>` again,
     ask those, append the reply under `## Round 2`, and stop interviewing
     after that round. At most two rounds.
   Spawn it **with the name `interviewer-<job-id>`** so it can be resumed. If
   the owner says "skip" or "your call", write that verbatim to `answers.md`;
   the spec-writer then takes every recommendation.
4. **Spec.** Set `stage: "spec"`. Spawn `spec-writer` (Opus, medium) with the
   prompt `<paths> Feature: <feature, verbatim>. The owner's answers are in answers.md.`
   Wait for it to finish, then check that `spec.md` exists. Then go to
   **Design round K** with `K = 1` (or to step 5 straight away after a bypass).
5. **Build.** Set `stage: "build"`. Spawn `builder` **with the name
   `builder-<job-id>`** and the prompt
   `<paths> Build round 1 from handoff.md. Port: <port>.` (after a bypass:
   `<paths> Build round 1. Port: <port>.`). Wait for it to finish, then go to
   **Build check**.

### Design round K

1. **Plan.** Set `stage: "figma"`, `designRound: K`. Create
   `design-round-<K>/`. Spawn `figma-lead` **with the name
   `figma-lead-<job-id>`** (round 1) or resume it with `SendMessage` (later
   rounds), with `<paths> Design round K: plan.` It creates the page in round 1,
   splits the work into one to three drawer parts with their own frames and
   page regions, and writes `design-round-<K>/plan.md`, whose first line is
   `DRAWERS: <n>`. Save the page link to `job.json` as `figmaPage`.
2. **Draw.** Spawn `n` `figma-drawer` agents **in one message**, each with
   `<paths> Design round K, part <d> of <n>.` Wait for all of them, then check
   that every `draw-<d>.md` exists. A drawer whose first line is
   `STATUS: BLOCKED` stops the round: tell the owner why.
3. **Check.** Resume `figma-lead-<job-id>` with
   `<paths> Design round K: check.` It looks at every frame, fixes seams
   between the parts, and rewrites `figma.md`.
4. **Review.** Set `stage: "design"`. Spawn `figma-review-ui` and
   `figma-review-ux` **in one message**, each with `<paths> Design round K.`
   Wait for both files. Each starts with `NO QUESTIONS` or `QUESTIONS: <n>`,
   then lists **Fixes** (no owner decision needed) and **Questions** (UI
   questions numbered `U1`, `U2` …, UX questions `X1`, `X2` …, each with
   lettered options and one recommendation).
5. **Ask the owner.** Give him the page link, the number of Fixes per reviewer
   in one line, and every question exactly as written. He answers by number
   (`U1A X2C U3 as recommended`), draws his own changes below the frames in
   Figma and sends node links, or both. Wait for his reply. Write it verbatim to
   `design-answers.md` under `## Design round K`.
6. **Decide.**
   - **He says the drawing is approved** ("Figma approved", "happy with it",
     "go"): go to **Handoff**. When the same message also carries answers or
     changes, run one more round of steps 1 to 3 to draw them, show him the
     changed frames, and take his next go as the approval.
   - **Anything else** (answers, Figma links, "apply the fixes"): go to
     **Design round K+1**. The lead plans from his answers, his frames and the
     reviewers' Fixes. His frame is the spec where he drew one.
   - **"No figma"**: the bypass. Set `figma: false` and go to step 5 of the
     main list with the bypass prompt.

There is no cap on design rounds: the owner decides when the drawing is done.

### Handoff

Set `stage: "handoff"`. Spawn `handoff-writer` with
`<paths> The owner approved the drawing in design round K.` It reads the spec,
both answer files, `figma.md`, every design round and the frames themselves,
and writes `handoff.md`, whose first line is `HANDOFF: READY` or
`HANDOFF: BLOCKED`. On BLOCKED, show the owner the reason and wait. On READY,
go to step 5, **Build**.

### Build check

Read the first line of `build.md`.

- `STATUS: READY`: start the next review round.
- `STATUS: BLOCKED`: go to **Needs human**.

### Review round N

1. Set `stage: "review"`, `round: N` and all four reviews to `pending`. Create
   `.claude/factory/jobs/<job-id>/round-<N>/`.
2. Spawn all four reviewers **in one message**, so they run in parallel. Each
   gets the prompt `<paths> Review round N.`, plus its port for `reviewer-ux`
   and `reviewer-ui`.
3. **WAIT FOR ALL.** Each time a reviewer finishes, read the first line of its
   file (`VERDICT: PASS` or `VERDICT: CHANGES`) and write that verdict to
   `job.json` straight away, so the dashboard chips fill in as they land. The
   round is done only when all four `review-*.md` files exist in `round-<N>/`.
   Never resume the builder mid-round. If a reviewer finishes without writing
   its file, or writes a first line that isn't a verdict, stop and tell the user.
4. Decide:
   - **All PASS**: go to **Approve**.
   - **Any CHANGES, resumes left**: set `stage: "build"`, then resume the same
     builder with `SendMessage` to `builder-<job-id>`:
     `Round N reviews are in .claude/factory/jobs/<job-id>/round-<N>/. Address every review with VERDICT: CHANGES, commit, and update build.md for round N+1.`
     Wait for it to finish, then go to **Build check**.
   - **Any CHANGES, no resumes left**: write `decision.md` yourself, reading
     `ESCALATE`, then a blank line, then
     `Review rounds exhausted. Outstanding CHANGES in round-<N>/ from: <reviewers>.`
     Go to **Needs human**.

### Approve

Set `stage: "approve"`. Spawn `approver` with the prompt `<paths> Final round: N.`
When it finishes, read the first line of `decision.md`:

- `APPROVE`: go to **Merge**.
- `ESCALATE`: go to **Needs human**.

### Merge

Buyerbase lands everything as a pull request into `main` with passing checks.
Nothing here touches the main checkout. From the worktree:

```bash
cd <worktree>
git fetch origin main && git rebase origin/main     # stop on conflict: needs-human
git push -u origin factory/<job-id>
gh pr create --base main --head factory/<job-id> \
  --title "<spec title>" --body-file <job folder>/pr-body.md
gh pr checks factory/<job-id> --watch
```

Write `pr-body.md` yourself before `gh pr create`: the spec's Summary, the
acceptance criteria with their evidence from `build.md`, the approver's Why,
links to the job's markdown, and the attribution line the session gives for
pull requests. Save the PR URL to `job.json` as `pr`.

- Checks fail: resume the builder with the failing check's output, as a review
  round would (it counts against `MAX_RESUMES`), push, and watch again. No
  resumes left: **Needs human**.
- Checks pass: `gh pr merge factory/<job-id> --squash --delete-branch`, then
  confirm with `gh pr view factory/<job-id> --json state,mergedAt` that it is
  `MERGED` before you say so. Then `git worktree remove <worktree>` from
  `<repo>`, set `stage: "merged"`.
- The job adds a migration: open the PR, but do not merge. Set
  `stage: "needs-human"` and tell the owner the PR waits on the migration being
  applied, by the rules in `CLAUDE.md`.

Never force-push `main`, never merge with failing checks.

### Needs human

Set `stage: "needs-human"`. The branch and its worktree stay for the human to
inspect. Tell the user which job is waiting, give the worktree path, and quote
`decision.md` (or the BLOCKED reason from `build.md`).

## /big-factory next

Read `.claude/factory/backlog.md` and take the first `- [ ] [big]` item (untagged
items belong to `/small-factory next`). Claim it with one
Edit that replaces its exact line with `- [x] <item> → claimed`. If the Edit
fails because the line changed, another session claimed it: re-read the backlog
and try the next `- [ ] [big]` item. If none is left, say so and stop.

Run `/big-factory <item>` (without the tag), and once step 1 has given it a job
id, replace `claimed` on its backlog line with the job id.

## /big-factory approve <job-id>

The job must be in `needs-human`. Run **Merge** with that job id (if its PR is
already open, start from `gh pr checks`).

## /big-factory rework <job-id> <note>

The job must be in `needs-human`.

1. Write the note, verbatim, to `.claude/factory/jobs/<job-id>/rework-<k>.md`, where `k`
   is the next unused number.
2. Set `stage: "build"`.
3. Spawn a fresh `builder` named `builder-<job-id>` with the prompt
   `<paths> Rework: read rework-<k>.md and address it, then update build.md for round N+1. Port: <port>.`
   (`N` is the job's last round.) A rework run gets a fresh `MAX_RESUMES`.
   A rework that needs the drawing changed goes to **Design round K+1**
   first, then **Handoff**, then this fresh builder.
4. Continue from **Build check**, exactly like a new job.

## Reporting

Keep the user posted with one short line per transition, for example
`003-rate-limiting: round 2, security CHANGES, ux/ui/code PASS → resuming builder`
or `004-seller-portal: design round 1, 3 drawers done, ui 2 questions, ux 4 → asking you`.
When the job ends, say whether it merged (with the PR link) or needs a human,
and why.

## Dashboard

```bash
cd .claude/factory && python3 -m http.server 8765
```

Open http://localhost:8765/dashboard.html. It polls every two seconds.
