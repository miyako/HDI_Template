# 4D HDI ("How Do I") example

This repository is one of the **HDI** (*How Do I*) examples: a small 4D project, originally published by
4D for one release (mostly v16 / v17, as a binary `.4DB`), that demonstrates **one** 4D feature. It has
been converted to the `.4DProject` architecture (or restored) so that it runs on current 4D releases, and
is then modernised one effort at a time. The repositories are created from
[miyako/HDI_Template](https://github.com/miyako/HDI_Template); the index of all HDIs is
[miyako/4d-hdi](https://github.com/miyako/4d-hdi).

- `Project/<Name>.4DProject`, `Project/Sources/`: the 4D project (forms, methods, catalog, `menus.json`).
- `Resources/`: XLIFF files in `<lang>.lproj/` (English source, Japanese), sample data, images
  (`Images/HDIabout.png` is the splash background).
- `README.md`: the developer-facing guide to the demo.
- In HDI_Template itself there is no `Project/` yet: it holds the agent setup and the README template.

### Forms and methods every HDI shares

| Item | Role |
|---|---|
| `onStartup` database method | Calls the start method (usually `00_Start`) with no argument. |
| `00_Start` | Opens the splash form `HDI` with an options object (title, info, blog, minimum version, licence). |
| Form `HDI` | The standard splash: title, blog link, version / licence check, button `BtnDemo` that opens `HDI2`. |
| Form `HDI2` | The real demo. This is what the repository is about. |

The splash conventions (options, XLIFF keys, `BtnDemo`, error state) are in
`.github/instructions/hdi.startup.instructions.md`.

## Repository rules

1. **The demo is the point.** Never change what the demo demonstrates. Modernisation changes how the code
   is written, not what `HDI2` shows; keep the original feature, data and flow.
2. **One branch per modernisation effort.** Start each effort from the default branch on its own branch,
   named after the effort (kebab-case). Do not mix efforts in one branch.
3. **Each effort follows its skill** (table below). Do not rely on memory of 4D: read the skill first.
4. **Never invent token suffixes** (`:C123`, `:K12:34`). Write plain command and constant names unless the
   exact token already appears in the project (rules in skill `4dproject`).
5. **Validate what you change**: `.4dm` with skill `4dlsp`, `.4DForm` with skill `4dform`, XLIFF with
   `xmllint` (skill `4dtools`). Report what could not be validated.
6. **README rules**: `README.md` follows `.github/instructions/readme.structure.instructions.md`; its
   Modernisation notes table follows `.github/instructions/readme.branches.instructions.md` and is built
   from branches that really exist on the remote.
7. Do not commit `Data/`, `Logs/`, `Settings/`, provisioned `tools/4d*/` folders or secrets (see
   `.gitignore`).

### Modernisation efforts

| Effort | Use |
|---|---|
| `C_*` → `var` / `#DECLARE`, `Compiler_*` methods, deprecated `_o_` commands | skill `4dmodernise` (version detection: `4dproject`) |
| Splash / startup: `CALL WORKER`, `DIALOG(...; *)`, window reuse, no `QUIT 4D` | skill `4dstartup` + `.github/instructions/hdi.startup.instructions.md` |
| Menu items that wrap one command (`m_Quit`) → standard actions in `menus.json` | skill `4dproject` (`references/menus.md`) |
| Hide subroutines and form-dependent methods from Run Method (`invisible`) | skill `4dmethods` |
| XLIFF localisation of menus, forms, tips and messages (English + Japanese) | skill `4dlocalise` |
| Dark mode, `automatic` colours, Liquid Glass / Fluent UI button heights | skill `4dcss` |
| List box defaults (`truncateMode: none`, `resizingMode: legacy`) | skill `4dform` |
| 4D AI Kit: asynchronous calls and streaming | skill `4daikit` |
| Rewrite `README.md` as a developer guide | `.github/instructions/readme.structure.instructions.md` |

## Workflow and checkpoints

1. **Survey.** Read `Project/*.4DProject` (`compatibilityVersion`, `tokenizedText`; skill `4dproject`) and
   list the forms, methods and existing branches. **STOP:** propose the efforts that apply and their order.
2. **Effort branch.** Create the branch, apply the skill to the whole project (every form, every method --
   not only the first match), validate. **STOP:** summarise the changes and the validation results; let the
   user try the demo in 4D before merging.
3. **README.** Add the branch's row to the Modernisation notes table once the branch is on the remote, and
   keep the rest of `README.md` in line with the code.

## Skills

Reusable 4D knowledge lives in the `.agents` submodule (`miyako/skills`, branch `dist`):

- Each skill is at `.agents/skills/<name>/SKILL.md`. Copilot, Codex and Claude Code discover them automatically.
- Read `.agents/AGENTS.md` first: it explains which skill to use for which 4D artifact
  (`.4DProject`, `.4DForm`, `.4DCatalog`, `.4DSettings`, `.4dm`, XLIFF …).
- Prefer the most specific skill over general knowledge of 4D.
- Never edit files under `.agents/`: they are overwritten by the weekly skills update.

## Setup

```sh
git submodule update --init .agents
```

If `.agents/` is empty, run the command above before doing anything else. `--recursive` is not needed.
Tools provisioned by the skills go to `tools/4dcatalog/`, `tools/4dform/`, `tools/4dlsp/`, `tools/4dlang/` and
`tools/4dcli/`;
they are ignored by `.gitignore` and must not be committed.

## Instructions vs skills

- **Repository-specific rules** go in this file, or in `.github/instructions/*.instructions.md` with an
  `applyTo` glob when they only apply to some paths.
- **Reusable know-how** (4D language, forms, CSS, XLIFF, project settings …) goes to
  [miyako/skills](https://github.com/miyako/skills). Never copy a skill or its content into this repository;
  improve the skill there instead, and mention what you learnt to the user so it can be upstreamed.

## README

`README.md` is for developers. Never put Copilot model, mode, token or session content (usage tables,
model selection advice, session logs) in `README.md`.
