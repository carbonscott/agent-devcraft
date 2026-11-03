---
name: proof-ready-pr
description: Open pull requests that prove, in the PR itself, that the change works end to end (a matrix of runs, captured output with commit SHA and date, a screenshot, recording or log, before-and-after for bug fixes), and check other people's PRs for that proof, drafting a request when it's missing. Use whenever you are about to open or finish a PR, write a PR description, or say a change is done or works, and whenever the user asks to review a PR, check whether a PR is ready to merge, or ask an author for proof, especially for PRs an AI agent wrote.
argument-hint: [PR number, branch, or nothing for the current branch]
---

# Proof-ready PR

The bar, in the words of the Amp team's prompt:

> I want you to give me 100% proof that what you did works. Test this end to end. In many
> ways. Run through a matrix of test cases. Show me a screenshot, a video, anything that's
> solid proof! Dazzle me.

**Why:** running the change and capturing the result costs an agent only tokens.
Rebuilding that run from the diff costs a reviewer hours, and reviewers miss things.
Proof shown in the PR moves the work to where it is cheap. This is a review aid, not an
engineering-practice rule: it adds no testing, coverage or CI requirement.

**When it applies:** PRs an agent wrote most of. Skip it for changes whose whole effect is
visible in the diff (wording, comments, templates, a link). The test is simple: if reading
the diff can't tell a reviewer whether the change works, prove it.

## Mode A: make your own PR proof-ready

1. **Start from an accepted issue.** If the issue is an open proposal nobody agreed to,
   say so before building. Don't fold a second, unaccepted feature into the same PR.
2. **Design the matrix before running anything.** List the dimensions the change touches:
   input kinds, modes or sources, install modes (checkout and installed package),
   platforms, real data and synthetic data, failure cases. Choose cells that would break if
   the change were wrong. Include a real-world case whenever one exists.
3. **Run it end to end** through the real entry points (the CLI, the server, the API).
   Unit tests with mocks are welcome, but they are not end-to-end proof.
4. **Fit the proof to what the change claims.** A PR often claims more than one of these;
   prove each claim it makes.

   | The change claims to | Proof |
   |---|---|
   | Fix a bug | The issue's own reproduction fails at the base commit and passes at the head. Put base and head in two `git worktree`s and run both. |
   | Add a feature | The need met: the user's actual workflow, running. |
   | Make something faster or smaller | The same measurement before and after, in the same environment, repeated, with the spread and the command to rerun it. |
   | Change nothing visible (a refactor) | The same inputs give the same outputs at base and head, compared directly. |
   | Handle failures | Each failure triggered on purpose (bad input, a missing permission, a resource limit, a lost connection), with what the user sees and whether it recovers. |
   | Be correct in ways the output doesn't show | A tool that makes it visible, such as a sanitizer, a race detector, or a comparison against a trusted reference, run on the inputs the change touches. |
   | Document a procedure | The procedure run exactly as written, from the reader's starting point (for example a package install, not a checkout). |

   Record the environment with every result: versions, hardware where it matters, and the
   ID of any job or run, so the result can be traced and repeated.

5. **Capture it:** the commands, their output, the commit SHA and the date. Anything
   visual (rendered docs, plots, UIs, notebooks) gets a screenshot or a recording. For
   command-line work the captured log is the screenshot; a terminal recording can help.
6. **Say what you could not run, and why:** an environment you can't reach, special
   hardware, production data or scale. The list of gaps is part of the proof.
7. **Check your claims against CI.** Every check you say passes must pass in CI too. When
   CI disagrees with your local run, CI wins: fix it before asking for review.
8. **Keep the proof out of the diff.** Put it in the PR description, or link it from
   there. Evidence, notes and logs committed to the repo bury a small change under
   thousands of lines and defeat the purpose. Attach images to the description (ask the
   person to drag them in if you can't upload) or host them outside the PR's diff.
9. **Report only what you ran.** Never present a number you did not produce in this
   session as your result. If a run did not finish, say so.

Fit the repo's own PR template and checklist, and add this section to it:

```markdown
## Proof it works
Ran on <machine>, at <commit SHA>, on <date>.

| Case | Command | Result |
|---|---|---|
| ... | ... | ... |

<details><summary>Raw output</summary>

...

</details>

Before and after (bug fix): base <sha> exits 1 ...; head <sha> exits 0 ...
Not run: <what, and why>
To rerun: <commands>
```

## Mode B: check someone else's PR

You are assessing whether the PR shows proof. Don't run the change yourself unless asked.

1. **Read what the PR shows:** the body, the diff size, and its own CI.
   `gh pr view <n> --json body,files,commits,statusCheckRollup,mergeable`, `gh pr checks <n>`,
   and `gh run view <run-id> --log-failed` for a failing check.
2. **Sort it.** Rules or a diff-visible change: the authors' call, no request. Anything
   else: does it show captured end-to-end runs with a SHA and a date? Mocked tests and
   descriptions of runs ("I started a server and it worked") do not count.
3. **Note contradictions:** a check the body says passes but CI fails, merge conflicts,
   claims with no output behind them.
4. **Draft the request,** with a matrix tailored to this PR (the issue's reproduction for
   a bug fix, real inputs, the ways the code is installed or run):

   ```markdown
   🤖 **Agent comment.** Written by an AI agent; read it yourself or hand it to your own agent.

   **I want you to give me 100% proof that what you did works. Test this end to end. In many ways. Run through a matrix of test cases. Show me a screenshot, a video, anything that's solid proof! Dazzle me.**

   <One line: what the PR shows now, and what is missing.> A matrix that would cover it:

   - **<dimension>:** <cells>
   - **<dimension>:** <cells>

   Whatever you show, keep the commit SHA and the date in frame, and paste the text output too so others can rerun it.
   ```

5. **Post only when the user says so:** `gh pr comment <n> --body-file <file>`. To edit
   later, `gh api -X PATCH repos/<owner>/<repo>/issues/comments/<id> -F body=@<file>`.
   Edits send no notification.

## Proposing this bar for a repo's PR template

If asked to suggest the bar to a project, propose this section, open with the scope (for
easier review, not an engineering-practice rule), and cite PRs from that repo as examples,
not issues:

```markdown
## Proof it works (required if an AI agent wrote most of this PR)
Show the change working, so a reviewer can check the result instead of rebuilding it from the diff.
- Run it end to end, in more than one way: a matrix of the cases it touches.
- Show the result in whatever form fits: a log, timings, tool output, a screenshot or a recording.
  Keep the commit hash and date in it, and paste the text output so a reviewer can rerun it.
- Bug fix: the reproduction from the issue fails before the change and passes after it.
- Performance: the same measurement before and after, on the same hardware, with the command to rerun it.
- If something can't be run here, say what and why.
- Put the proof in the PR description, or link it there. Don't commit it to the repo.
Skip this section for changes whose effect is fully visible in the diff, such as wording, comments or templates.
```
