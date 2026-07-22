---
description: "Rules for using CSS stylesheets in 4D projects — dark mode support, automatic color values, media queries, specificity, and runtime color detection"
---

# 4D CSS Stylesheets — Agent Instructions

## Overview

4D applications can use CSS stylesheets to control form object appearance, including macOS/Windows dark mode support via automatic color values, `prefers-color-scheme` media queries, and runtime color detection using hidden reference objects.

---

## Automatic Color Values

4D provides special color keywords that adapt to the system color scheme automatically:

| Keyword | Applies to | Behaviour |
|---------|-----------|-----------|
| `"automatic"` | `stroke`, `fill` | Adapts text/foreground and background to light or dark mode |
| `"automaticAlternate"` | `alternateFill` | Adapts the alternate row background in listboxes |

### Rules

- Use `"automatic"` for text `stroke` and background `fill` whenever a fixed colour is not required.
- Use `"automaticAlternate"` for listbox alternate row fills instead of hardcoded near-white values like `#F8FCFF`.
- When a column-level property is the **same** as the listbox-level property and there is only one column, remove the column-level override and define it at the listbox level only. Column-level properties are for overrides, not repetition.
- Replace hardcoded `#000000` or `#FFFFFF` (or near-equivalents like `#212121`) in `stroke` for text and shape primitives with `"automatic"` **only if** you want them to adapt. If specific branded colours are needed, use CSS instead.
- Form objects that **omit** `fill` or `stroke` entirely use a 4D-internal default, which is **not** the same as `"automatic"`. To ensure they adapt to dark mode, explicitly set `"fill": "automatic"` and/or `"stroke": "automatic"`. This is especially important for full-form background rectangles that rely on the implicit default fill.

---

## CSS Stylesheets in 4D

### File Loading

4D automatically loads these three stylesheets if they exist in `/SOURCES/`:

```
Project/Sources/styleSheets.css          ← cross-platform
Project/Sources/styleSheets_mac.css      ← macOS only
Project/Sources/styleSheets_windows.css  ← Windows only
```

Dark mode rules are cross-platform, so they belong in `styleSheets.css`.

A form can also reference additional CSS files via its `"css"` property:
```json
{ "css": ["myCustomStyles.css"] }
```

### Syntax

4D CSS uses standard CSS selector syntax but the **property names are 4D form object property names in camelCase**, not HTML/CSS property names. Do not assume web CSS properties are valid in 4D.

Reference for property names:
- Form object properties: https://developer.4d.com/docs/FormObjects/propertiesReference
- CSS in 4D: https://developer.4d.com/docs/FormEditor/stylesheets

Common 4D CSS properties (not exhaustive):

| 4D CSS property | Purpose |
|----------------|---------|
| `fill` | Background colour |
| `stroke` | Text/foreground colour |
| `alternateFill` | Alternate row background |
| `visibility` | `"visible"` or `"hidden"` |
| `borderStyle` | Border rendering style |
| `horizontalLineStroke` | Listbox horizontal grid line colour |
| `verticalLineStroke` | Listbox vertical grid line colour |
| `horizontalPadding` | Horizontal cell padding |
| `fontWeight` | `"bold"` or `"normal"` |
| `fontStyle` | `"italic"` or `"normal"` |
| `textAlign` | Text alignment |
| `fontSize` | Font size |

### Media Queries

```css
@media (prefers-color-scheme: light) {
  .my-class {
	fill: #F1F6FB;
	stroke: #2C435A;
  }
}

@media (prefers-color-scheme: dark) {
  .my-class {
	fill: #1e2d3d;
	stroke: #B8D4E8;
  }
}
```

### Alpha Channel

4D does **not** support alpha channel in hex colours. Use only `#RRGGBB` (6 digits), not `#RRGGBBAA`.

### Specificity Rules (Critical)

4D CSS follows standard CSS specificity rules **with one critical addition**:

> **Properties defined directly in the `.4DForm` JSON have the highest specificity** and will override CSS unless `!important` is used.

Therefore, when moving a colour from a hardcoded form value to CSS:

1. **Remove** the property from the `.4DForm` JSON entirely
2. **Add** a `"class"` property to the object in the `.4DForm`
3. **Define** the colour in CSS for that class

If you leave the hardcoded value in the form, CSS will not override it (unless you use `!important`, which is discouraged).

### Assigning Classes

Add the `"class"` property to form objects in the `.4DForm` JSON:

```json
{
  "type": "text",
  "class": "my-label",
  "text": "Hello"
}
```

Multiple classes: `"class": "label primary"`

---

## Runtime Color Detection (Hidden Reference Objects)

### The Problem

4D has no API to directly query whether the current theme is dark or light in a boolean fashion. The commands:

- `Get application color scheme` (returns `"light"`, `"dark"`, or `"inherited"`)
- `FORM Get color scheme` (returns `"light"`, `"dark"`, or `"inherited"`)

…may return `"inherited"`, meaning the system theme is in use but you don't know which one it resolved to without additional logic.

### The Workaround

Use **hidden rectangle objects** on the form as colour references. Their rendered colours are set by CSS media queries, and at runtime you read the actual resolved colour with `OBJECT GET RGB COLORS`.

#### Step 1: Add Hidden Rectangles to the Form

Place them on page 0 (shared/always loaded) so they are available during `On Load`:

```json
"refMyColour": {
  "type": "rectangle",
  "top": 0,
  "left": 0,
  "width": 1,
  "height": 1,
  "class": "ref-my-colour",
  "stroke": "transparent"
}
```

#### Step 2: Define Colours in CSS

```css
@media (prefers-color-scheme: light) {
  .ref-my-colour {
	fill: #FFCCCC;
	visibility: hidden;
  }
}

@media (prefers-color-scheme: dark) {
  .ref-my-colour {
	fill: #5C2020;
	visibility: hidden;
  }
}
```

Note: `visibility: hidden` is set in **both** media queries to ensure the rectangle is always invisible regardless of scheme.

#### Step 3: Read at Runtime

```4d
var $fg; $bg : Integer
OBJECT GET RGB COLORS:C1074(*; "refMyColour"; $fg; $bg)
  // $bg contains the resolved fill colour as a longint (0x00RRGGBB)
```

#### Step 4: Convert to Hex String

Use a helper method to convert the longint to `#RRGGBB`:

```4d
  // RGBToHex method
#DECLARE($rgb : Integer)->$result : Text

var $r; $g; $b : Integer
$r:=($rgb >> 16) & 0x0000FF
$g:=($rgb >> 8) & 0x0000FF
$b:=$rgb & 0x0000FF

var $hex : Text
$hex:=""

var $i; $val : Integer
var $digits : Text
$digits:="0123456789ABCDEF"

For ($i; 1; 3)
  Case of 
	: ($i=1)
	  $val:=$r
	: ($i=2)
	  $val:=$g
	: ($i=3)
	  $val:=$b
  End case 
  $hex:=$hex+$digits[[$val\16+1]]+$digits[[$val%16+1]]
End for 

$result:="#"+$hex
```

### Use Case: Listbox Meta Expression

When a listbox uses `"metaSource"` to dynamically style rows/cells with fill colours, those colours are hardcoded strings. Use the hidden-rectangle technique to resolve theme-appropriate colours at form load time, then build the meta objects with those resolved values.

---

## 4D Method Token Reference

4D project mode source files may include token syntax (`:Cnnn` suffixes on commands). These tokens are **optional** — plain command names work correctly and 4D adds tokens automatically when it saves the file. **Never invent or guess token numbers**; an incorrect token silently resolves to the wrong command, causing hard-to-diagnose runtime errors. If unsure of a token, omit it entirely.

Known correct tokens (for reference only — omitting them is always safe):

| Command | Token | Notes |
|---------|-------|-------|
| `OBJECT GET RGB COLORS` | `:C1074` | Reads foreground/background of a named object |
| `New object` | `:C1471` | Create a new object |
| `Form` | `:C1466` | Access the form data object |

**Common mistake**: `:C382` is `_O_REDRAW LIST` (obsolete), not `OBJECT GET RGB COLORS`. This is exactly why guessing tokens is dangerous — always verify against existing project code or omit the token.

Command reference: https://developer.4d.com/docs/commands/

---

## Listbox-Specific Guidelines

### alternateFill

- Use `"automaticAlternate"` at the **listbox level** for automatic dark/light adaptation.
- Do **not** repeat the same `alternateFill` at the column level unless it is a deliberate column-specific override.
- Remove column-level `alternateFill` if it matches the listbox-level value (it adds no value and creates maintenance burden).

### Meta Source Colours

- Never hardcode light-mode-only colours in meta source methods.
- Use the hidden-rectangle reference technique described above.
- White text (`#FFFFFF`) on a coloured background (e.g., red `#FF4040`) is acceptable in both modes because the cell fill provides contrast.

---

## Colour Palette Recommendations

When choosing dark-mode equivalents, follow these principles:

| Light mode | Dark mode | Rationale |
|-----------|-----------|-----------|
| Near-white backgrounds (`#FFFFFF`, `#F8FCFF`) | Dark grey (`#1E1E1E`, `#2A2A2A`) | Sufficient contrast without pure black |
| Near-black text (`#212121`, `#000000`) | Light grey (`#E0E0E0`) | Readable on dark backgrounds |
| Muted text (`#696969`) | Lighter muted (`#A0A0A0`) | Maintains hierarchy without being invisible |
| Link blue (`#1E90FF`) | Lighter blue (`#5CB8FF`) | Accessible contrast on dark |
| Alert red fill (`#FF4040`) | Muted red (`#CC3333`) | Less harsh on dark backgrounds |
| Light pink fill (`#FFCCCC`) | Dark red (`#5C2020`) | Retains semantic meaning |
| Light green fill (`#B8EDB8`) | Dark green (`#1E4D1E`) | Retains semantic meaning |
| Panel grey (`#C0C0C0`) | Dark panel (`#3A3A3A`) | Structural differentiation |

---

## Checklist for Dark Mode Migration

1. **Scan forms** for hardcoded `stroke` and `fill` colours on text and shape objects.
2. **Scan forms** for objects that **omit** `fill` or `stroke` entirely — these use an internal default, not `"automatic"`. Add `"fill": "automatic"` or `"stroke": "automatic"` explicitly so they adapt to dark mode. Pay special attention to full-form background rectangles.
3. **Replace** `#000000`/`#FFFFFF` with `"automatic"` where appropriate.
3. **Replace** hardcoded `alternateFill` with `"automaticAlternate"`.
4. **Remove** column-level properties that duplicate listbox-level values.
5. **Move** branded/specific colours from `.4DForm` to CSS classes with media queries.
6. **Add** hidden reference rectangles for any runtime colour logic (meta expressions, programmatic styling).
7. **Create** `styleSheets.css` if it doesn't exist; define both `light` and `dark` media query blocks.
8. **Verify** no inline `.4DForm` property is overriding your CSS (specificity rule).
9. **Test** by toggling system appearance in System Preferences / Settings.

---

## References

- Color scheme setting: https://developer.4d.com/docs/settings/interface#color-scheme
- `Get application color scheme`: https://developer.4d.com/docs/commands/get-application-color-scheme
- CSS in 4D: https://developer.4d.com/docs/FormEditor/stylesheets
- Form CSS property: https://developer.4d.com/docs/FormEditor/propertiesForm#css
- Properties reference (camelCase names): https://developer.4d.com/docs/FormObjects/propertiesReference
