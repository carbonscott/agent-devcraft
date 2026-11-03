# agent-devcraft

Development skills for AI coding agents: issues that show their problem is real, and PRs
that prove they work.

## Install

```sh
git clone https://github.com/carbonscott/agent-devcraft
cd agent-devcraft
./install.sh
```

This symlinks each skill into `~/.claude/skills/`. Restart Claude Code and the
`/<skill-name>` commands appear. Options: `--copy` (copy instead of symlink), `--project`
(install into `./.claude/skills`), `--target=<dir>`. Install refuses to overwrite anything
already there. `./uninstall.sh` takes the same options and removes only symlinks that
point into this repo.

## Skills

| Skill | What it does |
|---|---|
| `/issue-bar` | Assess issues against the evidence bar and draft agent-labeled comments that ask for what is missing. |
| `/proof-ready-pr` | Open PRs that prove, in the PR, that the change works end to end; check other PRs for that proof and draft a request when it is missing. |

## The two bars

**Issues need evidence.** A bug needs a reproduction on the current version. A performance
issue needs a measurement and a target. A feature request needs someone who hit the
problem, what they do today, and what that costs. A proposal must name a problem today's
code can't solve; a standard on its own is not one. Rules and trivial changes are left to
their authors.

**PRs need proof.** In the words of the [Amp](https://ampcode.com) team's prompt: "I want
you to give me 100% proof that what you did works. Test this end to end. In many ways. Run
through a matrix of test cases. Show me a screenshot, a video, anything that's solid proof!
Dazzle me." The
proof lives in the PR description, with the commit SHA and the date. It is a review aid
for agent-written PRs, not an engineering-practice rule, and changes whose effect is fully
visible in the diff skip it.

Every comment these skills draft opens with a line saying an agent wrote it, and nothing
is posted until you say so.

## Tests

```sh
bash tests/test.sh
```

It checks install, uninstall and each skill's frontmatter. It also runs in CI on every
push.
