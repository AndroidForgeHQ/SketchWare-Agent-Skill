---
name: sketchware-resources-manager
description: Expert skill for managing, injecting, and cataloging Sketchware Pro project resources, images, sounds, fonts, icons, and asset archives.
---

# Sketchware Resources Manager Skill

## Overview
This skill specializes in static asset management for Sketchware Pro. It handles the `data/resource` catalog file (`@images`, `@sounds`, `@fonts`), packages raw binary files into the appropriate `.swb` zip paths, manages app icons, enforces Android AAPT naming conventions, and ensures empty asset folders contain required `.nomedia` markers.

---

## 1. Resource Catalog File (`data/resource`)

The `data/resource` file is an AES-encrypted text file structured into three main sections:

```text
@images
{"resName":"icon_home","rotate":0,"scaleType":"CENTER"}
{"resName":"banner_main","rotate":0,"scaleType":"FIT_CENTER"}

@sounds
{"resName":"beep","rotate":0,"scaleType":"CENTER"}

@fonts
{"resName":"poppins_bold","rotate":0,"scaleType":"CENTER"}
```

*Note: Even when empty, the section markers MUST still be present in `data/resource`:*
```text
@images
@sounds
@fonts
```

---

## 2. ImageBean JSON Schema

```json
{
  "resName": "logo_app",
  "rotate": 0,
  "scaleType": "CENTER"
}
```

### Supported Scale Types:
- `"CENTER"`
- `"CENTER_CROP"`
- `"CENTER_INSIDE"`
- `"FIT_CENTER"`
- `"FIT_START"`
- `"FIT_END"`
- `"FIT_XY"`

---

## 3. Physical Archive Paths in `.swb` (ZIP)

In a packaged `.swb` (ZIP) archive, physical assets reside in the `resources/` folder:

| Asset Type | Storage Path in Archive | Supported File Formats |
|---|---|---|
| **Images** | `resources/images/<resName>.png` | `.png`, `.jpg`, `.jpeg`, `.webp` |
| **Sounds** | `resources/sounds/<resName>.mp3` | `.mp3`, `.wav`, `.ogg` |
| **Fonts** | `resources/fonts/<resName>.ttf` | `.ttf`, `.otf` |
| **Custom App Icon**| `resources/icons/<sc_id>/icon.png` | `.png` (512x512 recommended) |

---

## 4. The `.nomedia` File Rule (CRITICAL)

Android media scanner ignores folders with `.nomedia`. Sketchware Pro's archive extractor expects empty asset directories to contain a 0-byte `.nomedia` file to maintain the folder tree on disk:
- `resources/fonts/.nomedia`
- `resources/images/.nomedia`
- `resources/sounds/.nomedia`
- `resources/icons/.nomedia`

If an asset folder is empty, the `.nomedia` file MUST be included in the archive.

---

## 5. Android AAPT Naming Conventions

All resource names (`resName`) MUST adhere to strict Android resource naming rules:
1. **Lowercase Only:** `[a-z0-9_]` (e.g. `icon_user`, `sound_click_01`).
2. **No Spaces or Hyphens:** Never use `icon user` or `my-image`. Use underscores: `icon_user`.
3. **Cannot Start with a Digit:** `img_1` is valid; `1_img` will cause AAPT build errors.
4. **No Extension in `resName`:** Use `"resName": "logo"`, NOT `"resName": "logo.png"`.

---

## 6. App Icon Configuration

To set a custom application icon:
1. In project metadata (`project` file), set `"custom_icon": true`.
2. Place the icon binary at: `resources/icons/<sc_id>/icon.png`.
3. If `"custom_icon": false`, Sketchware will use its default Android robotic icon.

---

## 7. Inter-Skill Integration Protocol
- **With `sketchware-layout-designer`:** When an ImageView or Button specifies an image name, ensure it is registered in `@images` and physically packed.
- **With `sketchware-blocks-engine`:** Provide valid asset names for `setImage`, `soundPoolPlay`, and `setTypeface` blocks.
- **With `sketchware-project-generator`:** Write `data/resource` and package the binary asset files and `.nomedia` placeholders into the `.swb` zip.
