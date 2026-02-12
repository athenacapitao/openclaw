---
name: nano-pdf
description: Edit PDFs with natural-language instructions using the nano-pdf CLI.
homepage: https://pypi.org/project/nano-pdf/
metadata:
  {
    "openclaw":
      {
        "emoji": "📄",
        "requires": { "bins": ["nano-pdf"] },
        "install":
          [
            {
              "id": "uv",
              "kind": "uv",
              "package": "nano-pdf",
              "bins": ["nano-pdf"],
              "label": "Install nano-pdf (uv)",
            },
          ],
      },
  }
---

# nano-pdf

Use `nano-pdf` to apply edits to a specific page in a PDF using a natural-language instruction.

## When to Use This Skill

**Use when:**

- Editing PDF content with natural-language instructions
- Changing text, titles, or fixing typos in PDFs
- Making quick edits to specific PDF pages

**Don't use when:**

- Need to extract text from PDF → use standard PDF tools (pdftotext)
- Need to merge/split PDFs → use other PDF tools
- Need to create PDFs from scratch → use other tools

**Success Criteria:**

- Output PDF contains expected edits
- Visual verification confirms changes are correct

## Quick start

```bash
nano-pdf edit deck.pdf 1 "Change the title to 'Q3 Results' and fix the typo in the subtitle"
```

Notes:

- Page numbers are 0-based or 1-based depending on the tool's version/config; if the result looks off by one, retry with the other.
- Always sanity-check the output PDF before sending it out.

### Common Pitfalls

**What NOT to do:**

- Assuming page numbering is always 1-based: may be 0-based depending on version
- Sending output without visual verification: edits may be off
- Editing multiple pages in one command: process one page at a time
