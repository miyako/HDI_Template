---
description: "Uniform README structure for HDI example repositories - developer-facing feature guides covering what the demo shows, key 4D commands, how the code works, points of interest, and references; excludes all Copilot model/token/session content"
---

# Uniform README specification for `4d-hdi` repositories

Every repository tagged `4d-hdi` must have a `README.md` that follows this exact
structure. The README is written **for a developer who lands on the repo looking
for insight into a 4D feature** -- not for the maintainer. Nothing about Copilot
models, token cost, session history, or prompts belongs in it.

---

## Required structure (in this exact order)

```markdown
![version](https://img.shields.io/badge/version-20%2B-E23089)
![platform](https://img.shields.io/static/v1?label=platform&message=mac-intel%20|%20mac-arm%20|%20win-64&color=blue)

# <RepoName>

<One sentence: what 4D feature this demo showcases.> Originally published by 4D as a
**HDI** (*How Do I*) example for **4D <version>**; <converted from the binary `.4DB`
to the `.4DProject` architecture | restored> so it runs on current 4D releases.

## What it demonstrates

- <bullet: a concrete capability the demo shows, phrased as a developer takeaway>
- <4 to 8 bullets; each must be backed by code that actually exists in the repo>

## Key commands

| Command | Used for |
|---|---|
| `<4D command or function>` | <what the demo uses it for> |

## How it works

<2 to 5 short paragraphs, or a bulleted walkthrough, describing the actual code:
which forms exist, which methods drive them, and the flow a reader should follow
first. Name real files. Point out the single most interesting piece of code.>

## Points of interest

- <bullet: a gotcha, a platform difference, a deprecation, a behaviour that
  surprised you, or advice that is not obvious from the documentation>
- <2 to 5 bullets. Omit the whole section only if there is genuinely nothing.>

## Modernisation notes

<Converted repos only. Short prose plus the existing Branches table, unchanged.>

| Branch | Description | Instructions |
|--------|-------------|--------------|
| ... | ... | ... |

## References

- [4D blog: <post title>](<url>)
- [4D documentation: <page>](<url>)
- Original download: [<file>.zip](<url>)
- Index of v16/v17 HDIs: [miyako/4d-hdi](https://github.com/miyako/4d-hdi)

## Screenshots

<keep the existing screenshot <img> tags, unchanged>
```

---

## Rules

1. **Delete entirely**, wherever they appear:
   - `## Copilot Token Usage` and its table
   - `## Model Selection Assessment` and all its subsections
   - `### Mode Selection Guidance`
   - Any cross-project token comparison, "Key Takeaways", or
     "How to make it even better next time" prose
   - Any mention of Claude/Opus/Sonnet/Haiku, token counts, turns, or session names
   - The `## Origin` heading (fold its content into the intro paragraph and the
     References section)

2. **Keep, verbatim**:
   - All existing `<img>` screenshot tags, under `## Screenshots`
   - The `## Branches` table rows and their instruction-file links (move under
     `## Modernisation notes`)
   - The blog post URL and original download URL (move into `## References`)

3. **Every factual claim must come from the repo.** Read the `.4dm` method files,
   the form `.4DForm` JSON, and the `.4DCatalog` before writing. Do not describe
   behaviour you have not seen in the code. Do not invent 4D command names --
   copy them from the source, dropping the `:C1234` token designator.

4. **Command table**: list only commands central to the demo, typically 4-10 rows.
   Skip boilerplate (`ALERT`, `Localized string`, `ARRAY TEXT`, compiler methods).

5. **Tone**: terse and technical. British spelling. ASCII punctuation only --
   use `--` rather than an em dash, straight quotes, no ellipsis character.

6. **Restoration repos** (those whose README currently says
   "restoration of vNN demo") have no `## Branches` table and no blog post.
   For those: omit `## Modernisation notes`, and in `## References` link the 4D
   documentation page for the feature plus the `miyako/4d-hdi` index. Keep the
   existing badges and screenshots. The intro sentence uses "restored" wording.

7. **Do not touch any file other than `README.md`.**

---

## Delivery

For each repo:

```bash
git checkout -b readme-developer-guide
# edit README.md
git add README.md
git commit -m "Rewrite README as a developer-facing feature guide"
git push -u origin readme-developer-guide
gh pr create --title "Rewrite README as a developer-facing feature guide" --body "<summary>"
```

Commit trailer to include on every commit:

```
Co-authored-by: Copilot App <223556219+Copilot@users.noreply.github.com>
```
