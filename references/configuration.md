# Dependencies & Installation

## Required Python Packages

The skill requires PDF extraction tools for Op1 (Ingest). Install via `uv` or `pip`:

```bash
# Core extraction (recommended — fastest, most reliable)
uv pip install pymupdf pymupdf4llm pdfplumber

# Fallback for image-heavy PDFs (requires Java 21)
uv pip install opendataloader-pdf
```

## Verification

```bash
python3 -c "import pymupdf; print('pymupdf OK')"
python3 -c "import pymupdf4llm; print('pymupdf4llm OK')"
python3 -c "import pdfplumber; print('pdfplumber OK')"
```

## PDF Tool Fallback Chain

When extracting text from PDFs, the skill tries tools in this order:

```
pymupdf4llm  →  opendataloader-pdf  →  pdfplumber  →  PyMuPDF(fitz)  →  mark as "extraction failed"
```

## Java Runtime (Optional)

Required only if you use `opendataloader-pdf`:
```bash
apt-get install default-jre-headless   # Debian/Ubuntu
brew install openjdk                  # macOS
```

## No External Services Required

This skill runs entirely offline. No API keys, no internet access, no third-party services.
