# nk-breakable-selftest

A [Claude Code](https://code.claude.com) skill. Make a checker, validator, linter, gate or test suite prove it can fail.

Part of [nickkk-skills](https://github.com/NickkkLian/nickkk-skills) — skills that stop an AI coding agent's
"done, tested, safe" from being taken on faith.

## What it does

- Run a **break matrix**: mutate the guarded code one line at a time in a sandbox; the self-test must go red on the named assertion, never via a crash, and the unmutated control must stay green.
- Find **decorative checks**: lines whose removal leaves the self-test green.
- Ten design rules for detectors and guardrails (`references/design-rules.md`), each tied to a real incident (`references/incidents.md`).

The full procedure, the boundaries and where the rules came from are in [SKILL.md](SKILL.md).

## Install

Copy the folder into your skills directory (the skill is the repository root):

```bash
git clone https://github.com/NickkkLian/nk-breakable-selftest ~/.claude/skills/nk-breakable-selftest
```

or inside one project: `git clone … .claude/skills/nk-breakable-selftest`.

As a plugin, through the marketplace in the index repository:

```
/plugin marketplace add NickkkLian/nickkk-skills
/plugin install nk-breakable-selftest@nickkk-skills
```

To try it for one session without installing: `claude --plugin-dir ./nk-breakable-selftest`.

## Verify

```bash
python3 scripts/breakcheck.py --selftest
```

Standard library only, Python 3.9+. Before publishing, the guarded lines of each script were
mutated one at a time in a sandbox copy and the self-test was confirmed to go red on the named
assertion, without a traceback; the unmutated control stayed green.

## License

MIT. Read a script before letting it run in your environment.
