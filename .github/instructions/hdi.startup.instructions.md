---
description: "HDI splash specifics: the 00_Start options object, the HDI splash form (version / licence check, BtnDemo, error state), the hand-over to HDI2 and the splash XLIFF keys. The generic non-blocking startup pattern is the 4dstartup skill."
applyTo: "Project/Sources/Methods/00_Start.4dm, Project/Sources/DatabaseMethods/onStartup.4dm, Project/Sources/Forms/HDI/**, Resources/**/messages*.xlf"
---

# HDI splash form

Every HDI opens the same splash form, `HDI`, before the demo form, `HDI2`. This file only lists what is
specific to that splash. Read these skills first; do not restate them here:

- **`4dstartup`** (`.agents/skills/4dstartup/SKILL.md`): the start-method pattern (`#DECLARE($params : Object)`,
  window reuse, `CALL WORKER(1; ...)`, `SET MENU BAR(1)`, `Plain form window`, `DIALOG(...; *)`), the
  Accept-action button that opens the next form, `INVOKE ACTION(ak return to design mode)` instead of
  `QUIT 4D`, and the anti-patterns (`New process`, `CLOSE WINDOW`, `Pop up form window`, v16 version checks,
  `Shift down` / `$cr` scaffolding).
- **`4dlocalise`**: XLIFF files, `Localized string`, `:xliff:` references.
- **`4dproject`**: token suffixes (never invent one), `menus.json`.
- **`4dform`**: the `"method"` / `"events"` wiring of object methods in `form.4DForm`.

## Start method `00_Start`

- Called with no argument by `onStartup` (and by the demo menu item); it is an entry point, so it stays
  visible in the Run Method dialog.
- Implements the `4dstartup` pattern with **form `HDI`** and the **empty string** as the window title
  (`$splashWindowTitle:=""`): the reuse loop identifies the splash by that title.
- Code that must run once before the splash (e.g. importing sample data from `Resources/` when the tables are
  empty) goes in the no-argument branch, after the reuse loop and before `CALL WORKER`.
- In the worker branch, pass this options object to `DIALOG("HDI"; $options; *)`:

| Property | Value | Effect in the splash |
|---|---|---|
| `title` | `Localized string("HDI_Title")` | Text of `txtTitle`; hidden when `Null` |
| `info` | `Localized string("HDI_Info")` (e.g. "4D View Pro feature") | `labelInfo` / `txtInfo`; hidden when `Null` |
| `blog` | the blog post URL or host (e.g. `"blog.4d.com"`) | `labelBlog` / `txtBlog` (click: `OPEN URL(Form.blog)`); hidden when `Null` |
| `minimumVersion` | the version the original HDI required, in `Application version` format (`"1700"` = v17, `"1760"` = v17 R6) | `labelVersion` / `TxtVersion` (`{version}` placeholder); error state when `Application version` is lower |
| `license` | optional: `4D View license` or `4D Write license` | error state when `Is license available` is false |

## Form `HDI`

The form method handles `On Load` only:

1. `Form.quit:=False`; hide the label/value pairs whose option is `Null`; set `txtTitle` from `Form.title`.
2. Version check: build "4D vXX[ Rn]" from `Form.minimumVersion` (switch `Icon4D` to `4DR.png` for an R
   release), replace `{version}` in `TxtVersion`. If `Application version<Form.minimumVersion`, enter the
   error state (with `{version}` replaced in `ErrorMainText` too).
3. Licence check (only if `Form.license#Null` and not already in error): if the licence is missing, enter
   the error state with `ErrorViewPro*` or `ErrorWritePro*` texts.

**Error state:** `Form.quit:=True`, `OBJECT SET TITLE(*; "BtnDemo"; Localized string("BtnClose"))`, set
`ErrorMainText` / `ErrorSubText` and make them and `White90` visible.

All labels of the form use `:xliff:` references (form file `HDI{LANG}.xlf`); never hardcode text in the form
method.

## Button `BtnDemo`

- Accept standard action, `"method": "ObjectMethods/BtnDemo.4dm"`, `"events": ["onClick"]` in `form.4DForm`.
- On `On Clicked`: if `Form.quit`, `INVOKE ACTION(ak return to design mode)`; otherwise open `HDI2` with the
  `4dstartup` "next form" pattern, passing `Form` and copying the splash window title
  (`Get window title(Current form window)`) so that re-running `00_Start` finds the demo window.
- In the error state its label is "Close" (`BtnClose`), never "Quit 4D".

## Splash XLIFF keys

ID-based keys in `Resources/<lang>.lproj/messages{LANG}.xlf` (`messagesEN.xlf`, `messagesJA.xlf`), present
in every language:

| Key | English source |
|---|---|
| `HDI_Title` | *(project-specific: what the demo shows, phrased after "How do I ...")* |
| `HDI_Info` | *(project-specific: e.g. "4D View Pro feature")* |
| `BtnClose` | Close |
| `ErrorViewProMain` | Sorry, this "How do I" (HDI) example demonstrates a 4D View Pro feature. |
| `ErrorViewProSub` | You must have a valid 4D View Pro license to continue. |
| `ErrorWriteProMain` | Sorry, this "How do I" (HDI) example demonstrates a 4D Write Pro feature. |
| `ErrorWriteProSub` | You must have a valid 4D Write Pro license to continue. |

Keep the existing wording of these keys when a repository already has them, so all HDIs read the same.

## Checklist

- [ ] `00_Start` follows `4dstartup`, opens `HDI` with the empty window title and the options object above
- [ ] No `Application version<"16xx"` check, no `QUIT 4D`, no `New process`, no `CLOSE WINDOW`
- [ ] `HDI` form method: `Form.quit`, hidden empty options, version and licence checks, error state
- [ ] `BtnDemo`: Accept action, object method referenced in `form.4DForm`, opens `HDI2` with `Form` and the same title
- [ ] Splash keys present in `messagesEN.xlf` and `messagesJA.xlf`; form labels use `:xliff:`
