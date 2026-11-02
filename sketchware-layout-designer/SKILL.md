---
name: sketchware-layout-designer
description: Expert skill for designing, structuring, and generating Sketchware Pro UI layouts, XML view hierarchies, widgets, and FAB configurations for 1-click importable projects.
---

# Sketchware Layout Designer Skill

## Overview
This skill specializes in creating and managing the UI layer of Sketchware Pro projects. It produces the exact JSON specifications required for `data/view` in `.swb` backup packages and provides full support for widget hierarchy, styling, responsive layouts, and floating action buttons (FAB).

---

## 1. File Structure in Backup Archive
In a Sketchware project or `.swb` archive, all screen layouts are combined into a **single AES-encrypted file**: `data/view`.

### Multi-Screen & Section Headers
Layouts are divided by section markers:
- `@<activity_xml_name>.xml` (e.g. `@main.xml`, `@home.xml`)
- `@<activity_xml_name>.xml_fab` (for activities with a Floating Action Button)
- `@<custom_view_name>.xml` (for CustomViews used in ListViews/Spinners)

```text
@main.xml
<ViewBean JSON for view 1>
<ViewBean JSON for view 2>
<ViewBean JSON for view 3>

@main.xml_fab
<ViewBean JSON for FAB>

@item_row.xml
<ViewBean JSON for custom view items>
```

---

## 2. ViewBean JSON Schema & Required Fields

Every UI element is a single-line JSON string representing a `ViewBean`:

```json
{
  "id": "button_submit",
  "type": 3,
  "parent": "root",
  "parentType": 0,
  "index": 0,
  "convert": "Button",
  "customView": "",
  "enabled": 1,
  "clickable": 1,
  "alpha": 1.0,
  "translationX": 0.0,
  "translationY": 0.0,
  "scaleX": 1.0,
  "scaleY": 1.0,
  "firstDayOfWeek": 1,
  "dividerHeight": 1,
  "choiceMode": 0,
  "checked": 0,
  "indeterminate": "false",
  "progress": 0,
  "max": 100,
  "progressStyle": "?android:progressBarStyle",
  "spinnerMode": 1,
  "inject": "",
  "adSize": "",
  "adUnitId": "",
  "preIndex": 0,
  "preParentType": -1,
  "parentAttributes": {},
  "layout": {
    "width": -1,
    "height": -2,
    "orientation": -1,
    "gravity": 0,
    "layoutGravity": 0,
    "weight": 0,
    "weightSum": 0,
    "marginLeft": 16,
    "marginTop": 16,
    "marginRight": 16,
    "marginBottom": 16,
    "paddingLeft": 12,
    "paddingTop": 12,
    "paddingRight": 12,
    "paddingBottom": 12,
    "backgroundColor": 16777215,
    "borderColor": -16740915
  },
  "text": {
    "text": "Submit",
    "textSize": 16,
    "textColor": -16777216,
    "hint": "",
    "hintColor": 16777215,
    "singleLine": 0,
    "line": 0,
    "inputType": 1,
    "imeOption": 0,
    "textFont": "default_font",
    "textType": 1
  },
  "image": {
    "rotate": 0,
    "scaleType": "CENTER"
  }
}
```

---

## 3. Widget & Layout Type Constants

| Type Int | Widget / Layout Class | `convert` String | Description |
|---|---|---|---|
| `0` | `LinearLayout` | `LinearLayout` | Linear container (Horizontal or Vertical) |
| `1` | `RelativeLayout` | `RelativeLayout` | Relative container |
| `2` | `HorizontalScrollView` | `HorizontalScrollView` | Horizontal scrolling container |
| `3` | `Button` | `Button` | Interactive push button |
| `4` | `TextView` | `TextView` | Text display label |
| `5` | `EditText` | `EditText` | Text input field |
| `6` | `ImageView` | `ImageView` | Image container |
| `7` | `WebView` | `WebView` | In-app browser / web frame |
| `8` | `ProgressBar` | `ProgressBar` | Circular or horizontal loading indicator |
| `9` | `ListView` | `ListView` | Scrollable vertical item list |
| `10` | `Spinner` | `Spinner` | Dropdown selection list |
| `11` | `CheckBox` | `CheckBox` | Two-state checkable box |
| `12` | `ScrollView` | `ScrollView` | Vertical scrolling container |
| `13` | `Switch` | `Switch` | Two-state toggle switch |
| `14` | `SeekBar` | `SeekBar` | Draggable value slider |
| `15` | `CalendarView` | `CalendarView` | Interactive month calendar |
| `16` | `FloatingActionButton` | `FloatingActionButton` | Standard Material FAB (in `_fab` section) |
| `17` | `AdView` | `AdView` | Google AdMob banner |
| `18` | `MapView` | `MapView` | Google Maps container |

---

## 4. Hierarchy & Parent Rules (CRITICAL)

1. **Root-Level Views:**
   - For elements attached directly to the screen base, set `"parent": "root"`.
   - Set `"parentType": 0` (or `-1` for FAB).
2. **Child Views in Containers:**
   - Set `"parent": "<container_view_id>"` (e.g. `"parent": "linear_main"`).
   - Set `"parentType": <container_type_int>` (e.g. `0` for LinearLayout).
3. **Index Ordering:**
   - `"index"` MUST be `0, 1, 2, 3...` sequentially among siblings sharing the same `"parent"`.
4. **Layout Dimensions:**
   - `MATCH_PARENT`: `-1`
   - `WRAP_CONTENT`: `-2`
   - Exact DP: positive integer (e.g. `48` for 48dp).
5. **Linear Layout Orientation:**
   - Horizontal: `"orientation": 0`
   - Vertical: `"orientation": 1`
   - Non-container widgets: `"orientation": -1`

---

## 5. Styling, Colors & Typography

- **ARGB Hex to 32-bit Signed Int:**
  Sketchware stores colors as 32-bit signed integers:
  - Transparent / Default: `16777215` (`0x00FFFFFF`)
  - Black (`#FF000000`): `-16777216`
  - White (`#FFFFFFFF`): `-1`
  - Sketchware Primary Blue (`#FF008DCD`): `-16740915`
  - Material Accent Red (`#FFFF4081`): `-49023`
  - Material Grey (`#FF9E9E9E`): `-6381922`
- **Text Styles (`textType`):**
  - `0`: Normal
  - `1`: Bold
  - `2`: Italic
  - `3`: Bold Italic
- **Input Types (`inputType` for EditText):**
  - `1`: Normal Text
  - `2`: Text (No suggestions)
  - `33`: Email Address (`TYPE_TEXT_VARIATION_EMAIL_ADDRESS`)
  - `129`: Password (`TYPE_TEXT_VARIATION_PASSWORD`)
  - `2`: Number (`TYPE_CLASS_NUMBER`)
  - `3`: Phone (`TYPE_CLASS_PHONE`)
- **Scale Types (`scaleType` for ImageView):**
  - `"CENTER"`, `"CENTER_CROP"`, `"CENTER_INSIDE"`, `"FIT_CENTER"`, `"FIT_START"`, `"FIT_END"`, `"FIT_XY"`

---

## 6. Floating Action Button (FAB) Setup

If an Activity enables FAB:
1. In `data/file` (`@activity`), set `"options": 1` (or bitmask with FAB).
2. In `data/view`, add an entry under `@<activity>.xml_fab`:
```json
@main.xml_fab
{"adSize":"","adUnitId":"","alpha":1.0,"checked":0,"choiceMode":0,"clickable":1,"convert":"","customView":"","dividerHeight":1,"enabled":1,"firstDayOfWeek":1,"id":"_fab","image":{"rotate":0,"scaleType":"CENTER"},"indeterminate":"false","index":0,"inject":"","layout":{"backgroundColor":16777215,"borderColor":-16740915,"gravity":0,"height":-2,"layoutGravity":85,"marginBottom":16,"marginLeft":16,"marginRight":16,"marginTop":16,"orientation":-1,"paddingBottom":0,"paddingLeft":0,"paddingRight":0,"paddingTop":0,"weight":0,"weightSum":0,"width":-2},"max":100,"parentAttributes":{},"parentType":-1,"preIndex":0,"preParentType":0,"progress":0,"progressStyle":"?android:progressBarStyle","scaleX":1.0,"scaleY":1.0,"spinnerMode":1,"text":{"hint":"","hintColor":16777215,"imeOption":0,"inputType":1,"line":0,"singleLine":0,"text":"","textColor":16777215,"textFont":"default_font","textSize":12,"textType":0},"translationX":0.0,"translationY":0.0,"type":16}
```
3. In `data/logic`, the click event binds to target `_fab`:
   - Event header: `@MainActivity.java_events` -> `{"eventName":"onClick","eventType":1,"targetId":"_fab","targetType":16}`
   - Logic block header: `@MainActivity.java__fab_onClick`

---

## 7. Inter-Skill Integration Protocol

- **With `sketchware-events-manager`:** Every interactive view (`Button`, `EditText`, `ImageView`, `ListView`, `_fab`) created here is referenced in `data/logic` via its `id` and `targetType` for click, change, and item listeners.
- **With `sketchware-blocks-engine`:** View IDs are passed into UI block parameters (e.g. `setText`, `getText`, `setVisible`, `setImageUrl`).
- **With `sketchware-project-generator`:** Generated view JSON strings are combined under their respective `@<screen>.xml` headers and encrypted into `data/view`.
