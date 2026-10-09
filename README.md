# Bonsai's test pack

A small, public **test fixture** for [Bonsai](https://github.com/LastStep/Bonsai)'s walking skeleton (Bonsai's
`design/plan.md`, part 4a) and its step 5 (`design/plan-5.md`, piece 5.1.1: consent to code). It is not for real use:
its role, skill, files, hook line and deny rule are made up.

A Bonsai pack is a Claude Code plugin plus a `bonsai/` folder that Bonsai's engine reads (Bonsai spec section 5).
Bonsai's checks fetch this pack, install it and update it between its commits, **A to F**, and depend on exactly
what changes between them. So the pack changes only by appended commits, never by rewriting one, and each changes only
what the table below says.

- **Source:** `https://github.com/LastStep/bonsai-test-pack`. It is public, so a fetch needs no login.
- **Pinned by commit, never by tag.** The pack has no tags and no releases. Find the commits by their subjects, which
  start `A:` to `F:`: `git log --format='%H %s'`.
- **No `version` in `.claude-plugin/plugin.json`, on purpose.** With one, Claude Code keeps a machine on its cached
  copy whatever the commit; without one, the version is the commit (Bonsai spec section 5). So
  `claude plugin validate --json .` gives exactly one warning, the missing `version`, and nothing else.

## Layout

| File | What it is |
|---|---|
| `.claude-plugin/plugin.json` | The plugin manifest. Its `name` is the pack's id, `test-pack`; no `version`. Documented below, since JSON holds no comments |
| `agents/marker.md` | The `marker` role (`test-pack:marker`): it says which commit a session loaded ("commit A" or "commit B") |
| `skills/hello/SKILL.md` | The `hello` skill (`test-pack:hello`): shows a session loaded the pack's skills; the same at every commit |
| `hooks/hooks.json` | From commit F: a hook of the plugin itself, which Claude Code runs on its own (not one the engine writes into a project). Documented below, since JSON holds no comments |
| `bonsai/pack.yaml` | The Bonsai manifest (`bonsai.pack/1`): the pack's id and version, what it needs, and everything the engine writes into a project (files, the block, the hook line, the deny rule) |
| `bonsai/block.md` | The pack's part of the instruction block the engine writes into a project's `CLAUDE.md` (kind `block`) |
| `bonsai/files/guide.md` | A pack file (kind `pack`) written to `test-pack/guide.md`; commits B, C and E change it |
| `bonsai/files/start.md` | A starter file (kind `once`) written to `test-pack/start.md`; the same at every commit |
| `bonsai/files/extra.md` | A pack file (kind `pack`) written to `test-pack/extra.md`; commit B adds it |
| `.gitattributes` | Every file is text with LF line endings on every checkout, Windows included |
| `README.md` | This page: what the pack is, what each commit changes, every file and field, and the choices made |

Every file documents itself: `pack.yaml` with a `#` comment on every key and a header listing every field; the role
and the skill with `#` comments in their YAML frontmatter (Claude Code reads frontmatter as YAML and skips comments); the
markdown files under `bonsai/` with an HTML comment at the top; `plugin.json` here.

## The commits

| Commit | What changes (and nothing else) | Who uses it |
|---|---|---|
| **A** | Everything above but `bonsai/files/extra.md`. The marker says "commit A"; `guide.md` is edition 1; one hook line, `echo test-pack hook A` | Part 2 (the reader reads `pack.yaml`); part 3 (`init`, checks 1-4 and 12); part 4b (`project-a`, check 8, hand check a) |
| **B** | `agents/marker.md` says "commit B"; `bonsai/files/guide.md` becomes edition 2; `bonsai/files/extra.md` is new, and `bonsai/pack.yaml` gains its one `files` entry | Check 5 (A to B: one file updated, one created, the rest unchanged; a new session loads B's role); check 6 (A to B, the first conflict); part 4b (`project-b`, `project-a-worktree`) |
| **C** | `bonsai/files/guide.md` becomes edition 3 | Check 6 (B to C: the same file changed again, a conflict again after `--keep`) |
| **D** | One line of `bonsai/pack.yaml`: the hook line's `command`, from `echo test-pack hook A` to `echo test-pack hook D` | Part 3's refusal of a hook-line change (exit 4, nothing written, `--allow-exec` named); part 5's verifier reads it; step 5.1.1 (C to D needs `--allow-exec`) |
| **E** | The mixed update: `bonsai/files/guide.md` becomes edition 4, and the hook line's `command` becomes `echo test-pack hook E` (its entry gains `runs: []`, the pack files it runs: none; `pack.yaml`'s header documents `runs`). This README says so | Step 5.1.1: D to E changes a pack file and the hook line together, so it is refused whole without `--allow-exec` (exit 4, nothing written) and written whole with it |
| **F** | Only `hooks/hooks.json`, a hook of the plugin itself: on `SessionStart` (matcher `startup`) it runs `echo test-pack plugin hook F`. This README says so | Step 5.1.1: E to F changes nothing the engine writes but the plugin wiring, yet the plugin now carries a code part, so the update needs `--allow-exec`; check 5's new session, at F, runs it |

How the checks use them:

- **Check 5** (Bonsai part 3; its last clause in part 4b): a project at A, `bonsai update` to B. In the project,
  `test-pack/guide.md` is updated, `test-pack/extra.md` is created, and `test-pack/start.md`, the `CLAUDE.md` block,
  the hook line and the deny rule are unchanged. A new session then shows the marker saying "commit B".
- **Check 6** (part 3): a project at A, a person edits `test-pack/guide.md`, then `update --yes` to B is a conflict
  (exit 5) naming that file; `--keep test-pack/guide.md` applies the rest; C changes the file again, so the update to
  C is a conflict again; `--adopt` takes C's copy.
- **The hook-line change** (part 3; step 5.1.1): update a project **from C to D**. In the pack only the hook line
  differs between them, so the update is a pure hook-line change: refused without `--allow-exec` (exit 4, nothing
  written), written with it.
- **The mixed update** (step 5.1.1): update a project **from D to E**. A pack file and the hook line change together:
  without `--allow-exec` nothing is written, not even the file (all or nothing: the lock records one commit per pack).
- **A hook of the plugin itself** (step 5.1.1): update a project **from E to F**. Only the plugin's own
  `hooks/hooks.json` is new; Bonsai counts a plugin's code parts (hooks, MCP and LSP servers, monitors, mods) as
  running code, so the update needs `--allow-exec`, and so does a first link to F.
- **Every first link to this pack needs `--allow-exec`** (step 5.1.1): every commit carries a hook line, which is a
  pack's code. Bonsai's own hook lines alone would be written on `--yes`.
- **Check 8 and hand check a** (part 4b): `project-a` at A, `project-b` at B, and `project-a-worktree` (a worktree of
  `project-a`) at B. `/agents` shows the marker with "commit A" or "commit B" in its description;
  `claude --agent test-pack:marker --bg` starts the role; asking for "hello from the test pack" shows the skill loaded.
- **Every move between commits also changes the plugin wiring** the engine writes into `.claude/settings.json` (the
  locked commit and the marketplace name made from it). That is the engine's own entry, not a change in this pack.
- **Part 5** (the guard): its protected list comes from the project's own `bonsai.yaml` (`project-guard` lists
  `protected.txt`); this pack adds nothing to it. Its hook line runs in those sessions too, and blocks nothing.
- **Part 6**: the sample `claude plugin eval` can aim at the `hello` skill, whose answer is one fixed line. The pack
  holds no eval cases of its own.

## What the engine writes into a project

From `bonsai/pack.yaml`. The file kinds are Bonsai's (contract section 14, spec section 6).

| In the project | Kind | From | Changes at |
|---|---|---|---|
| `test-pack/guide.md` | `pack` | `bonsai/files/guide.md` | B, C and E |
| `test-pack/start.md` | `once` | `bonsai/files/start.md` | never |
| `test-pack/extra.md` | `pack` | `bonsai/files/extra.md` | new in B |
| `CLAUDE.md`, inside Bonsai's marked region | `block` | `bonsai/block.md`, without its leading HTML comment | never |
| `.claude/settings.json`: one `SessionStart` hook line | `keys` | `hooks` in `pack.yaml` | D and E |
| `.claude/settings.json`: one deny rule, `Edit(test-pack/never.txt)` | `keys` | `deny` in `pack.yaml` | never |

The hook line, `echo test-pack hook A` on `SessionStart` (matcher `startup`; `D` at D, `E` at E), prints one line when a session starts,
which Claude Code adds to the session's context. It cannot block, calls no shell by name, and `echo` works in every
shell a hook may run in (sh, Git Bash, PowerShell). The deny rule names a file that need not exist.

## `.claude-plugin/plugin.json`, field by field

| Field | Meaning | Allowed values | Example |
|---|---|---|---|
| `name` | The plugin's name and the pack's id (equal to `id` in `bonsai/pack.yaml`). Claude Code loads the pack's roles and skills under it | lower-case letters, digits and dashes | `"test-pack"` |
| `description` | One sentence on what the plugin is, shown by Claude Code's plugin commands | text | `"A test fixture for Bonsai's walking skeleton: ..."` |
| `author` | Who makes the plugin: `name`, and a `url` | an object with `name` (text) and `url` (a link) | `{"name": "LastStep", "url": "https://github.com/LastStep"}` |
| `repository` | Where the plugin's source lives | a link | `"https://github.com/LastStep/bonsai-test-pack"` |
| `version` | **Never present** in a Bonsai pack (above): Bonsai pins the commit instead | absent | (none) |

## `hooks/hooks.json` (from commit F), field by field

Claude Code's own hooks file for a plugin (its plugin reference), not one of Bonsai's: Claude Code runs it from its
copy of the plugin, on its own, in every session where the plugin is on. Bonsai writes nothing from it into a project.

| Field | Meaning | Allowed values | Example |
|---|---|---|---|
| `hooks` | The event map, as in `.claude/settings.json`'s `hooks` | an object of event names | `{"SessionStart": [...]}` |
| `hooks.SessionStart[].matcher` | Which session starts the group runs on | `startup`, `resume`, `clear`, `compact` | `"startup"` |
| `hooks.SessionStart[].hooks[].type` | The kind of hook | `command` here | `"command"` |
| `hooks.SessionStart[].hooks[].command` | The shell line Claude Code runs; it calls no shell by name and blocks nothing | a shell line | `"echo test-pack plugin hook F"` |

## `bonsai/pack.yaml`, in short

Format-1 YAML (Bonsai contract section 2.4): `format:` first, keys `[a-z][a-z0-9_]*`, no anchors, aliases, tags or
flow mappings, and every text value that does not start with a letter quoted. Its own comments document every field;
in short:

| Field | Meaning | Here |
|---|---|---|
| `format` | The format and its version, always first | `bonsai.pack/1` |
| `id` | The pack's id, equal to `plugin.json`'s `name` | `test-pack` |
| `version` | The pack's version; a release tag must equal it | `"0.1.0"` at every commit |
| `needs.claude_code` | The oldest Claude Code the pack works with; an older one gets a `check` warning, never a refusal | `"2.1.0"` |
| `block` | The pack's part of `CLAUDE.md`'s block, a file in `bonsai/` (kind `block`), or null | `block.md` |
| `files` | Files written into a project: `path` (in the project), `from` (in `bonsai/files/`), `kind` (`pack` or `once`) | two at A; three from B |
| `hooks` | Hook lines for `.claude/settings.json` (kind `keys`): `event`, `matcher`, `command`, `runs` (the pack's own files the command runs, by their path in the project; `[]` for none, and a missing `runs` reads as `[]`), `why` | one; `runs: []` from E |
| `deny` | Deny rules for `.claude/settings.json` (kind `keys`): `rule`, `why` | one |

The role (`agents/marker.md`) uses Claude Code's `name`, `description`, `tools` (read-only: `Read, Grep, Glob`) and
`model` (`sonnet`); the skill (`skills/hello/SKILL.md`) uses `name` and `description`.

## Choices made where Bonsai's plan and spec are silent

Each is the smallest thing that serves the skeleton's checks.

1. **The pack's id is `test-pack`**, in the repository `bonsai-test-pack`, as the `workflow` pack lives in
   `bonsai-workflow`. Its role loads as `test-pack:marker`, its skill as `test-pack:hello`.
2. **The hook line is declared in `bonsai/pack.yaml`** (`hooks`, written by the engine as kind `keys`), not in a plugin
   `hooks/` folder: Bonsai's hooks never ride in a plugin (spec section 5), and `update` can only refuse a hook-line
   change it can see. **Commit F's `hooks/hooks.json` is the one exception, on purpose:** it tests that Bonsai counts
   a plugin's own code parts as running code (step 5.1.1). It runs only `echo` and blocks nothing. Each entry has `event`, `matcher`, `command` and a `why` sentence for the preview.
3. **The hook line is `echo test-pack hook A` on `SessionStart`** (matcher `startup`): it cannot block, so it is
   harmless in every check's sessions, and `echo` works in every shell a hook may run in (sh, Git Bash,
   PowerShell). **Commit D** changes it to `echo test-pack hook D`, one line, nothing else; **commit E** to
   `echo test-pack hook E`, beside a change of `guide.md` (the mixed update).
4. **The file that B and C change is `test-pack/guide.md`, kind `pack`**, so a person's edit to it is a conflict
   (check 6). **B's new file, `test-pack/extra.md`, is kind `pack`** as well, so `update` creates it.
5. **Every kind `init` writes is present once:** a `pack` file, a `once` file that never changes, the `block`, and two
   `keys` entries (the hook line and one deny rule with its `why`, the preview's sentence from the pack).
6. **`version` is `"0.1.0"` at every commit:** D may change only the hook line, and the pack has no tags. Bonsai
   tells the commits apart by commit only.
7. **`needs.claude_code` is `"2.1.0"`**, a made-up floor below the Claude Code versions the skeleton runs on, so
   its `check` gives no version warning. The pack was validated with Claude Code 2.1.294.
8. **`bonsai/block.md` keeps its documentation in a leading HTML comment**, and the engine writes only the text after
   it: the block in `CLAUDE.md` carries no docs (spec section 5).
9. **The marker names the commit that last changed it:** "commit A" at A; "commit B" at B and every later commit,
   since those change only what the plan says.
10. **One skill, `hello`, the same at every commit:** check 8 and the gate report ask whether a pack's roles *and
    skills* load in each kind of session.
11. **The role is read-only** (`Read, Grep, Glob`) and runs on `sonnet`, so a `--bg` run of it can change nothing.
12. **This README was written whole at A**, describing B, C and D, so those commits touch only the files the plan
    names. **E and later commits each update it in the same commit** (Bonsai's `design/plan-5.md`, "How the work runs
    in step 5", item 6). Not here: a licence, a CI workflow (the pack template's `pack.yml` comes with Bonsai's step
    5.5), labels, lanes and eval cases.
13. **E's hook entry says `runs: []`:** its `echo` runs no file of the pack. The field is new with Bonsai's step
    5.1.1; a missing `runs` reads the same, so A to D need no change.

## Checking the pack

- `claude plugin validate --json .` at each commit: one warning, the missing `version`; no error (F's hook
  included).
- `git ls-files --eol`: every file `i/lf w/lf`.
- `bonsai/pack.yaml` reads under Bonsai's format-1 rules and with a YAML 1.1 library alike.
