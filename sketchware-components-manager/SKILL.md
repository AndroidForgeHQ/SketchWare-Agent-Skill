---
name: sketchware-components-manager
description: Expert skill for creating, configuring, and connecting Sketchware Pro system components, Intents, Storage, Firebase, Networking, and Timers.
---

# Sketchware Components Manager Skill

## Overview
This skill specializes in Android system components and background services in Sketchware Pro. It manages component instances declared in `@<ActivityJavaName>.java_components`, configures component parameters (file names, database paths, mime types), and binds component-specific blocks and asynchronous callbacks.

---

## 1. Component Registration Schema (`_components`)

All non-UI components (Intents, Shared Preferences, Timers, Network clients, etc.) are registered under `@<ActivityJavaName>.java_components` in `data/logic`. Each component is serialized as a JSON object:

```json
{
  "componentId": "net",
  "type": 17,
  "param1": "",
  "param2": "",
  "param3": ""
}
```

### Field Definitions
- **`componentId`** (`String`): The unique variable name of the component (e.g. `"in"`, `"sp"`, `"timer"`, `"net"`).
- **`type`** (`int`): Integer identifier from `ComponentBean.COMPONENT_TYPE_*`.
- **`param1`**, **`param2`**, **`param3`** (`String`): Configuration parameters required by specific components:
  - For `SharedPreferences` (`type: 2`): `param1` is the preference file name (e.g. `"user_session"`).
  - For `FirebaseDB` (`type: 6`): `param1` is the database path (e.g. `"chat_messages"`).
  - For `FilePicker` (`type: 16`): `param1` is the MIME type filter (e.g. `"*/*"` or `"image/*"`).
  - For `FirebaseStorage` (`type: 14`): `param1` is the storage bucket path.

---

## 2. Complete Component Types Directory

| Type Int | Component Name | Java Class / Purpose | `param1` |
|---|---|---|---|
| `1` | `Intent` | `android.content.Intent` (Screen navigation, actions) | `""` |
| `2` | `SharedPreferences` | `android.content.SharedPreferences` (Key-value storage) | File name (e.g. `"data"`) |
| `3` | `Calendar` | `java.util.Calendar` (Date & time calculations) | `""` |
| `4` | `Vibrator` | `android.os.Vibrator` (Haptic feedback) | `""` |
| `5` | `Timer` | `java.util.Timer` & `TimerTask` (Scheduled execution) | `""` |
| `6` | `FirebaseDB` | Firebase Realtime Database reference | Path (e.g. `"users"`) |
| `7` | `Dialog` | `android.app.AlertDialog` (Popups & confirmations) | `""` |
| `8` | `MediaPlayer` | `android.media.MediaPlayer` (Audio playback) | `""` |
| `9` | `SoundPool` | `android.media.SoundPool` (Short sound effects) | `""` |
| `10` | `ObjectAnimator` | `android.animation.ObjectAnimator` (View animations) | `""` |
| `11` | `Gyroscope` | `android.hardware.Sensor` (Motion sensors) | `""` |
| `12` | `FirebaseAuth` | Firebase Authentication (Email/Password login) | `""` |
| `13` | `InterstitialAd` | Google AdMob Fullscreen Interstitial Ad | `""` |
| `14` | `FirebaseStorage`| Firebase Cloud Storage (File uploads/downloads) | Bucket/folder path |
| `15` | `Camera` | `android.hardware.Camera` / Hardware camera | `""` |
| `16` | `FilePicker` | Storage Access Framework / File selector | MIME type (e.g. `"*/*"`) |
| `17` | `RequestNetwork` | HTTP/HTTPS REST client (GET/POST/PUT/DELETE) | `""` |
| `18` | `TextToSpeech` | `android.speech.tts.TextToSpeech` (Speech synthesis) | `""` |
| `19` | `SpeechToText` | `android.speech.SpeechRecognizer` (Voice recognition)| `""` |
| `20` | `BluetoothConnect`| Bluetooth RFCOMM Serial Connection | `""` |
| `21` | `LocationManager`| GPS & Network location provider | `""` |
| `22` | `RewardedVideoAd`| Google AdMob Rewarded Video Ad | `""` |
| `23` | `ProgressDialog` | Modal spinner progress dialog | `""` |
| `24` | `DatePickerDialog`| Android date selector dialog | `""` |
| `25` | `TimePickerDialog`| Android time selector dialog | `""` |
| `26` | `Notification` | System status bar notification manager | `""` |
| `27` | `FragmentAdapter`| ViewPager FragmentPagerAdapter | `""` |
| `28` | `FirebaseAuthPhone`| Firebase SMS verification / Phone login | `""` |
| `30` | `FirebaseCloudMessage`| Push notification messaging receiver | `""` |
| `31` | `FirebaseAuthGoogleLogin`| Google One-Tap / Firebase Google Sign-In | `""` |

---

## 3. Standard Component Block Methods

### A. Intent (`type: 1`)
| opCode | Spec | Parameters |
|---|---|---|
| `intentSetAction` | `%m.intent setAction %m.action` | `[componentId, "ACTION_VIEW"|"ACTION_CALL"]` |
| `intentSetData` | `%m.intent setData %s` | `[componentId, uriString]` |
| `intentSetScreen` | `%m.intent setScreen %m.activity` | `[componentId, TargetActivityName]` |
| `intentPutExtra` | `%m.intent putExtra key %s value %s` | `[componentId, key, value]` |
| `startActivity` | `startActivity %m.intent` | `[componentId]` |

### B. SharedPreferences / File (`type: 2`)
| opCode | Spec | Parameters |
|---|---|---|
| `fileGetData` | `%m.file getData key %s` | `[componentId, key]` |
| `fileSetData` | `%m.file setData key %s value %s` | `[componentId, key, value]` |
| `fileRemoveData` | `%m.file removeData key %s` | `[componentId, key]` |

### C. Timer (`type: 5`)
| opCode | Spec | Parameters |
|---|---|---|
| `timerAfter` | `%m.timerTask after %d ms` | `[componentId, delayMs]` |
| `timerEvery` | `%m.timerTask after %d ms for every %d ms` | `[componentId, delayMs, intervalMs]` |
| `timerCancel` | `%m.timerTask cancel` | `[componentId]` |

### D. RequestNetwork (`type: 17`)
| opCode | Spec | Parameters |
|---|---|---|
| `netStartRequest` | `%m.network start request network %m.requestMethod to %s with tag %s` | `[componentId, "GET"|"POST", url, tag]` |
| `netSetHeaders` | `%m.network set request headers %m.map` | `[componentId, headersMap]` |
| `netSetParams` | `%m.network set request params %m.map` | `[componentId, paramsMap]` |

### E. Dialog (`type: 7`)
| opCode | Spec | Parameters |
|---|---|---|
| `dialogSetTitle` | `%m.dialog setTitle %s` | `[componentId, title]` |
| `dialogSetMessage`| `%m.dialog setMessage %s` | `[componentId, message]` |
| `dialogOkButton` | `%m.dialog OK Button %s` | `[componentId, buttonText]` |
| `dialogCancelButton`| `%m.dialog Cancel Button %s`| `[componentId, buttonText]` |
| `dialogShow` | `%m.dialog show` | `[componentId]` |

---

## 4. Inter-Skill Integration Protocol
- **With `sketchware-events-manager`:** Whenever asynchronous components (`RequestNetwork`, `FirebaseDB`, `FirebaseAuth`, `ObjectAnimator`, `SpeechToText`) are added, register their corresponding callback events in `@<Activity>.java_events`.
- **With `sketchware-blocks-engine`:** Supply component IDs to blocks manipulating intents, preferences, timers, and requests.
- **With `sketchware-library-manager`:** If Firebase components (`6`, `12`, `14`) or Google Ads (`13`, `22`) are used, notify the library manager to activate their respective libraries in `data/library`.
- **With `sketchware-project-generator`:** Write the component JSON objects into the `@<Activity>.java_components` section in `data/logic`.
