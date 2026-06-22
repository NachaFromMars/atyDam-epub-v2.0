---
name: atyDam-epub-v2.0
aliases:
  - pdf-epub-v2
  - pdf-epub v2
  - pdf2epub-v2
  - pdf2epub v2
  - pdf-to-epub-v2
  - pdf to epub v2
  - pdf2epub
  - pdf-to-epub
  - pdf-epub
  - convert-pdf-epub
  - book-to-epub
  - sach-to-epub
  - sach-pdf-epub
  - epub-from-pdf
  - atyDam-epub
  - atyDam epub
  - pdf-epub-pipeline
trigger_phrases:
  - "convert pdf sang epub"
  - "chuyển pdf thành epub"
  - "chuyển pdf sang epub"
  - "build epub từ pdf"
  - "build epub từ sách này"
  - "làm epub từ pdf"
  - "tạo epub từ pdf"
  - "pdf to epub"
  - "pdf -> epub"
  - "pdf2epub"
  - "convert sách này"
  - "chuyển sách này thành epub"
  - "đóng epub"
  - "đóng sách này"
  - "xây epub"
  - "xử lý sách pdf"
description: Automated PDF→EPUB conversion pipeline v2.0 (base version). Smart 9-phase pipeline + closed-loop audit to produce premium EPUB from any PDF book (especially Vietnamese + Pali Buddhist texts with broken font encoding). NO GPT-STYLE bold emphasis. For bold version, use atyDam-epub v3.0. Aliases: pdf-epub-v2, pdf2epub-v2, pdf-to-epub-v2, atyDam-epub.
---

# atyDam-epub v2.0 — Automated PDF→EPUB Pipeline (Base Version)

## When to use this skill

When user wants to convert a PDF book to premium EPUB format **without** bold emphasis:
- PDF with broken font encoding (Vietnamese + Pali Buddhist texts)
- Books with complex tables, figures, footnotes
- Books needing professional EPUB output (cover, hierarchy, italic markup)
- **No GPT-style bold** — plain clean text

For **GPT-style bold emphasis** version, use **atyDam-epub v3.0** instead.

## Pipeline Overview — 9 Phases + Audit Loop

```
PDF input
  │
  ├─► Phase A: EXTRACT (PyMuPDF)
  │     • Extract images, render cover, detect chapters/TOC/blank pages
  │     • Output: extract_meta.json
  │
  ├─► Phase 1: TEXT RECOVERY (Gemini Flash + REST API)
  │     • Split PDF to 4500-char chunks  
  │     • Fix Vietnamese diacritics + Pali subset font damage
  │     • CRITICAL: thinkingBudget=0 to avoid truncation
  │     • Output: recovered/chunk_NNNN.txt
  │
  ├─► Phase B2: PARAGRAPH RECONSTRUCT (regex)
  │     • Join physical lines into real paragraphs
  │     • Detect headings, lists, captions
  │     • Fix orphan letters (page break artifacts)
  │
  ├─► Phase B3: SMART LIST SPLIT (regex)
  │     • Split inline "1)... 2)... 3)..." patterns
  │     • Mark tables with TABLE_START/END for B9
  │
  ├─► Phase B6: AI STRUCTURE DETECTION (Gemini)
  │     • Hierarchy: CHƯƠNG > Roman > Letter > Number
  │     • Heading + body merge logic
  │     • Inline list to proper markdown
  │
  ├─► Phase B7: PERFECT STRUCTURE (Gemini)
  │     • Semantic paragraph boundaries (8 RULES vàng)
  │     • Merge broken lines, split on topic shift
  │
  ├─► Phase B9: TABLE VISION (Gemini Vision)
  │     • Render PDF page → image
  │     • Gemini Vision reads table layout from image
  │     • Output: HTML <table> with rowspan/colspan
  │
  ├─► Phase B5: PROOFREAD (Gemini)
  │     • Spelling fixes + Phật học terms
  │     • Dictionary + AI suggestions
  │
  ├─► Phase C-D: MARKUP + BUILD (Python + Pandoc)
  │     • Add YAML front matter
  │     • Pali italic wrapping
  │     • Image embeds + caption
  │     • Cover image, premium CSS
  │     • EPUB 3.2 structure
  │
  └─► AUDIT LOOP (closed-loop, max 5 iterations)
        • Layer 1: Structural (inline headings, paragraph)
        • Layer 2: Semantic (AI - orphan fragments)
        • Layer 3: Spelling (dict + AI suggestions)
        • Layer 4: Tables (validation, empty cells)
        • Layer 5: Images (file verify)
        • Layer 6: Cross-ref (numbered gaps)
        • Layer 7: AI meta-review (deep read)
        • Loop until 0 issues OR max iterations
```

## Step-by-step Execution

### Step 1: Setup workspace
```bash
WORKDIR="/root/.openclaw/workspace/<book-name>-epub"
mkdir -p "$WORKDIR/{chunks,recovered,images}"
cd "$WORKDIR"
cp scripts/* .  # Copy all phase scripts
```

### Step 2: Run pipeline
```bash
export GEMINI_API_KEY="..."

# Phase A: Extract
python3 phase_a_extract.py

# Phase 1: Recovery (paid tier required, ~$0.50 for 400-page book)
python3 pipeline_parallel.py  # 5 workers, thinkingBudget=0

# Phase B2-B3: Structure prep
python3 phase_b2_paragraphs.py
python3 phase_b3_smart_list.py

# Phase B6-B7: AI structure  
python3 phase_b6_smart_structure.py
python3 phase_b7_perfect_structure.py

# Phase B9: Vision tables
python3 phase_b9_tables_vision.py

# Phase B5: Proofread
python3 phase_b5_proofread.py

# Phase C-D: Build
python3 phase_c_markup.py
python3 phase_d_build.py

# Audit loop (closed)
python3 audit_loop.py
python3 audit_v2.py  # Phase A+B

# Final build
python3 phase_d_build.py  # Rebuild from cleaned text
```

### Step 3: Validate
```bash
java -jar /usr/share/java/epubcheck.jar TrietHoc-*.epub
# Expected: 0 fatals / 0 errors / 0 warnings
```

## Key Insights from Production Use

### Gemini Configuration (CRITICAL)
- **Model:** `gemini-2.5-flash` (paid tier)
- **thinkingBudget: 0** ← ESSENTIAL or output truncates
- **temperature: 0.1** for factual fidelity
- **maxOutputTokens: 16384** safety net
- **REST API** (not SDK 0.8.6) because old SDK doesn't support thinkingConfig

### Cost per book (~420 pages)
- Phase 1 Recovery: ~$0.54 (129 chunks)
- Phase 2 Tables: ~$0.20 (Vision rebuild)
- Phase 5 Proofread: ~$0.40
- Phase 7 Structure: ~$0.42
- Phase 8 Perfect: ~$0.42
- Audit V1: ~$0.79
- Audit V2: ~$0.38
- **Total: ~$3.15** (≈ 75,000 VNĐ)

### Common Issues & Fixes
1. **Output truncation** → `thinkingBudget: 0`
2. **Free tier 20 RPD limit** → Need paid billing
3. **Tables flat text** → Vision API with PDF page render
4. **Headers inline with text** → Regex split at sentence boundary + `\n\n`
5. **Pandoc auto-lists** → Use `markdown-fancy_lists-startnum` format
6. **Cell numbering becomes `<ol>`** → Replace `N.` with `(N)` in cells
7. **Caption duplicated in `<th>`** → Strip first row of thead
8. **Table click trắng xóa** → Remove background-color, only borders
9. **Reader theme override** → Don't force color/background
10. **Page break orphan letters** → Merge with next paragraph

### Quality Indicators (v2.0 target)
- ✅ Avg paragraph length 150-380 chars (proper semantic)
- ✅ 9 chapters proper hierarchy (not broken)
- ✅ All tables HTML with thead/tbody/th/td
- ✅ Italic Pali wrapping (~1,100+ terms)
- ✅ epubcheck: 0 errors / 0 warnings
- ✅ Images embedded (~30 figures)
- ✅ Cover from PDF page 1
- ✅ **NO bold emphasis** (plain clean text)

## Difference: v2.0 vs v3.0

| Feature | v2.0 | v3.0 |
|---------|------|------|
| Text recovery | ✅ | ✅ |
| Structure detection | ✅ | ✅ |
| Tables rebuild | ✅ | ✅ |
| Proofreading | ✅ | ✅ |
| **GPT-Style bold** | ❌ NO | ✅ YES (~1,500 patterns) |
| Cost per book | ~$3.15 | ~$3.53 |
| Output size | 1.4 MB | 1.4 MB |

## Files Structure

```
<book>-epub/
├── chunks/              # PDF text chunks (Phase A)
├── recovered/           # Font-recovered chunks (Phase 1)
├── images/              # Extracted images (Phase A)
├── extract_meta.json    # All metadata
├── cover.png            # Rendered cover
├── style-v2-premium.css # Tiểu Tâm color palette
├── pipeline_parallel.py # Parallel recovery (5 workers)
├── phase_a_extract.py   # Smart PDF extraction
├── phase_b2_paragraphs.py  # Paragraph reconstruct
├── phase_b3_smart_list.py  # Inline list split
├── phase_b5_proofread.py   # AI proofreading
├── phase_b6_smart_structure.py  # AI hierarchy
├── phase_b7_perfect_structure.py # Semantic paragraphs
├── phase_b9_tables_vision.py    # Tables with Vision
├── phase_c_markup.py    # Markdown markup
├── phase_d_build.py     # Pandoc EPUB build
├── audit_loop.py        # 7-layer closed-loop audit
├── audit_v2.py          # Deep audit + auto-apply
├── audit-log.json       # Audit findings
└── TrietHoc-A-Ty-Dam-VN.epub  # Final output (NO BOLD)
```

## Trigger Auto-run

When user provides a PDF path with intent to convert (standard aliases):
- "convert PDF sang EPUB"
- "pdf to epub"
- "pdf2epub"
- etc.

Em will run v2.0 by default. For **GPT-style version**, user must explicitly request **v3.0**.

## Memory Note

atyDam-epub v2.0 is the **base version** without GPT-style bold. Produced from work done 16/05/2026 on
Triết Học A-Tỳ-Đàm (Buddha Abhidhamma) — Dr. Mehm Tin Mon. 
- V10 (LANDMARK): v2.0 base pipeline
- V11: v3.0 with GPT-style bold emphasis

For **GPT-style version**, see **atyDam-epub v3.0** skill.
