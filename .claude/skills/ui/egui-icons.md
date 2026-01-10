# egui-icons

Add professional icons to egui applications using the egui-phosphor library.

## Problem

egui doesn't support color emoji fonts - they display as squares (□) because egui's font parser only supports basic TrueType/OpenType outline fonts. It cannot parse:
- Color bitmap emoji fonts (CBDT/CBLC tables)
- Color vector emoji fonts (COLR/CPAL tables)
- SVG emoji fonts (SVG tables)

## Solution

Use **egui-phosphor**, which provides Phosphor Icons as monochrome outline fonts compatible with egui.

## Implementation

### 1. Add Dependency

Add to `Cargo.toml`:
```toml
[dependencies]
egui-phosphor = "0.7"
```

### 2. Initialize Font

In your `main.rs` or app initialization:
```rust
use eframe::egui;

eframe::run_native(
    "app_name",
    native_options,
    Box::new(|cc| {
        // Load Phosphor icon font
        let mut fonts = egui::FontDefinitions::default();
        egui_phosphor::add_to_fonts(&mut fonts, egui_phosphor::Variant::Regular);
        cc.egui_ctx.set_fonts(fonts);

        Ok(Box::new(YourApp::new(cc)))
    }),
)
```

### 3. Use Icons in UI

Import the icon constants:
```rust
use egui_phosphor::regular;
```

Use in buttons and labels:
```rust
// Buttons with icons
if ui.button(format!("{} Open File", regular::FOLDER)).clicked() { }
if ui.button(format!("{} Refresh", regular::ARROW_CLOCKWISE)).clicked() { }
if ui.button(format!("{} Settings", regular::GEAR)).clicked() { }

// Labels with icons
ui.label(format!("{} Status: Ready", regular::CHECK_CIRCLE));
ui.colored_label(egui::Color32::RED, format!("{} Error occurred", regular::WARNING));
```

## Common Icons

### File Operations
- `regular::FOLDER` - Folder
- `regular::FOLDER_OPEN` - Open folder
- `regular::FILE` - File
- `regular::FLOPPY_DISK` - Save

### Actions
- `regular::ARROW_CLOCKWISE` - Refresh/Reload
- `regular::GEAR` - Settings
- `regular::LIGHTNING` - Quick action
- `regular::DOWNLOAD_SIMPLE` - Download/Install

### Status
- `regular::CHECK` - Checkmark
- `regular::CHECK_CIRCLE` - Success
- `regular::CHECK_SQUARE` - Checkbox checked
- `regular::SQUARE` - Checkbox unchecked
- `regular::X` - Close/Delete
- `regular::X_CIRCLE` - Error
- `regular::WARNING` - Warning
- `regular::INFO` - Information

### Navigation
- `regular::ARROW_LEFT` - Back
- `regular::ARROW_RIGHT` - Forward
- `regular::SKIP_FORWARD` - Skip/Next
- `regular::SKIP_BACK` - Previous

## Icon Variants

egui-phosphor provides multiple variants:
- `regular::` - Regular weight (default)
- `fill::` - Filled icons
- `bold::` - Bold weight
- `light::` - Light weight
- `thin::` - Thin weight
- `duotone::` - Duotone style

## Finding Icons

Browse all available icons at: https://phosphoricons.com/

Icon names in the library follow this pattern:
- Phosphor name: "folder-open" → Rust constant: `regular::FOLDER_OPEN`
- Phosphor name: "arrow-clockwise" → Rust constant: `regular::ARROW_CLOCKWISE`
- Hyphens become underscores, all uppercase

## Alternatives

Other icon libraries for egui:
- **egui-remixicon** - Remix Icons
- **egui-nerdfonts** - Nerd Fonts icons

All work with the same pattern: add font via FontDefinitions, use icon constants in UI.

## Benefits

✅ Works reliably across all platforms (Windows, macOS, Linux)
✅ Professional, consistent appearance
✅ No color emoji parsing issues
✅ Lightweight (outline fonts only)
✅ Comprehensive icon set (1000+ icons)
✅ Fallback font integration - icons work inline with text
