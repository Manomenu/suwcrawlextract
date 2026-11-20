# suwcrawlextract

Once a day, an AI agent reads what changed in the owner's apps and asks one question: is any of it worth moving into a
shared kit ([suwgate](https://github.com/Manomenu/suwgate), [suwfrontkit](https://github.com/Manomenu/suwfrontkit),
[suwbackkit](https://github.com/Manomenu/suwbackkit)), or did an app write again what a kit already ships? Each answer
is an issue in this repository. Nothing else is changed anywhere: the owner decides.

```
Claude Code cloud scheduled task (daily)
  clones: suwcrawlextract + the tracked apps + the three kits
  reads:  REVIEW.md here (the whole procedure)
  diffs:  each app since the commit the last run reviewed (the issue labelled `state`)
  writes: issues labelled `candidate` here, and the new state
```

There is no code: the procedure is [REVIEW.md](REVIEW.md), the state and the proposals are issues. A change of the rules
is a commit to REVIEW.md, and the next run follows it.

## Set up (once)

1. **GitHub access for cloud sessions.** In a terminal where `gh` is logged in as the owner, start `claude` and run
   `/web-setup`: it hands your `gh` token to your claude.ai account, so cloud sessions can clone every repository the token
   reaches, the private ones too. (The other way is installing the Claude GitHub App on each private repository.)
2. **The scheduled task**, at [claude.ai/code/routines](https://claude.ai/code/routines) or with `/schedule` in Claude Code:
    - prompt: `Read suwcrawlextract/REVIEW.md and follow it to the end.`
    - repositories: `suwcrawlextract` first, then `suwstockdrill`, `alfusion`, `suwagcms`, `suwalski-agent-eval`,
      `suwchatbot`, `solid-dotnet-app-tpl`, `suwgate`, `suwfrontkit`, `suwbackkit`
    - environment: Default (limited network is enough: only GitHub, through the proxy)
    - schedule: daily, a few minutes past the hour (on the hour can start late), e.g. 06:07
    - connectors: remove them all; the run needs none
3. **First run:** on the task's page, **Run now**, then open the run and read its summary. The first run reviews the last
   14 days of each app.

## Use it

- An issue labelled `candidate` is a proposal. Do it (move the code, then close the issue), or close it as not planned with
  one sentence why: the next runs read that sentence and stop proposing the like.
- Track another app: add it to the list in REVIEW.md and to the task's repositories.
- Usage: each run is a normal Claude Code cloud session and counts against the plan's usage limits; nothing is billed
  extra unless usage credits are turned on at claude.ai/settings/usage. Without them, a run that hits the limit is refused
  until the window resets.

## Licence

MIT, see [LICENSE](LICENSE).
