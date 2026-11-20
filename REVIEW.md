# The daily review

You are the daily run of suwcrawlextract, started by a Claude Code cloud scheduled task. The task clones this repository,
the tracked repositories and the three kits next to each other. Follow this file to the end; it is the whole procedure.

Your one question: **is there anything in the new changes of the tracked repositories that is worth moving into suwgate,
suwfrontkit or suwbackkit, or that an app wrote again although a kit already has it?** You answer by opening issues in
`Manomenu/suwcrawlextract`. You never change, push to or open pull requests in any other repository.

## The repositories

Tracked (read what changed): `suwstockdrill`, `alfusion`, `suwagcms`, `suwalski-agent-eval` (suwageval), `suwchatbot`,
`solid-dotnet-app-tpl` (the template every new app is generated from).

Kits (read what they already hold): `suwgate`, `suwfrontkit`, `suwbackkit`. Read each kit's README.md, docs/architecture.md
and TODO.md before you judge anything: a candidate already listed in a kit's TODO.md is not new.

All live under `https://github.com/Manomenu/`. Each is cloned into its own folder next to this one. A tracked repository
missing from the session is reported in the run summary (step 6), never guessed at.

## What belongs where

- **suwgate**, the quality gate (`suwgate check`, `suwgate.yml`, the CLI guards, the tool presets, the hooks, the NuGet
  guards `Suwgate.Architecture` and `Suwgate.Database`, the reusable CI workflows): a check, a guard, a lint rule, a gate
  step, a tool config or a CI job that a second repository would copy. It depends on no kit.
- **suwfrontkit**, the web look (Catppuccin Mocha, dark only, desktop only 16:9 and 21:9, React components, the theme):
  a visual component, a layout block, a formatting helper or a style any of the web apps could use as it is. One component per
  job: if the kit has a component for the same job, the candidate is a change of that component, never a second one.
- **suwbackkit**, the .NET runtime (`Suwbackkit.Server`, `Suwbackkit.Postgres`, later more `Suwbackkit.<Area>`): code
  that **at least two apps** already have in some form. One app's need stays in that app however general it looks; the
  second app that needs it is what earns it a place. No domain logic of any one app.
- **The template**: generic code that landed in the template belongs in a kit; the template only wires.

Not a candidate: domain logic of one app; code that is generic but used once, unless it is a suwfrontkit component or a
suwgate check whose use is obvious elsewhere; anything a kit's TODO.md or an open issue here already covers.

Also a candidate, often the most valuable one: **an app that wrote again what a kit already ships** (its own button, its
own copy of a guard, its own health endpoint). Then the proposal is "use the kit's X", with the places to change.

## Steps

1. **State.** Read the open issue labelled `state` in `Manomenu/suwcrawlextract` (its body is a YAML block: repository
   name → the last commit of its default branch that a run reviewed). A repository without an entry is reviewed over its
   last 14 days.
2. **What changed.** In each tracked repository: `git log --no-merges <reviewed>..HEAD` on the default branch, then the diff.
   Nothing new anywhere: go to step 5.
3. **Judge.** For each change that looks generic, read the code around it, not only the diff, and look for the same thing
   in the other tracked repositories and in the kits (`rg` across the folders). Write down the evidence: file paths at
   commit SHAs in every repository that has it. Drop what fails the rules above.
4. **Report.** Before writing, list every issue labelled `candidate`, open and closed
   (`gh api "repos/Manomenu/suwcrawlextract/issues?state=all&labels=candidate&per_page=100"`), and read the last comment
   of the closed ones: the owner's reason for closing says what they do not want. Then for each surviving candidate:
    - already an open issue about it: add a comment with the new evidence, nothing else;
    - closed as not planned: propose it again only when the evidence is new in kind (another app has it now), and say so;
    - otherwise open a new issue (format below).
    At most **3 new issues per run**: the strongest first. Fewer, well-proven proposals beat many weak ones; a run with
    no proposal is a good run when nothing qualifies.
5. **Save the state.** Set every tracked repository's entry in the `state` issue to the commit you reviewed up to, and
   PATCH the issue body (`gh api -X PATCH repos/Manomenu/suwcrawlextract/issues/<n> -f body=...`). Do this even when
   nothing was proposed, so the next run starts where this one ended.
6. **Summary.** End the session with a short summary: the repositories and commit ranges reviewed, the issues opened or
   commented on, and the candidates you dropped with one reason each.

## The issue

Write the issue in Polish (the owner reads Polish); code, names and paths stay as they are.

- **Title:** `[<kit>] <what moves, in a few words>`, for example `[suwfrontkit] Pasek postępu z alfusion i suwagcms`.
- **Labels:** `candidate` and one of `kit:suwgate`, `kit:suwfrontkit`, `kit:suwbackkit`; `use-the-kit` when the proposal is
  to use what a kit already has.
- **Body**, these headings in this order:
    - `## Co` — what it is, two or three sentences.
    - `## Gdzie jest dziś` — a link to each place, at its commit SHA (`https://github.com/Manomenu/<repo>/blob/<sha>/<path>#L<a>-L<b>`).
    - `## Dlaczego do kitu` — which rule above it meets; for suwbackkit, name the two apps.
    - `## Jak by wyglądało w kicie` — the proposed name and shape (a signature, props or a guard id), short; what the apps
      would delete afterwards.
    - `## Koszt i ryzyko` — S/M/L, and what could break.
    - last line: `Klucz: <kit>/<short-slug>` (the run uses it to find the issue again).

## How to call GitHub

The cloud session reaches GitHub through Anthropic's proxy: use `gh api` with REST paths
(`gh api repos/Manomenu/suwcrawlextract/issues -f title=... -f body=... -f "labels[]=candidate"`). `gh issue` and `gh pr`
use GraphQL, which the proxy refuses. Write long bodies to a file and pass them with `-F body=@file.md`.
