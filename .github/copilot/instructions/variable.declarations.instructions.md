---
description: "Rules for migrating deprecated C_* variable declarations to modern var/#DECLARE syntax in 4D projects, including type mapping, return values, compiler methods, and common pitfalls"
---

# 4D Variable Declaration Modernisation — Agent Rules

## Overview

Starting with 4D 20 R7 (`compatibilityVersion >= 2070`), the `C_*` family of variable declaration commands (`C_LONGINT`, `C_TEXT`, `C_OBJECT`, `C_REAL`, `C_BOOLEAN`, `C_POINTER`, `C_BLOB`, `C_DATE`, `C_TIME`, `C_PICTURE`, `C_VARIANT`) is deprecated. They must be replaced with the modern `var` keyword (for local/process/interprocess variables) and `#DECLARE` (for method parameters and return values).

---

## Determining Whether Migration Is Required

Read the `compatibilityVersion` property from `Project/{ProjectName}.4DProject`:

```json
{ "compatibilityVersion": 2130 }
```

### Version Decoding

| Value pattern | Meaning | Example |
|---------------|---------|---------|
| `XXYY` where `YY < A0` | Version XX, revision YY | `2070` → 20 R7, `2130` → 21.3 |
| `XXAY` | Version XX, R(10+Y) | `20A0` → 20 R10 |

If `compatibilityVersion >= 2070`, all `C_*` declarations must be migrated.

---

## Scanning for Deprecated Declarations

Search all `.4dm` files for lines starting with a `C_` command:

```
C_LONGINT, C_TEXT, C_REAL, C_OBJECT, C_BOOLEAN,
C_POINTER, C_BLOB, C_DATE, C_TIME, C_PICTURE, C_VARIANT
```

These appear in three contexts:

| Context | File location | Example |
|---------|---------------|---------|
| Local variable declarations | Any `.4dm` | `C_LONGINT:C283($x; $y)` |
| Method parameter declarations | Project method `.4dm` | `C_LONGINT:C283($1)` or `C_OBJECT:C1216($1)` |
| Compiler variable/method declarations | `Compiler_Variables.4dm`, `Compiler_Methods.4dm` | `C_OBJECT:C1216(Address_EN; $0)` |

---

## Type Mapping

| Legacy Command | Token | Modern Type | Notes |
|---------------|-------|-------------|-------|
| `C_LONGINT` | `:C283` | `Integer` | 4D `Integer` is 64-bit since v20; replaces both `C_LONGINT` (32-bit) and `C_REAL` for whole numbers |
| `C_REAL` | `:C285` | `Real` | Floating-point numbers |
| `C_TEXT` | `:C284` | `Text` | |
| `C_OBJECT` | `:C1216` | `Object` | |
| `C_BOOLEAN` | `:C305` | `Boolean` | |
| `C_POINTER` | `:C301` | `Pointer` | |
| `C_BLOB` | `:C286` | `Blob` | |
| `C_DATE` | `:C307` | `Date` | |
| `C_TIME` | `:C306` | `Time` | |
| `C_PICTURE` | `:C286` | `Picture` | |
| `C_VARIANT` | `:C1683` | `Variant` | |
| `C_COLLECTION` | `:C1488` | `Collection` | |

---

## Migration Rules

### Rule 1: Local Variables — `C_*` → `var`

**Before:**
```4d
C_LONGINT:C283($x; $y)
C_TEXT:C284($name)
C_OBJECT:C1216($options)
```

**After:**
```4d
var $x; $y : Integer
var $name : Text
var $options : Object
```

**Key points:**
- Multiple variables of the same type can share one `var` declaration, separated by semicolons.
- The `var` keyword does **not** use command tokens (it is a language keyword, not a command).
- The `#DECLARE` keyword also does **not** use command tokens.

### Rule 2: Method Parameters — `C_*($1; $2; ...)` → `#DECLARE`

**Before:**
```4d
C_LONGINT:C283($1)
C_TEXT:C284($2)
```

**After:**
```4d
#DECLARE($param1 : Integer; $param2 : Text)
```

**Key points:**
- `#DECLARE` must be the first non-comment, non-attribute line in the method — this includes coming before **any** executable statement, not just before other declarations. If the original `C_*($N)` line was preceded by executable code (e.g. `SET MENU BAR:C67(1)` before `C_TEXT:C284($1)`), the `#DECLARE` line must be moved above that code when migrating, not simply substituted in place.
- **Numbered parameters (`$1`, `$2`, `$0`) are illegal in `#DECLARE`** — you must assign a named local variable (e.g., `$param1`, `$flag`).
- `#DECLARE` replaces **all** `C_*($N)` declarations for that method.
- For variadic methods, use `#DECLARE(... : Type)` with `Count parameters` and `${N}` notation, or `Copy parameters` to forward arguments to another method.

**⚠️ Common ordering mistake:** Migrating in-place (replacing the `C_*($N)` line with `#DECLARE` at the same line position) is wrong if any executable statement appears earlier in the method. Always re-check the resulting file so that `#DECLARE` (and any `var` lines that follow it) sit directly under the `//%attributes` line, before all other code:

```4d
//%attributes = {}
SET MENU BAR:C67(1)      // ❌ WRONG — executable code before C_* declarations
C_TEXT:C284($1)
```

```4d
//%attributes = {}
#DECLARE($title : Text)  // ✅ CORRECT — #DECLARE moved above the executable code
var $window : Integer

SET MENU BAR:C67(1)
```

### Rule 3: Return Values — `C_*($0)` → `#DECLARE` return syntax

The legacy pattern uses `$0` as a special "return" variable:

**Before:**
```4d
C_OBJECT:C1216($result; $0)
$result:=New object:C1471
$result.name:="test"
$0:=$result
```

**After:**
```4d
#DECLARE->$result : Object
$result:=New object:C1471
$result.name:="test"
```

**Key points:**
- The `->` arrow syntax in `#DECLARE` declares the return variable.
- The named return variable (e.g., `$result`) is automatically returned; no explicit `$0:=` assignment is needed.
- If the method also takes parameters: `#DECLARE($param : Text)->$result : Object`

### Rule 4: Process/Interprocess Variables

Process variables (no `$` prefix) and interprocess variables (`<>` prefix) use the same `var` syntax:

**Before:**
```4d
C_OBJECT:C1216(MyProcessVar)
C_LONGINT:C283(<>MyInterprocessVar)
```

**After:**
```4d
var MyProcessVar : Object
var <>MyInterprocessVar : Integer
```

### Rule 5: Compiler Methods Cleanup

4D projects often have `Compiler_Variables.4dm` and `Compiler_Methods.4dm` that pre-declare all process variables and method signatures for the compiler. When migrating:

- **`Compiler_Variables.4dm`**: Convert all `C_*` lines to `var` declarations. This file remains necessary — it declares process-scope variables used across methods.
- **`Compiler_Methods.4dm`**: Remove entries for any method that now uses `#DECLARE`, since `#DECLARE` provides the same type information to the compiler. If all methods use `#DECLARE`, this file can become empty (but keep it with just the attributes line for consistency).

**Before (`Compiler_Methods.4dm`):**
```4d
//%attributes = {"invisible":true}
C_LONGINT:C283(MyMethod; $1)
C_OBJECT:C1216(MyMethod; $0)
```

**After (if MyMethod uses `#DECLARE`):**
```4d
//%attributes = {"invisible":true}
  // MyMethod now uses #DECLARE
```

---

## Common Pitfalls

### ❌ Passing non-variables to `C_*` commands

```4d
C_OBJECT:C1216($lang; 0)
```

**Why this is wrong:** `C_*` commands accept one or more **variable names** to declare. `0` is an integer literal, not a variable. This was likely a typo for `$0` (the return variable). This is a **syntax error** that may cause unpredictable behaviour depending on the 4D version.

**Detection:** Search for `C_*` calls where any argument is not a valid variable name (i.e., does not start with `$`, `<>`, or a letter).

**Fix:** Identify the intended variable (usually `$0` when found at the end of the argument list), then migrate to `#DECLARE` return syntax.

### ❌ Mixing `#DECLARE` with `C_*` parameter declarations

```4d
#DECLARE($param : Text)
C_LONGINT:C283($1)   // WRONG — conflicts with #DECLARE
```

**Why:** `#DECLARE` replaces all `C_*` parameter declarations. Having both creates a conflict. Remove all `C_*($N)` lines when `#DECLARE` is present.

### ❌ Using numbered parameters in `#DECLARE`

```4d
#DECLARE($1 : Integer)   // WRONG — $1 is illegal in #DECLARE
```

**Why:** `#DECLARE` requires named local variables, not numbered parameters (`$1`, `$2`, `$0`). Numbered parameters are a legacy concept from `C_*` declarations. Use a descriptive name instead:

```4d
#DECLARE($flag : Integer)   // CORRECT
```

For variadic methods where you need numbered access, use the spread syntax:

```4d
#DECLARE(... : Real) : Real
var $i; $total : Real
For ($i; 1; Count parameters:C259)
    $total+=${$i}
End for
return $total
```

Or use `Copy parameters` to forward arguments to another method.

### ❌ Forgetting to remove `$0:=` assignments

```4d
#DECLARE->$result : Object
$result:=New object:C1471
$0:=$result   // WRONG — unnecessary and creates a compiler warning
```

**Why:** The `#DECLARE` return variable is automatically returned. Remove the `$0:=` line.

### ❌ Leaving redundant `Compiler_Methods` entries

After adding `#DECLARE` to a method, forgetting to remove the corresponding `C_*(MethodName; $0)` or `C_*(MethodName; $1)` entries from `Compiler_Methods.4dm`. These are now redundant and may cause compiler warnings about duplicate declarations.

### ❌ Using `Integer` where `Real` is needed

`C_LONGINT` maps to `Integer`, but if the variable is used in floating-point arithmetic or assigned decimal values, use `Real` instead. Check the variable's usage context.

Similarly, `C_REAL` used for a variable that only holds whole numbers (e.g., a button value from a form object) should still be typed as `Real` if that is what the form object produces — 4D button values are `Real` by default.

---

## Audit Procedure

### Step 1: Check project version

Read `compatibilityVersion` from `Project/{ProjectName}.4DProject`. If `>= 2070`, proceed with migration.

### Step 2: Find all `C_*` declarations

Search all `.4dm` files:
```
grep -rn "^C_(LONGINT|TEXT|REAL|OBJECT|BOOLEAN|POINTER|BLOB|DATE|TIME|PICTURE|VARIANT|COLLECTION)" Project/Sources/
```

### Step 3: Categorise each occurrence

| Category | Pattern | Action |
|----------|---------|--------|
| Local variables | `C_*($varName)` | → `var $varName : Type` |
| Parameters | `C_*($1)`, `C_*($2)` | → `#DECLARE($name : Type)` |
| Return value | `C_*($0)` or `C_*(MethodName; $0)` | → `#DECLARE->$name : Type` |
| Process variables | `C_*(VarName)` (no `$`) | → `var VarName : Type` |
| Compiler method entries | `C_*(MethodName; $N)` | → Remove if method has `#DECLARE` |

### Step 4: Apply changes

For each method:
1. Replace parameter `C_*($N)` declarations with a single `#DECLARE` line.
2. If `$0` was declared, add return syntax `->$name : Type` to `#DECLARE`.
3. Replace local `C_*($var)` declarations with `var $var : Type`.
4. Remove `$0:=` assignment lines (the return variable is implicit).
5. Update `Compiler_Methods.4dm` — remove entries for migrated methods.
6. Update `Compiler_Variables.4dm` — convert `C_*` to `var` for process variables.

### Step 5: Verify

Confirm no `C_*` declarations remain:
```
grep -rn "^C_(LONGINT|TEXT|REAL|OBJECT|BOOLEAN|POINTER|BLOB|DATE|TIME|PICTURE|VARIANT|COLLECTION)" Project/Sources/
```

---

## Checklist

- [ ] `compatibilityVersion` confirmed `>= 2070` in `.4DProject`
- [ ] All `C_*` local variable declarations replaced with `var`
- [ ] All `C_*($N)` parameter declarations replaced with `#DECLARE`
- [ ] `#DECLARE` placed above **all** executable statements in the method, not just in the position of the old `C_*` line
- [ ] All `C_*($0)` return declarations converted to `#DECLARE->$name : Type`
- [ ] All `$0:=` assignment lines removed where `#DECLARE` return syntax is used
- [ ] `Compiler_Methods.4dm` cleaned of entries for methods now using `#DECLARE`
- [ ] `Compiler_Variables.4dm` converted from `C_*` to `var` declarations
- [ ] No non-variable arguments passed to any remaining `C_*` calls (e.g., `0` instead of `$0`)
- [ ] Type mapping is correct (`C_LONGINT` → `Integer`, `C_REAL` → `Real`, etc.)
- [ ] No mixing of `#DECLARE` and `C_*($N)` in the same method
- [ ] `var` and `#DECLARE` do **not** include command tokens (they are keywords, not commands)

---

## References

- Modern variable declarations: https://developer.4d.com/docs/Concepts/variables
- Legacy C_* directives (pre-20 R7): https://developer.4d.com/docs/20/Concepts/variables#using-a-c_-directive
- `#DECLARE` for parameters: https://developer.4d.com/docs/Concepts/parameters#declaring-parameters
- Compiler methods: https://developer.4d.com/docs/Concepts/compiler#compiler-methods
- Simplified command names (20 R7+): https://blog.4d.com/simplified-commands-for-cleaner-codebase/
