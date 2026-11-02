---
name: sketchware-events-manager
description: Expert skill for declaring, binding, and routing Sketchware Pro events, activity lifecycles, view interactions, and component callbacks into execution sections.
---

# Sketchware Events Manager Skill

## Overview
This skill specializes in the event architecture of Sketchware Pro projects. It defines and binds all user interactions (clicks, text changes, touch gestures), activity lifecycle hooks (`onCreate`, `onBackPressed`), and component callbacks (network responses, database listeners, animation completions) into the project's logic registry.

---

## 1. Event Registration Schema (`_events`)

All events for an activity are registered in the `@<ActivityJavaName>.java_events` section in `data/logic`. Each event is serialized as a JSON object:

```json
{
  "eventName": "onClick",
  "eventType": 1,
  "targetId": "button_submit",
  "targetType": 3
}
```

### Field Definitions
- **`eventName`** (`String`): The callback identifier (e.g. `"onClick"`, `"initializeLogic"`, `"onResponse"`).
- **`eventType`** (`int`): The category of event:
  - `1`: View Event (`EVENT_TYPE_VIEW`)
  - `2`: Component Event (`EVENT_TYPE_COMPONENT`)
  - `3`: Activity Lifecycle Event (`EVENT_TYPE_ACTIVITY`)
  - `4`: Drawer View Event (`EVENT_TYPE_DRAWER_VIEW`)
  - `5`: Custom / Etc Event (`EVENT_TYPE_ETC`)
- **`targetId`** (`String`): The ID of the originating entity (widget ID, component ID, or lifecycle hook name).
- **`targetType`** (`int`):
  - For Views: The `ViewBean.VIEW_TYPE_*` integer (e.g. Button = `3`, EditText = `5`, FAB = `16`).
  - For Components: The `ComponentBean.COMPONENT_TYPE_*` integer (e.g. RequestNetwork = `17`, Timer = `5`).
  - For Activities: `-1` or `0`.

---

## 2. Event Section Routing Rule (CRITICAL)

When an event is registered, its corresponding visual logic blocks MUST be stored under a matching section header in `data/logic`:

$$\text{Section Header} = \text{@} + \text{ActivityJavaName} + \text{_} + \text{targetId} + \text{_} + \text{eventName}$$

### Examples:
| Event Description | `targetId` | `eventName` | Section Header in `data/logic` |
|---|---|---|---|
| Activity onCreate | `"onCreate"` | `"initializeLogic"` | `@MainActivity.java_onCreate_initializeLogic` |
| Activity Back Pressed | `"onBackPressed"` | `"onBackPressed"` | `@MainActivity.java_onBackPressed_onBackPressed` |
| Button Click | `"button1"` | `"onClick"` | `@MainActivity.java_button1_onClick` |
| FAB Click | `"_fab"` | `"onClick"` | `@MainActivity.java__fab_onClick` |
| EditText Text Changed | `"edittext1"` | `"onTextChanged"` | `@MainActivity.java_edittext1_onTextChanged` |
| RequestNetwork Success | `"net"` | `"onResponse"` | `@MainActivity.java_net_onResponse` |
| Firebase Child Added | `"fdb"` | `"onChildAdded"` | `@MainActivity.java_fdb_onChildAdded` |

---

## 3. Comprehensive Event Directory

### A. Activity Lifecycle Events (`eventType: 3`, `targetType: -1`)

| Event Name | `targetId` | `eventName` | Triggered When |
|---|---|---|---|
| **Activity onCreate** | `"onCreate"` | `"initializeLogic"` | App activity starts and views are initialized |
| **Back Button Pressed**| `"onBackPressed"`| `"onBackPressed"` | User presses device or gesture back button |
| **Activity onStart** | `"onStart"` | `"onStart"` | Activity becomes visible |
| **Activity onResume** | `"onResume"` | `"onResume"` | Activity enters the foreground and accepts focus |
| **Activity onPause** | `"onPause"` | `"onPause"` | Another activity comes to foreground |
| **Activity onStop** | `"onStop"` | `"onStop"` | Activity is no longer visible |
| **Activity onDestroy**| `"onDestroy"` | `"onDestroy"` | Activity is finishing or killed by system |
| **Activity onPostCreate**| `"onPostCreate"`| `"onPostCreate"` | System initialization completes |

### B. View Interaction Events (`eventType: 1`)

| Widget Type | `targetType` | Available `eventName` | Trigger / Purpose |
|---|---|---|---|
| `Button` | `3` | `"onClick"`, `"onLongClick"` | User taps or holds button |
| `TextView` | `4` | `"onClick"`, `"onLongClick"` | User taps label (if clickable) |
| `EditText` | `5` | `"onTextChanged"` | User types or deletes text in field |
| `ImageView` | `6` | `"onClick"` | User taps image |
| `CheckBox` | `11` | `"onCheckedChange"` | Check status is toggled |
| `Switch` | `13` | `"onCheckedChange"` | Switch is turned ON/OFF |
| `SeekBar` | `14` | `"onProgressChanged"`, `"onStartTrackingTouch"`, `"onStopTrackingTouch"` | Slider dragged or released |
| `ListView` | `9` | `"onItemClicked"`, `"onItemLongClicked"` | List row clicked or held |
| `Spinner` | `10` | `"onItemSelected"` | Dropdown item chosen |
| `WebView` | `7` | `"onPageStarted"`, `"onPageFinished"` | Web page loading lifecycle |
| `CalendarView`| `15` | `"onDateChange"` | User selects calendar date |
| `FAB` | `16` | `"onClick"` | Floating Action Button tapped |

### C. Component Callback Events (`eventType: 2`)

| Component Type | `targetType` | Available `eventName` | Trigger / Purpose |
|---|---|---|---|
| **RequestNetwork** | `17` | `"onResponse"`, `"onErrorResponse"` | HTTP request succeeds or fails |
| **FirebaseDB** | `6` | `"onChildAdded"`, `"onChildChanged"`, `"onChildRemoved"`, `"onCancelled"` | Realtime database CRUD updates |
| **FirebaseAuth** | `12` | `"onCreateUserComplete"`, `"onSignInUserComplete"`, `"onResetPasswordEmailSent"` | Auth operations finish |
| **ObjectAnimator** | `10` | `"onAnimationStart"`, `"onAnimationEnd"`, `"onAnimationCancel"` | Property animation stages |
| **FirebaseStorage**| `14` | `"onUploadProgress"`, `"onDownloadProgress"`, `"onUploadSuccess"`, `"onFailure"` | File upload/download progress |
| **Camera** | `15` | `"onPictureTaken"`, `"onPictureTakenCancel"` | Image captured by camera |
| **FilePicker** | `16` | `"onFilesPicked"`, `"onFilesPickedCancel"` | User selects files from storage |
| **SpeechToText** | `19` | `"onSpeechResult"`, `"onSpeechError"` | Voice input recognized |
| **BluetoothConnect**|`20` | `"onConnected"`, `"onDataReceived"`, `"onConnectionError"` | Serial BT socket activity |
| **LocationManager**| `21` | `"onLocationChanged"` | GPS / Network location update |

---

## 4. Multi-Screen Events Management

For applications with multiple screens (e.g. `MainActivity.java`, `DetailActivity.java`):
1. In `data/file` (`@activity`), both activities are registered.
2. In `data/logic`:
   - `@MainActivity.java_events` lists events for `MainActivity`.
   - `@DetailActivity.java_events` lists events for `DetailActivity`.
   - Each event blocks section must be prefixed by its own activity name:
     - `@MainActivity.java_onCreate_initializeLogic`
     - `@DetailActivity.java_onCreate_initializeLogic`

---

## 5. Inter-Skill Integration Protocol
- **With `sketchware-layout-designer`:** Read all widget IDs and types created in the layout to register the corresponding view events.
- **With `sketchware-components-manager`:** Read all component IDs and types created to register component callback events.
- **With `sketchware-blocks-engine`:** Provide the exact section headers where the blocks engine will place the executable block chains.
- **With `sketchware-project-generator`:** Write the finalized event list and section blocks into `data/logic`.
