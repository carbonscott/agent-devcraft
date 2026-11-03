---
name: issue-bar
description: Assess GitHub issues against an evidence bar (is the problem real, shown, and something today's code can't do?) and draft agent-labeled comments that close the evidence gap. Use whenever the user asks to triage, review, assess, clean up or respond to issues, a backlog or an issue tracker, asks whether an issue is real, actionable, justified or worth doing, or wants to know which feature requests to park, even if they never say "evidence bar".
argument-hint: <owner/repo> [issue numbers]
---

# Issue bar

Keep the tracker full of actionable, justified work. Nothing moves forward on opinion
alone: an issue has to show that its problem or need is real and understood. Your job is
to find the evidence gap and ask for what would close it. Deciding what gets built stays
with people; an issue becomes work only after a person accepts it.

Issues need **evidence, not proof**. Irrefutable proof is the PR's job (see the
`proof-ready-pr` skill). Do not build a dossier of permalinks for every claim in an issue;
one quick check that changes the verdict is worth it, and a fact-check of every sentence
is not.

## 1. Gather

- `gh issue list -R <repo> --state open --limit 200 --json number,title,author,createdAt,labels,comments`
- `gh issue view <n> -R <repo> --json body,comments,labels` for each item. Read each one
  once, comments included.
- Note what the list alone hides: issues filed together in one batch, issues that depend
  on each other, linked PRs, and agent-written text (trailers such as `Assisted-by:`).

## 2. Type each issue and hold it to its bar

| Type | Evidence it needs before it is actionable |
|---|---|
| Bug | A reproduction on the current version: steps, a script or a failing test, with actual against expected output. |
| Performance | How it was measured (benchmark or profile), the environment, real numbers, and a target. "Feels slow" does not count. |
| Feature request | Who actually hit this, what they do today (the workaround and its cost), and how often it comes up. |
| Idea or proposal | A problem today's code can't solve, stated as a problem ("users can't do X"), not a solution ("build Y"). It stays a proposal until a person accepts it. |
| Docs gap | Where a reader (a person or an agent) needed something, did not find it, and had to guess. |
| Unclear | Not actionable yet. Give your best reading, try it under that reading, and ask specific questions. |
| Rules or process | Contribution rules, templates, naming. The authors' call: list it, do not assess it. |
| Trivial | The whole effect is visible in the issue. Nothing to assess. |

## 3. Ask the central question

For every feature request and proposal: **what problem does this solve that the current
code can't, and for whom, today?**

- **A standard alone is not a problem.** A spec, a format other tools use, or "best
  practice" becomes a reason only when someone needs it: a named user, system, platform
  or deadline. Issues that start from a spec instead of from something that went wrong
  are wishlists until someone shows up who needs them.
- **Look for the cheap workaround.** If the system already does most of it (for example,
  a free-form field already holds the value), the request must show where that falls
  short. A request with no real user behind it and a cheap workaround is weak.
- **Ask what breaks if we never do this.** "Nothing breaks, but we'd match standard X"
  means it is an idea, and it stays parked.
- **Check external premises.** If an issue says "X requires field Y", look it up at a
  pinned version. A wrong premise can remove the stated need entirely.

## 4. Check cheaply, and keep sources straight

Spend minutes, not hours. Worth checking: does the bug still reproduce on the default
branch (line numbers drift), was it fixed on another branch, are the PRs an issue cites
still open, does a number have a source. Mark each fact as observed (you ran or read it),
reported (someone else's account) or inferred (your reasoning). Say what you did not check.

## 5. Give each issue a verdict

- **Evidence bar:** Meets, Partly, Missing, or n/a (rules, trivial).
- **Do:** Ready (accept), Needs info, Park (no user yet), Close here (belongs in another
  tracker), Owner's call (a decision only the maintainer can make), Their call (rules,
  trivial).
- **Why:** one line.

Recognise these patterns, because they recur:
- **Implementation ahead of acceptance:** a PR that builds an open proposal, or takes an
  issue's "decision needed" on its own.
- **A dependency web of proposals:** A needs B, C and D, none accepted. Park the web, not
  one node of it.
- **A shared blocker:** if one issue in a batch says it should wait for something (a real
  user, a target system), check whether that reason covers the rest of the batch.

## 6. Draft comments that close the gap

Draft only where a comment changes something. Every comment opens with this line, so
readers can choose to read it themselves or hand it to their own agent:

```
🤖 **Agent comment.** Written by an AI agent; read it yourself or hand it to your own agent.
```

- **Ask specific questions.** "Who hit this, on which version, and what did they do
  instead?" works. "Can you clarify?" does not.
- **Lead with the problem question,** and leave out solution-level advice (merging issues,
  field names, ordering) on proposals nobody has accepted. Engaging with design details
  reads as acceptance.
- **Keep the facts that show a need isn't real:** a wrong premise, nothing at risk yet.
- **One note for a cluster.** When several issues share a gap, put the full note on one of
  them and a short pointer, plus any item-specific line, on the rest.
- **Assess the issue, not the person.** Credit what is good: a real reproduction, real
  measurements, an honest "not measured".

## 7. Report, then post only when told

Report with the answer first: how many issues are ready, need info, should be parked, or
are someone's call. Then one table of every item (number, title, type, bar, do, why), the
drafts, and a short list of questions only the user can answer.

Post nothing until the user says so: comments are public and notify people. When told to:

- Post: `gh issue comment <n> -R <repo> --body-file <file>`
- Edit later: `gh api -X PATCH repos/<owner>/<repo>/issues/comments/<id> -F body=@<file>`.
  Before editing, check nobody else changed it. Edits send no notification, so say so
  when the change matters to people who already read the first version.
