# nk-breakable-selftest

An agent skill for [Claude Code](https://code.claude.com) and [OpenAI Codex](https://developers.openai.com/codex). Make a checker, validator, linter, gate or test suite prove it can fail.

**What you get.** One real run of nk-breakable-selftest 0.1.4, copied from the terminal on 2026-10-01:

```text
$ python3 scripts/breakcheck.py --demo
demo: a 12-line checker with rule A and rule B; its self-test has a sample for rule A only
$ python3 checker.py --selftest   →  selftest ok (exit 0)
$ breakcheck.py --root . --cmd "python3 checker.py --selftest" --auto checker.py --pattern '\.append\(\('
  ✔ control (no mutation)            green  ok
  ✔ L5: out.append(("A", "rule A"))  red    CAUGHT
  ✘ L7: out.append(("B", "rule B"))  green  UNCOVERED
✘ 2 mutations, 1 problem — UNCOVERED = decorative check or missing sample; CRASH = mutation invalid, not a detection
```

![nk-breakable-selftest](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/social/nk-breakable-selftest.png)

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## Try it

Nothing is installed and nothing under `~/.claude` changes: clone, run the self-test, run the example (it only writes inside the clone).

```bash
git clone https://github.com/NickkkLian/nk-breakable-selftest && cd nk-breakable-selftest
python3 scripts/breakcheck.py --selftest
python3 scripts/breakcheck.py --demo
```

The self-test prints:

```text
breakcheck selftest · 23/23 passed
```

The last command prints the block at the top of this page; its last line is the one below, and its exit code is 1 (non-zero on purpose: it found something).

```text
✘ 2 mutations, 1 problem — UNCOVERED = decorative check or missing sample; CRASH = mutation invalid, not a detection
```

![nk-breakable-selftest demo: before and after](https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/nk-breakable-selftest.gif)

## What it does

- Run a **break matrix**: mutate the guarded code one line at a time in a sandbox; the self-test must go red on the named assertion, never via a crash, and the unmutated control must stay green.
- Find **decorative checks**: lines whose removal leaves the self-test green.
- `--demo` runs the matrix on a bundled 12-line checker, so you can see a CAUGHT and an UNCOVERED in two seconds.
- One break run, on 2026-09-30, of the author's ten published checking tools (this one included; thirteen scripts, default settings): in eight scripts at least one switched-off line went unnoticed or crashed, and in four of them it was the line that sets the exit code (`publish_gate.py`, `evidence.py`, `indent_guard.py`, `rules_check.py`). All four have a sample now. The run after those fixes found a fifth exit-code line, in nk-handoff-package's `handback_check.py`; it is still open and named in that README. Both raw outputs ship in [nk-deck](https://github.com/NickkkLian/nk-deck)'s `references/example-data/`, and each tool's Verify section says what its own break run shows, line numbers included.
- A hand-written `--spec` on nk-git-guardrail-hook found that the two rules that read a repository had no sample at all.
- Ten design rules for detectors and guardrails (`references/design-rules.md`); the incidents behind four of them (rules 1, 2, 3 and 8) are written up in `references/incidents.md`.

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## Next to mutmut and cosmic-ray

This is mutation testing, and the Python tools people know for it are [mutmut](https://github.com/boxed/mutmut) and
[cosmic-ray](https://github.com/sixty-north/cosmic-ray) (Stryker does it for JavaScript, C# and Scala). They change a whole codebase
with many operators (a flipped comparison, a changed constant, a swapped operator), run your test suite against every mutant and
report which mutants survived. If you have a package and a test suite, use one of them: they find far more than this does.
`breakcheck.py` is for the smaller case they are not built around, a single script whose only test is its own `--selftest`.
It is one file with no install and no test runner. It has one operator: it switches off the lines that report (a finding recorded,
a failing exit code returned), one at a time. It keeps a crash apart from a catch: a mutant that ends in a traceback is listed as
`CRASH`, not as caught. With `--spec` it also checks that the red names the check that was broken. It says nothing about the logic
between the reporting lines. mutmut and cosmic-ray were read about (their READMEs, 2026-10-01), not installed or run here, so no
head-to-head result is claimed.

## How it works

1. Same code path. The self-test must call the production function(s).
2. One sample per rule, exclusive. Every rule gets a sample that trips *only* that rule.
3. A clean control. One sample that must produce zero findings.
4. Run the break matrix. `python3 scripts/breakcheck.py --root <dir> --cmd "python3 checker.py --selftest" --auto checker.py`.
5. Read the verdicts. `CAUGHT` is the only good one.
6. Pick break points that can actually change behaviour.

With `--spec`, `must_mention` counts only on failure lines: the first non-space character is `✘`, `✗`
or `×`, or the first word is FAIL, FAILED, FAILURE or ERROR (any case, optionally followed by `:`).
A name on a passing line cannot satisfy it. Output with no failure lines cannot satisfy it either;
a red run without a crash then reads `RED-ELSEWHERE`.

## Why it is built this way

**The idea.** A self-test that cannot be made to fail proves nothing. Break the line it guards; the self-test must go red.

**Where it came from.** Own practice, 2026-08 to 2026-09: the rule came from a detector whose self-test stayed green while the real run emitted 266 false positives, and was refined by the cases in `references/incidents.md`.

**Evidence.** What was broken on purpose to show that the self-tests can fail is under [Verify](#verify); what was run end to end, and in which agent, is under [Compatibility](#compatibility).

## Install

Pick one of four ways: three for Claude Code, one for OpenAI Codex. Skills load when a session starts, so open a **new** session after installing.

### 1 · Terminal, one command

```bash
git clone https://github.com/NickkkLian/nk-breakable-selftest ~/.claude/skills/nk-breakable-selftest
```

1. Run the command above (for one project only, clone into `.claude/skills/nk-breakable-selftest` inside that project).
2. Start a new Claude Code session.
3. Check it loaded: type `/nk-breakable-selftest` — it appears in the slash-command menu. Or just ask for the task; the skill triggers on its own.

### 2 · Claude Code in a terminal session (plugin)

The plugin route goes through the [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) marketplace. Add it once; after that each skill is one command.

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-breakable-selftest@nickkk-skills
```

1. In a Claude Code session, run the first line (once per machine).
2. Run the second line.
3. Start a new session (or run `/reload-plugins`). The skill shows up as `nk-breakable-selftest:nk-breakable-selftest`.

Without opening a session, the same two steps work from a shell: `claude plugin marketplace add NickkkLian/nickkk-skills` then `claude plugin install nk-breakable-selftest@nickkk-skills`.

### 3 · Claude desktop app (Code tab)

**Add the marketplace first — Discover only searches marketplaces you have already added.**

<img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/panel-route.gif" alt="Adding the marketplace and installing a skill in the desktop app" width="640">

<sub>Recorded on 2026-09-16, when the marketplace listed ten skills, all at version 0.1.0; it lists more now. The repository list in this recording shows the recorder's own repositories because a GitHub account is connected; yours will show yours. Type the full name as in step 4.</sub>

1. In the chat box, type `/plugin marketplace` and press Enter (or open **Settings → Customize → Plugins**). The **Plugins** panel opens.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step1-type-plugin-marketplace.png" alt="/plugin marketplace typed in the chat box" width="480">
2. Top right, open **Add ▾** and choose **Add marketplace**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step2-add-menu.png" alt="The Add menu with Add marketplace" width="480">
3. Choose **Add from a repository**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step3-add-from-repository.png" alt="Add marketplace dialog: Add from a repository" width="480">
4. In **URL**, type the full `NickkkLian/nickkk-skills`. At the bottom of the list choose the row **Use "NickkkLian/nickkk-skills"**, then press **Sync**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step4-url-then-sync.png" alt="URL filled in, Sync button" width="480">
5. You land on **Discover**, filtered to the new marketplace (**Filter · 1**). Find **Nk breakable selftest** and press **Add**. Installed ones show **✓ Added**.
   <br><img src="https://raw.githubusercontent.com/NickkkLian/nickkk-skills/main/gallery/panel-route/step5-discover-add.png" alt="Discover list with Added and Add buttons" width="480">
6. Close the panel and start a new session.

To try it for one session without installing anything: `claude --plugin-dir ./nk-breakable-selftest` from a clone.

### 4 · OpenAI Codex CLI

```bash
git clone https://github.com/NickkkLian/nk-breakable-selftest.git ~/.agents/skills/nk-breakable-selftest
```

1. Run the command above (for one project only, clone into `.agents/skills/nk-breakable-selftest` inside that project).
2. Start a new Codex session.
3. Check it loaded, without spending a model call: `codex debug prompt-input | grep -o -- '- nk-breakable-selftest[a-z0-9:-]*' | sort -u` prints `- nk-breakable-selftest:nk-breakable-selftest:`. Codex adds the `nk-breakable-selftest:` prefix because this repository also carries a Claude Code plugin manifest. Ask for the task and the skill triggers on its own, or type `$` and pick it from the list.

## Compatibility

| Agent | Tested | What was checked |
|---|---|---|
| Claude Code (CLI 2.1.173, macOS) | yes | In a fresh project with an isolated Claude config, inside a macOS sandbox that blocked reading the tester's ~/.claude folder (settings, session history, memory), Desktop, Documents and Downloads, SSH keys and git identity, a plain request that never names the skill triggered it and it ran its bundled script. The route 2 plugin commands were also run from a shell with an isolated config: marketplace add, install, list. |
| OpenAI Codex CLI (0.154.0-alpha.6.2, gpt-5.6-sol, low reasoning, macOS) | yes | Copied into `~/.agents/skills` of a temporary home (the folder route 4 clones into), in a fresh project, without the user's Codex config. From a plain request that never names the skill, Codex read SKILL.md, ran `scripts/breakcheck.py` against the checker: the control stayed green, two mutations were caught, and the two for the branch the self-test never feeds were reported uncovered. |
| Cursor, Gemini CLI | no | Not tested. Their documentation says both read `~/.agents/skills`, the folder route 4 clones into; Gemini CLI asks before it activates a skill. |

In this skill's Codex run, every call into the skill folder's scripts/ used that folder's absolute path. Route 4 was checked for this repository: cloned from GitHub into a temporary home's `~/.agents/skills`, it was listed by the step 3 command. This skill's frontmatter uses only name, description, license and metadata.

## Verify

```bash
python3 scripts/breakcheck.py --selftest
```

Standard library only, Python 3.9+. On 2026-10-01 every self-test above passed, and
`breakcheck.py` from [nk-breakable-selftest](https://github.com/NickkkLian/nk-breakable-selftest) broke each script on purpose in a sandbox copy (the tool run on itself):

- `breakcheck.py`: 6 lines broken one at a time; each turned the self-test red without a traceback.

The unmutated control stayed green every time. Only lines that record a finding, raise, or return a failing exit code
were broken (the tool's pattern, or the hand-written list); a line number refers to the script as shipped in this version.
This shows those lines are covered. It does not show that nothing else can fail.

## Limits

- `--auto` only understands Python line structure; for shell/JS/other files use `--spec`.
- `--auto` skips docstrings (0.1.3); a pattern word inside any other string still counts as a line.
- To see a matrix before pointing it at your own code: `python3 ${CLAUDE_SKILL_DIR}/scripts/breakcheck.py --demo`.
- Point `--auto` at the file that does the checking, not at the self-test's own assertions: neutralising an assertion can only make the self-test *more* lenient, so every such row reads UNCOVERED by construction.
- Neutralising a line that is part of a multi-line expression yields `CRASH`; use `--spec` for those.
- The matrix proves the self-test reacts to the breaks you listed. It says nothing about failure modes nobody wrote a rule for (see rule 8).

## License

MIT. Read a script before letting it run in your environment.
