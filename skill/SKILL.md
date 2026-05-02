---
name: image-converter
description: Convert images between PNG, JPEG, WebP, BMP, GIF, TIFF formats. Supports quality adjustment, resizing, and batch processing. Use when the user asks to convert, compress, resize, or change the format of images.
---

# Image Converter

A skill for converting images between formats using Python Pillow, with automatic handling of transparency, quality tuning, and batch operations.

## Trigger

Use this skill when the user asks to:
- Convert a picture to another format (e.g., "turn this PNG into a JPEG")
- Compress or reduce an image's file size
- Resize or scale images
- Batch convert a folder of images
- Change image quality

## Prerequisites

Pillow is the only required dependency. The skill auto-installs it if missing.

## Conversion Logic

### Step 1: Confirm the parameters

Before converting, ask (or infer) the following:

| Parameter | Options | Default |
|-----------|---------|---------|
| Output format | PNG, JPEG, WebP, BMP, GIF, TIFF | Ask if unclear |
| Quality | 10–100 (JPEG/WebP only) | 90 |
| Size | original / percentage / exact pixels | original |
| Output path | any writable path | same dir, `_converted` suffix |

### Step 2: Run the conversion

Use this Python script as the template. Replace the placeholders at the top with the actual values.

```python
from PIL import Image
import os, sys

# === CONFIGURE THESE ===
input_path = "INPUT_FILE_PATH"
output_format = "JPEG"   # PNG, JPEG, WebP, BMP, GIF, TIFF
quality = 90             # JPEG/WebP only, 1-100
resize_ratio = None      # e.g. 0.5 = 50%, None = keep original
custom_size = None       # e.g. (800, 600), overrides resize_ratio
# === END CONFIG ===

img = Image.open(input_path)
original_bytes = os.path.getsize(input_path)
original_size = img.size

# ── Transparency → solid background (for formats that don't support alpha) ──
if output_format in ("JPEG", "BMP") and img.mode in ("RGBA", "LA", "P"):
    if img.mode == "P":
        img = img.convert("RGBA")
    bg = Image.new("RGB", img.size, (255, 255, 255))
    if img.mode == "RGBA":
        bg.paste(img, mask=img.split()[-1])
    img = bg
elif img.mode == "P":
    img = img.convert("RGBA")

# ── Resize ──
if custom_size:
    img = img.resize(custom_size, Image.LANCZOS)
elif resize_ratio and resize_ratio != 1.0:
    w, h = img.size
    img = img.resize((int(w * resize_ratio), int(h * resize_ratio)), Image.LANCZOS)

# ── Output path ──
ext_map = {"JPEG": "jpg", "JPG": "jpg"}
ext = ext_map.get(output_format, output_format.lower())
output_path = os.path.splitext(input_path)[0] + f"_converted.{ext}"

# ── Save ──
save_kwargs = {"optimize": True}
if output_format in ("JPEG", "WebP"):
    save_kwargs["quality"] = quality
img.save(output_path, format=output_format, **save_kwargs)

# ── Report ──
new_bytes = os.path.getsize(output_path)
reduction = (1 - new_bytes / original_bytes) * 100
sign = "smaller" if reduction > 0 else "larger"
print(f"✓  Converted: {os.path.basename(output_path)}")
print(f"   {original_size[0]}x{original_size[1]} | "
      f"{original_bytes/1024:.1f} KB → {new_bytes/1024:.1f} KB "
      f"({abs(reduction):.0f}% {sign})")
```

### Step 3: Report results

After conversion, show the user:
- Output file name and path
- Before/after dimensions and file size
- Percentage change

## Batch Conversion

When the user wants to convert multiple files in a directory:

```python
from PIL import Image
import os, glob, sys

input_dir = "INPUT_DIR"
output_format = "JPEG"
quality = 90
extensions = ("*.png", "*.jpg", "*.jpeg", "*.webp", "*.bmp", "*.gif")

files = []
for ext in extensions:
    files.extend(glob.glob(os.path.join(input_dir, ext)))

if not files:
    print("No image files found.")
    sys.exit(1)

print(f"Found {len(files)} files. Converting to {output_format}...\n")

for i, fp in enumerate(files, 1):
    try:
        img = Image.open(fp)
        # ... same transparency + resize logic as single file ...
        out = os.path.splitext(fp)[0] + f".{output_format.lower()}"
        save_kwargs = {"optimize": True}
        if output_format in ("JPEG", "WebP"):
            save_kwargs["quality"] = quality
        img.save(out, format=output_format, **save_kwargs)
        print(f"  [{i}/{len(files)}] {os.path.basename(out)}")
    except Exception as e:
        print(f"  [{i}/{len(files)}] SKIP {os.path.basename(fp)}: {e}")

print(f"\nDone. {len(files)} files processed.")
```

## Format-Specific Notes

| Scenario | Handling |
|----------|----------|
| PNG → JPEG | Replace transparency with white background |
| JPEG → PNG | File grows (PNG is lossless); only useful when transparency is needed |
| Animated GIF | Pillow only reads frame 1 by default. Warn the user if input is animated. |
| WebP output | Best balance of size/quality, but older apps may not support it |
| HEIC/HEIF input | Requires `pillow-heif` (`pip install pillow-heif --break-system-packages`) |
| SVG input | Requires `cairosvg` (`pip install cairosvg --break-system-packages`) |

## Companion Tool

This skill ships with `image-converter.html` — a standalone browser-based converter that works without Python. Use it when:
- The user doesn't have Python available
- The user wants a visual drag-and-drop interface
- The user needs to convert images on a machine without a terminal

Tell the user: "Open `image-converter.html` in any browser — drag images in, pick a format, and download the result."
