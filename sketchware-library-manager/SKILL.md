---
name: sketchware-library-manager
description: Expert skill for configuring, enabling, and managing Sketchware Pro libraries, AppCompat, Firebase, AdMob, Google Maps, Material3, and build permissions.
---

# Sketchware Library Manager Skill

## Overview
This skill specializes in project dependencies, external SDKs, and build settings for Sketchware Pro. It configures the `data/library` archive file, enables AndroidX/AppCompat, injects Firebase credentials, sets up AdMob unit IDs, configures Google Maps API keys, and manages manifest permissions.

---

## 1. File Structure in Backup Archive (`data/library`)

The `data/library` file is a single AES-encrypted text file structured into sections headed by `@<library_name>`:

```text
@firebaseDB
{"adUnits":[],"appId":"","configurations":{},"data":"","libType":0,"reserved1":"","reserved2":"","reserved3":"","testDevices":[],"useYn":"N"}

@compat
{"adUnits":[],"appId":"","configurations":{},"data":"","libType":1,"reserved1":"","reserved2":"","reserved3":"","testDevices":[],"useYn":"Y"}

@admob
{"adUnits":[],"appId":"","configurations":{},"data":"","libType":2,"reserved1":"","reserved2":"","reserved3":"","testDevices":[],"useYn":"N"}

@googleMap
{"adUnits":[],"appId":"","configurations":{},"data":"","libType":3,"reserved1":"","reserved2":"","reserved3":"","testDevices":[],"useYn":"N"}
```

---

## 2. ProjectLibraryBean JSON Schema

```json
{
  "libType": 1,
  "useYn": "Y",
  "appId": "",
  "data": "",
  "reserved1": "",
  "reserved2": "",
  "reserved3": "",
  "adUnits": [],
  "testDevices": [],
  "configurations": {}
}
```

### Field Definitions
- **`libType`** (`int`):
  - `0`: Firebase (`PROJECT_LIB_TYPE_FIREBASE`)
  - `1`: AppCompat / AndroidX (`PROJECT_LIB_TYPE_COMPAT`)
  - `2`: AdMob (`PROJECT_LIB_TYPE_ADMOB`)
  - `3`: Google Maps (`PROJECT_LIB_TYPE_GOOGLE_MAP`)
- **`useYn`** (`String`): `"Y"` if the library is enabled, `"N"` if disabled.
- **`appId`** (`String`): AdMob App ID or Firebase App ID.
- **`data`** (`String`): API key or primary token (e.g. Google Maps API key, Firebase Storage bucket).
- **`reserved1`**, **`reserved2`**, **`reserved3`** (`String`):
  - For Firebase:
    - `reserved1`: Firebase Project ID
    - `reserved2`: Firebase Web API Key
    - `reserved3`: Firebase Database URL (e.g. `https://my-app-default-rtdb.firebaseio.com`)
  - For AdMob:
    - `reserved1`: AdMob App ID
- **`adUnits`** (`List<AdUnitBean>`): Array of registered AdMob ad units:
  ```json
  [{"id":"banner1","unitId":"ca-app-pub-3940256099942544/6300978111"}]
  ```
- **`testDevices`** (`List<AdTestDeviceBean>`): Array of test device test IDs.

---

## 3. Library Dependency & Compatibility Rules (CRITICAL)

1. **AppCompat / AndroidX (`libType: 1`):**
   - **MANDATORY FOR MODERN APPS:** In Sketchware Pro, always set `useYn: "Y"` for `@compat`.
   - Without AppCompat enabled, modern Material widgets (CardView, TabLayout, MaterialButton, Toolbar, CoordinatorLayout) fail to compile.
2. **Firebase Dependency Rule:**
   - If `@firebaseDB` has `useYn: "Y"`, `@compat` MUST ALSO be `useYn: "Y"`.
3. **AdMob Dependency Rule:**
   - If `@admob` has `useYn: "Y"`, valid `appId` and at least one `adUnit` entry should be declared.
4. **Google Maps Rule:**
   - If `@googleMap` has `useYn: "Y"`, the `data` field must contain the Google Maps Android API V2 key.

---

## 4. Permissions Management (`data/permission`)

When specialized components or libraries are used, corresponding Android permissions must be declared in the project's permission list.
In `.swb` projects, permissions are stored in `data/permission` as a line-delimited list of permission strings:

### Common Permissions by Feature
- **Internet & Networking (`RequestNetwork`, `WebView`, `Firebase`):**
  - `android.permission.INTERNET`
  - `android.permission.ACCESS_NETWORK_STATE`
- **Storage & Files (`FilePicker`, `saveFile`):**
  - `android.permission.READ_EXTERNAL_STORAGE`
  - `android.permission.WRITE_EXTERNAL_STORAGE`
- **Camera (`Camera` component):**
  - `android.permission.CAMERA`
- **Location (`LocationManager`, `MapView`):**
  - `android.permission.ACCESS_FINE_LOCATION`
  - `android.permission.ACCESS_COARSE_LOCATION`
- **Haptic / Hardware (`Vibrator`, `BluetoothConnect`):**
  - `android.permission.VIBRATE`
  - `android.permission.BLUETOOTH`
  - `android.permission.BLUETOOTH_CONNECT`

---

## 5. Inter-Skill Integration Protocol
- **With `sketchware-components-manager`:** Whenever Firebase (`6`, `12`, `14`), AdMob (`13`, `22`), or MapView is added, this skill activates the corresponding library flags and configures API credentials.
- **With `sketchware-layout-designer`:** Enables support for Material and custom container widgets through AppCompat.
- **With `sketchware-project-generator`:** Formats the complete `data/library` content and coordinates encryption.
