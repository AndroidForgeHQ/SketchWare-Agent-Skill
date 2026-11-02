---
name: sketchware-blocks-engine
description: Expert skill for creating, connecting, and formatting Sketchware Pro visual logic blocks, variables, lists, control flow chains, and direct code injections.
---

# Sketchware Blocks Engine Skill

## Overview
This skill governs the core logic generation of Sketchware Pro. It handles block creation, chaining, parameter embedding, variable declarations, list management, control structures (`if`/`else`/`loop`), and MoreBlocks functions.

---

## 1. BlockBean JSON Schema

Every visual block in Sketchware is serialized as a single JSON line:

```json
{
  "id": "10",
  "opCode": "setText",
  "type": " ",
  "typeName": "",
  "spec": "%m.textview setText %s",
  "color": -16354097,
  "parameters": ["textview1", "@11"],
  "nextBlock": 12,
  "subStack1": -1,
  "subStack2": -1
}
```

### Field Definitions
- **`id`** (`String`): Unique numeric string ID within the event scope (e.g. `"10"`, `"11"`, `"12"`).
- **`opCode`** (`String`): The internal identifier for the operation.
- **`type`** (`String`): Return type character:
  - `" "` (space): Action block (statement with top/bottom notch)
  - `"s"`: String expression (rounded pill)
  - `"b"`: Boolean expression (hexagonal)
  - `"d"`: Number / Double expression (rounded pill)
  - `"v"`: Variable reference
  - `"c"`: Component reference
  - `"e"`: Event block header
- **`typeName`** (`String`): Type qualifier if required (usually `""`).
- **`spec`** (`String`): The visual label with parameter placeholders (`%s`, `%d`, `%b`, `%m.view`, etc.).
- **`color`** (`int`): ARGB 32-bit signed integer for the block category.
- **`parameters`** (`List<String>`): Array of parameter values:
  - Literal text/number: `"Hello"`, `"100"`, `"my_var"`
  - Nested block reference: `"@<block_id>"` (e.g. `"@11"`)
- **`nextBlock`** (`int`): The numeric `id` of the subsequent statement block, or `-1` if end of chain.
- **`subStack1`** (`int`): The starting block `id` inside an `if` or `repeat` body, or `-1` if empty.
- **`subStack2`** (`int`): The starting block `id` inside an `else` body, or `-1` if empty.

---

## 2. Block Chaining Mechanics

### A. Linear Statement Chain
Action blocks connect vertically using `nextBlock`:
```text
Block 10 (nextBlock: 11) -> Block 11 (nextBlock: 12) -> Block 12 (nextBlock: -1)
```

### B. Nested Expression Inputs
Blocks that return values (`"s"`, `"d"`, `"b"`) are embedded into action block parameter slots using `"@<id>"`. They have `"nextBlock": -1`:
```json
{"id":"10","opCode":"setText","type":" ","parameters":["textview1","@11"],"nextBlock":-1}
{"id":"11","opCode":"getText","type":"s","parameters":["edittext1"],"nextBlock":-1}
```

### C. Control Flow Sub-stacks
Control flow blocks branch into inner block sequences:
- `if` block: `subStack1` points to the first inside block.
- `ifElse` block: `subStack1` points to the `then` body, `subStack2` points to the `else` body.
- When an inner stack finishes, its last block has `nextBlock: -1`. The parent control block continues via its own `nextBlock`.

```json
{"id":"20","opCode":"ifElse","type":" ","parameters":["@21"],"nextBlock":25,"subStack1":22,"subStack2":24}
{"id":"21","opCode":"stringEquals","type":"b","parameters":["@26","admin"],"nextBlock":-1}
{"id":"22","opCode":"showMessage","type":" ","parameters":["Welcome Admin"],"nextBlock":-1}
{"id":"24","opCode":"showMessage","type":" ","parameters":["Access Denied"],"nextBlock":-1}
{"id":"25","opCode":"finishActivity","type":" ","parameters":[],"nextBlock":-1}
```

---

## 3. Variable & List Storage Syntax (CRITICAL)

In `data/logic`, variables and lists are declared per activity under dedicated section headers.
**Note:** They are stored as line-delimited `<typeInt>:<name>` strings, NOT JSON objects.

### A. Variables: `@<ActivityJavaName>.java_var`
Syntax: `<typeInt>:<variableName>`
- `1`: Boolean (`boolean <name>;`)
- `2`: Number / Double (`double <name>;`)
- `3`: String (`String <name>;`)
- `4`: Map (`HashMap<String, Object> <name>;`)

Example:
```text
@MainActivity.java_var
3:user_name
2:login_count
1:is_logged_in
4:user_profile
```

### B. Lists: `@<ActivityJavaName>.java_list`
Syntax: `<typeInt>:<listName>`
- `1`: List String (`ArrayList<String> <name>;`)
- `2`: List Number (`ArrayList<Double> <name>;`)
- `3`: List Map (`ArrayList<HashMap<String, Object>> <name>;`)

Example:
```text
@MainActivity.java_list
1:messages_list
3:products_map_list
```

---

## 4. Comprehensive Block Palette Catalog

### A. Variable Blocks (Color: `-559806` / Orange)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `setVarString` | `" "` | `set %m.varStr to %s` | `[varName, value]` |
| `setVarInt` | `" "` | `set %m.varInt to %d` | `[varName, value]` |
| `setVarBoolean` | `" "` | `set %m.varBool to %b` | `[varName, value]` |
| `getVar` | `"s"` / `"d"` / `"b"` | `%m.var` | `[varName]` |
| `increaseInt` | `" "` | `increase %m.varInt by %d` | `[varName, increment]` |
| `decreaseInt` | `" "` | `decrease %m.varInt by %d` | `[varName, decrement]` |

### B. List Blocks (Color: `-1418873` / Gold-Orange)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `addListStr` | `" "` | `add %s to %m.listStr` | `[item, listName]` |
| `insertListStr` | `" "` | `insert %s at %d to %m.listStr` | `[item, index, listName]` |
| `deleteListStr` | `" "` | `delete %d of %m.listStr` | `[index, listName]` |
| `clearList` | `" "` | `clear %m.list` | `[listName]` |
| `getAtListStr` | `"s"` | `get %d of %m.listStr` | `[index, listName]` |
| `lengthList` | `"d"` | `length of %m.list` | `[listName]` |
| `containListStr` | `"b"` | `%m.listStr contains %s` | `[listName, item]` |

### C. Control Flow Blocks (Color: `-1872124` / Yellow)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `if` | `" "` | `if %b then` | `[condition]` |
| `ifElse` | `" "` | `if %b then else` | `[condition]` |
| `repeat` | `" "` | `repeat %d` | `[count]` |
| `forever` | `" "` | `forever` | `[]` |
| `break` | `" "` | `break` | `[]` |
| `continue` | `" "` | `continue` | `[]` |

### D. Math & Operator Blocks (Color: `-14392470` / Green)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `+` | `"d"` | `%d + %d` | `[a, b]` |
| `-` | `"d"` | `%d - %d` | `[a, b]` |
| `*` | `"d"` | `%d * %d` | `[a, b]` |
| `/` | `"d"` | `%d / %d` | `[a, b]` |
| `%` | `"d"` | `%d % %d` | `[a, b]` |
| `&&` | `"b"` | `%b and %b` | `[a, b]` |
| `||` | `"b"` | `%b or %b` | `[a, b]` |
| `!` | `"b"` | `not %b` | `[a]` |
| `<` | `"b"` | `%d < %d` | `[a, b]` |
| `>` | `"b"` | `%d > %d` | `[a, b]` |
| `=` | `"b"` | `%d = %d` | `[a, b]` |
| `random` | `"d"` | `pick random %d to %d` | `[min, max]` |
| `mathPow` | `"d"` | `%d ^ %d` | `[base, exp]` |
| `mathSqrt` | `"d"` | `sqrt %d` | `[val]` |

### E. String Blocks (Color: `-14392470` / Green)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `stringJoin` | `"s"` | `join %s and %s` | `[str1, str2]` |
| `stringLength` | `"d"` | `length of %s` | `[str]` |
| `stringEquals` | `"b"` | `%s = %s` | `[str1, str2]` |
| `stringContains` | `"b"` | `%s contains %s` | `[str, query]` |
| `stringSub` | `"s"` | `substring %s from %d to %d` | `[str, start, end]` |
| `trim` | `"s"` | `trim %s` | `[str]` |
| `toUpperCase` | `"s"` | `to upper case %s` | `[str]` |
| `toLowerCase` | `"s"` | `to lower case %s` | `[str]` |
| `toString` | `"s"` | `to string %d without decimal` | `[num]` |
| `toNumber` | `"d"` | `to number %s` | `[str]` |
| `asdString` | `"s"` | `string %s.inputOnly` | `["javaExpression()"]` |

### F. View Blocks (Color: `-16354097` / Blue)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `setText` | `" "` | `%m.textview setText %s` | `[viewId, text]` |
| `getText` | `"s"` | `%m.textview getText` | `[viewId]` |
| `setVisible` | `" "` | `%m.view setVisible %m.visible` | `[viewId, "VISIBLE"|"INVISIBLE"|"GONE"]` |
| `setEnable` | `" "` | `%m.view setEnabled %b` | `[viewId, bool]` |
| `setBgColor` | `" "` | `%m.view setBackgroundColor %d` | `[viewId, colorInt]` |
| `setImage` | `" "` | `%m.imageview setImage %m.resource` | `[viewId, imageName]` |
| `setImageUrl` | `" "` | `%m.imageview set image from URL %s` | `[viewId, url]` |

### G. Raw Code Injections & MoreBlocks (Color: `-16729254` / Dark Cyan)
| opCode | Type | Spec | Parameters |
|---|---|---|---|
| `addSourceDirectly` | `" "` | `add source directly %s.inputOnly` | `["// Java code here"]` |
| `asdString` | `"s"` | `string %s.inputOnly` | `["// Java expression here"]` |

---

## 5. MoreBlocks (Functions) Authoring

MoreBlocks are defined in `@<ActivityJavaName>.java_func`:
Format: `<functionName>(<type1> <param1>, <type2> <param2>)`
Example:
```text
@MainActivity.java_func
calculateTotal(double price, double tax)
```
Its body blocks are stored in a dedicated section:
`@MainActivity.java_moreBlock_calculateTotal`
Call it from any other event using:
```json
{
  "id": "50",
  "opCode": "calculateTotal",
  "type": " ",
  "spec": "calculateTotal %d %d",
  "parameters": ["100", "15"],
  "nextBlock": -1
}
```

---

## 6. Inter-Skill Integration Protocol
- **With `sketchware-layout-designer`:** Reference UI IDs created by the layout designer in all `setText`, `getText`, `setVisible` view blocks.
- **With `sketchware-components-manager`:** Invoke component actions (e.g. `intentSetAction`, `fileWrite`, `timerAfter`) using the component IDs registered in `_components`.
- **With `sketchware-events-manager`:** Block chains are authored specifically for the target events defined in `_events`.
- **With `sketchware-project-generator`:** All blocks, variables, lists, and functions are formatted and written into `data/logic`.
