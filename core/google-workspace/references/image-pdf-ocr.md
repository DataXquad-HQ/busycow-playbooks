# Image-Based PDF OCR Pattern

## Detection

PDFs downloaded from Google Drive may be fully image-based (no embedded text layer).
Detection via pymupdf:

```python
import pymupdf
doc = pymupdf.open("/tmp/file.pdf")
page = doc[0]
text = page.get_text()           # returns '' for image PDFs
blocks = page.get_text("blocks") # returns [] for image PDFs
d = page.get_text("dict")
# Image PDF signature: single block of type 1 (image)
is_image_pdf = (len(d["blocks"]) == 1 and d["blocks"][0]["type"] == 1)
```

## Setup (one-time)

```bash
sudo apt-get install -y tesseract-ocr poppler-utils
~/.hermes/hermes-agent/venv/bin/pip3 install pytesseract pdf2image
```

## OCR Pattern

```python
from pdf2image import convert_from_path
import pytesseract

images = convert_from_path("/tmp/file.pdf", dpi=150)  # dpi=150 is fast; use 200 for better accuracy
full_text = ""
for i, img in enumerate(images):
    full_text += f"\n--- Page {i+1} ---\n{pytesseract.image_to_string(img)}"

with open("/tmp/file_ocr.txt", "w") as f:
    f.write(full_text)
```

## Performance

- At dpi=150: ~15-25s per page depending on content density
- Multi-file batches (3 PDFs, 30 pages total) easily exceed 60s foreground timeout
- **Always run OCR in background** for multi-file or multi-page jobs:

```python
# Write script to /tmp/ocr_all.py, then:
terminal(command="python3 /tmp/ocr_all.py", background=True, notify_on_complete=True)
```

Include a progress file so you can verify completion:
```python
with open("/tmp/ocr_progress.txt", "a") as f:
    f.write(f"Done: {name} ({len(full_text)} chars)\n")
# Write "ALL DONE\n" at the end
```

## Notes

- `pdftotext` (poppler) also returns empty for image PDFs — same problem, different tool
- tesseract handles English well at dpi=150; add `lang="chi_tra"` for Traditional Chinese if needed
- OCR quality on PDFs generated from NotebookLM / presentation exports is generally good
