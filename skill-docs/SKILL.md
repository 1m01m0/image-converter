---
name: document-converter
description: Convert office documents (DOCX, PPTX, XLSX) to/from PDF. Use when the user asks to convert a Word document, PowerPoint, or Excel file to PDF — or extract a PDF back into an editable office format.
---

# Document Converter

Convert between office formats (DOCX, PPTX, XLSX) and PDF. Relies on LibreOffice for the office→PDF direction (most reliable) and Python libraries for the PDF→office direction.

## Trigger

Use this skill when the user asks to:
- Convert a Word / PPT / Excel file to PDF
- Convert a PDF to Word / PPT / Excel
- "Export this document as PDF"
- "Turn this PDF into an editable file"
- Batch convert a folder of documents

## Direction Summary

| Direction | Engine | Reliability |
|-----------|--------|-------------|
| DOCX → PDF | LibreOffice | Excellent — preserves formatting, fonts, images |
| PPTX → PDF | LibreOffice | Excellent — preserves layouts, charts |
| XLSX → PDF | LibreOffice | Good — may need page-break tuning |
| PDF → DOCX | pdf2docx / LibreOffice | Moderate — text and basic formatting come through; complex layouts degrade |
| PDF → PPTX | Manual via pdfplumber | Low — text can be extracted, but slides must be rebuilt |
| PDF → XLSX | pdfplumber + openpyxl | Low — works for tables; freeform data is unreliable |

**Always warn the user** when going from PDF back to office formats — the result will need manual cleanup.

## Prerequisites

LibreOffice is the primary engine. Python libraries handle the reverse direction.

```bash
# LibreOffice (required for office → PDF)
which soffice || sudo apt-get install -y libreoffice-core libreoffice-writer libreoffice-impress libreoffice-calc

# Python packages (for PDF → office)
pip install pdf2docx pdfplumber pymupdf python-docx python-pptx openpyxl Pillow --break-system-packages 2>&1 | tail -3
```

If LibreOffice cannot be installed (e.g., no sudo), fall back to Python-only approaches and clearly communicate the quality trade-off.

## Office → PDF

### Single file

```bash
soffice --headless --convert-to pdf --outdir "OUTPUT_DIR" "INPUT_FILE"
```

LibreOffice auto-detects the input format from the extension. The output PDF lands in `--outdir` with the same base name.

### Batch conversion

```bash
# Convert all office files in a directory to PDF
mkdir -p output_pdfs
for f in "INPUT_DIR"/*.{docx,doc,dotx,pptx,ppt,xlsx,xls}; do
  [ -f "$f" ] || continue
  echo "Converting: $(basename "$f")"
  soffice --headless --convert-to pdf --outdir output_pdfs "$f"
done
echo "Done. PDFs are in output_pdfs/"
```

### Python fallback (DOCX only, simpler but lower fidelity)

When LibreOffice is not available, use python-docx + fpdf2 for DOCX:

```python
from docx import Document
from fpdf import FPDF
import os

doc = Document("input.docx")
pdf = FPDF()
pdf.add_page()

# Register a Unicode-capable font (fallback to built-in if not found)
for font_path in [
    "/usr/share/fonts/truetype/noto/NotoSansCJK-Regular.ttc",
    "/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf",
    None
]:
    if font_path is None or os.path.exists(font_path):
        break
if font_path and os.path.exists(font_path):
    pdf.add_font("Unicode", "", font_path, uni=True)
else:
    font_path = None

for para in doc.paragraphs:
    text = para.text.strip()
    if not text:
        pdf.ln(6)
        continue
    style = para.style.name.lower()
    if "heading 1" in style:
        pdf.set_font("Unicode" if font_path else "Helvetica", "B", 16)
    elif "heading 2" in style:
        pdf.set_font("Unicode" if font_path else "Helvetica", "B", 14)
    else:
        pdf.set_font("Unicode" if font_path else "Helvetica", "", 11)
    # Simple text wrapping
    pdf.multi_cell(0, 6, text)

pdf.output("output.pdf")
print("Converted (Python fallback — formatting may differ from original).")
```

Note: This fallback **loses all formatting** (tables, images, headers, footers, columns). Always prefer LibreOffice.

## PDF → Office

### PDF → DOCX (best reverse option)

Use `pdf2docx` — it preserves paragraphs, basic formatting, and tables.

```python
from pdf2docx import Converter

pdf_path = "input.pdf"
docx_path = pdf_path.rsplit(".", 1)[0] + "_converted.docx"

cv = Converter(pdf_path)
cv.convert(docx_path)
cv.close()

print(f"Converted: {docx_path}")
print("Note: Complex layouts may need manual cleanup.")
```

For better image/table extraction, use pymupdf as an alternative:

```python
import fitz  # pymupdf
from docx import Document
from docx.shared import Inches
import os, io

pdf = fitz.open("input.pdf")
doc = Document()

for page in pdf:
    # Extract text blocks with positions
    blocks = page.get_text("blocks")
    for b in sorted(blocks, key=lambda x: (x[1], x[0])):  # sort by y, then x
        text = b[4].strip()
        if text:
            doc.add_paragraph(text)

    # Extract images
    for img_index, img in enumerate(page.get_images(full=True)):
        xref = img[0]
        base_image = pdf.extract_image(xref)
        image_bytes = base_image["image"]
        ext = base_image["ext"]
        img_path = f"page{page.number+1}_img{img_index}.{ext}"
        with open(img_path, "wb") as f:
            f.write(image_bytes)
        doc.add_picture(img_path, width=Inches(5))
        os.remove(img_path)

doc.save("output.docx")
pdf.close()
print("Converted with pymupdf (text + images extracted).")
```

### PDF → XLSX (table extraction)

Only makes sense if the PDF contains tabular data:

```python
import pdfplumber
from openpyxl import Workbook

pdf = pdfplumber.open("input.pdf")
wb = Workbook()
ws = wb.active

for page_num, page in enumerate(pdf.pages):
    tables = page.extract_tables()
    for table_idx, table in enumerate(tables):
        if page_num > 0 or table_idx > 0:
            ws = wb.create_sheet()
        ws.title = f"Page{page_num+1}_T{table_idx+1}"[:31]
        for row in table:
            ws.append([cell or "" for cell in row])

pdf.close()
wb.save("output.xlsx")
print("Tables extracted to output.xlsx")
```

### PDF → PPTX (text extraction only)

There is no reliable automated conversion. Extract text and let the user decide how to arrange slides:

```python
import fitz
from pptx import Presentation
from pptx.util import Inches, Pt

pdf = fitz.open("input.pdf")
prs = Presentation()

for page in pdf:
    slide = prs.slides.add_slide(prs.slide_layouts[6])  # blank layout
    text = page.get_text()
    if text.strip():
        txBox = slide.shapes.add_textbox(Inches(1), Inches(1), Inches(8), Inches(5))
        tf = txBox.text_frame
        tf.word_wrap = True
        for line in text.strip().split("\n")[:30]:
            p = tf.add_paragraph()
            p.text = line
            p.font.size = Pt(14)

pdf.close()
prs.save("output.pptx")
print("Text extracted to output.pptx — slide content needs manual formatting.")
```

## Decision Flow

When the user requests a conversion, follow this logic:

```
User says: "Convert X to Y"

1. Is it office → PDF?
   → YES: Use LibreOffice headless. Done.
   → NO: Go to 2.

2. Is it PDF → DOCX?
   → YES: Use pdf2docx (best results). Warn about cleanup.
   → NO: Go to 3.

3. Is it PDF → XLSX (and PDF has tables)?
   → YES: Use pdfplumber. Warn about data verification.
   → NO: Go to 4.

4. Is it PDF → PPTX or PDF → XLSX (no tables)?
   → Extract text, create skeleton file. Warn user that full reconstruction is manual.
```

## Important Warnings

Always include these disclaimers when converting:

**Office → PDF**: "Fonts embedded in the PDF may differ slightly from the original if the system doesn't have the same fonts installed."

**PDF → Office**: "PDF is a presentation format, not a data format. The converted document will need review and cleanup. Tables, images, and complex layouts may not survive the conversion perfectly."
