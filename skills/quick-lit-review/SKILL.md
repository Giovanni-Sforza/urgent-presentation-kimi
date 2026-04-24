---
name: quick-lit-review
description: "Rapid literature review for a given keyword + optional reference papers, producing a structured summary suitable for preparing a presentation or talk. Use when user says '帮我快速了解一下这个方向', 'review一下这个topic', '关键词综述', '快速文献调研', '准备presentation需要先理解', or wants a quick structured digest of a research area in 10-15 minutes."
argument-hint: "[keyword-or-topic] [--ref paper1.pdf|arxiv:1234.5678|URL ...]"
---

# Quick Lit Review: Rapid Structured Digest

Rapidly review and summarize: **$ARGUMENTS**

## Context

This skill is designed for **time-constrained preparation** (e.g., a professor assigns a keyword and asks for a presentation by tomorrow). It produces a concise, structured document that can be directly used to build slides or a lecture.

It is **NOT** a deep systematic survey — it trades completeness for speed and actionability. Typical runtime: 10-15 minutes of agent time.

## Workflow (Step-by-Step, Human Interruptible)

Each phase produces a file. The user can stop, edit, and resume at any point.

### Phase 1: Scope & Search

Parse `$ARGUMENTS` for:
- **Keyword / topic** (required)
- **`--ref`** flags: explicit reference papers (PDF paths, arXiv IDs, URLs)
- **`--depth`**: `lite` (3-5 papers, 10 min) | `standard` (8-12 papers, 15 min) | `deep` (15-20 papers, 25 min). Default: `standard`.
- **`--focus`**: specific angle (e.g., "methods", "applications", "theory", "recent trends")

Create output directory:
```bash
mkdir -p quick-review
```

#### 1A. Search for papers (if no `--ref` or to supplement)

Use available search tools to find the most relevant papers:

1. **arXiv search**: `/skill:arxiv "keyword"` (recent 2-3 years优先)
2. **Semantic Scholar**: `/skill:semantic-scholar "keyword"` (for published venue papers with citations)
3. **AlphaXiv** (if user provided arXiv URLs): `/skill:alphaxiv "arxiv-id"`
4. **Google Scholar / web** (fallback): web search for survey papers or seminal works

Selection criteria (in order):
1. Survey / review papers on the topic
2. Highly cited seminal papers
3. Recent SOTA papers (last 1-2 years)
4. Papers explicitly provided by user via `--ref`

Collect for each paper:
- Title, authors, year, venue
- 1-sentence core contribution
- Why it matters for this keyword

#### 1B. Read provided references & extract key figures

If user provided `--ref` papers:

**Step 1: Download / locate the paper**
- If ref is a local PDF path, use it directly
- If ref is an arXiv ID, **prefer downloading the TeX source tarball** (contains original figures)
- If ref is a URL, download the PDF to `quick-review/papers/`

**Step 2: Extract text**
- Read PDF text (use Shell tool with `pdftotext`, `pdfplumber`, or inline Python)
- Or read `.tex` source files if TeX source was downloaded
- Record core claims and key findings

**Step 3: Extract figures (CRITICAL)**

Figures and diagrams are essential for presentations. For arXiv papers, **always prefer the TeX source** for higher-quality original images.

```bash
# Create directories
mkdir -p quick-review/figures quick-review/papers

# Try to find the arxiv_fetch.py script
SCRIPT=$(find tools/ -name "arxiv_fetch.py" 2>/dev/null | head -1)
[ -z "$SCRIPT" ] && SCRIPT=$(find ~/.kimi/skills/arxiv/ -name "arxiv_fetch.py" 2>/dev/null | head -1)

# For arXiv IDs: download TeX source and extract images (PREFERRED)
for ID in ARXIV_ID1 ARXIV_ID2; do
    python3 "$SCRIPT" download-source "$ID" --dir quick-review/papers 2>/dev/null || true
done

# Copy extracted source images to figures directory
for src_dir in quick-review/papers/*_src; do
    [ -d "$src_dir" ] || continue
    prefix=$(basename "$src_dir" | sed 's/_src$//')
    find "$src_dir" -type f \( -iname '*.png' -o -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.pdf' -o -iname '*.eps' \) | while read -r img; do
        rel=$(realpath --relative-to="$src_dir" "$img" 2>/dev/null || echo "$(basename "$img")")
        safe_rel=$(echo "$rel" | tr '/' '_')
        cp "$img" "quick-review/figures/${prefix}_${safe_rel}"
        echo "Extracted: quick-review/figures/${prefix}_${safe_rel}"
    done
done
```

**If source download fails or for non-arXiv PDFs, fall back to PyMuPDF:**

```bash
# Fallback: extract images from PDF using PyMuPDF
python3 -c "
import fitz, sys, os
pdf = sys.argv[1]
prefix = os.path.basename(pdf).replace('.pdf','')
doc = fitz.open(pdf)
fig_count = 0
for page_num in range(len(doc)):
    page = doc[page_num]
    imgs = page.get_images(full=True)
    for idx, img in enumerate(imgs):
        xref = img[0]
        try:
            bi = doc.extract_image(xref)
            ext = bi['ext']
            if ext not in ('png', 'jpg', 'jpeg'):
                ext = 'png'
            path = f'quick-review/figures/{prefix}_p{page_num+1}_img{idx+1}.{ext}'
            with open(path, 'wb') as f:
                f.write(bi['image'])
            fig_count += 1
            print(path)
        except Exception as e:
            print(f'skip: {e}', file=sys.stderr)
print(f'Extracted {fig_count} images', file=sys.stderr)
" quick-review/papers/*.pdf 2>/dev/null || echo "PyMuPDF not available, trying fallback..."
```

If PyMuPDF is not available, install it:
```bash
pip install pymupdf 2>/dev/null || pip install PyMuPDF 2>/dev/null
```

**Step 4: Inspect extracted images with ReadMediaFile**

For each extracted image file in `quick-review/figures/`:
- Use `ReadMediaFile` to view the image
- Describe what you see: axes, curves, labels, captions, color scheme
- Judge if it is a **key figure** (illustrates the core method, main result, or important comparison)
- For key figures, record:
  - File path (relative to project root)
  - Brief visual description
  - Which paper it belongs to
  - Why it matters for the presentation

**Step 5: If source extraction and PyMuPDF both fail, use page screenshots**

Convert key pages (usually pages containing "Figure", "Fig.", or tables) to PNG:

```bash
# Fallback: pdftoppm (requires poppler)
pdftoppm -png -r 200 -f 1 -l 5 quick-review/papers/PAPER.pdf quick-review/figures/PAPER_page 2>/dev/null

# Or use sips on macOS
sips -s format png quick-review/papers/PAPER.pdf --out quick-review/figures/PAPER_page1.png 2>/dev/null
```

Then `ReadMediaFile` these PNGs to inspect figures visually.

**Priority rule**: A picture is worth a thousand words. If a paper has a famous figure (e.g., NeRF's ray marching diagram, Transformer architecture diagram), **always** extract and describe it.

### Phase 2: Synthesize Quick Review

Write `quick-review/QUICK_REVIEW.md` with the following **fixed structure**:

```markdown
# Quick Review: [Keyword]

## 1. Background & Definition
- What is [keyword]? (2-3 sentences, accessible to non-expert audience)
- Why does it matter? (motivation, real-world impact)
- Key terminology glossary (3-5 terms)

## 2. Key Papers & Contributions

| Paper | Year | Venue | Core Contribution | Why It Matters |
|-------|------|-------|-------------------|----------------|
| ... | ... | ... | ... | ... |

(Include 3-8 papers depending on `--depth`)

### 2.1. Seminal / Foundational Work
- [Paper A]: established the problem, first method, key dataset

### 2.2. Recent Advances
- [Paper B]: SOTA result, new paradigm
- [Paper C]: surprising finding, opened new direction

## 3. Methods Landscape

```
[Simple ASCII or text diagram showing method relationships]
Example:
Early Methods → Deep Learning Era → Transformer Era → Current SOTA
     ↓              ↓                  ↓               ↓
   Method A      Method B           Method C       Method D
```

### 3.1. Method Comparison

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| ... | ... | ... | ... |

## 4. Key Takeaways (for Presentation)
- **Takeaway 1**: [most important insight]
- **Takeaway 2**: [second insight]
- **Takeaway 3**: [practical implication]
- **Takeaway 4**: [open challenge]
- **Takeaway 5**: [future direction]

## 5. Open Questions & Debates
- What is still controversial?
- What are the limitations of current methods?
- What would be a good student project / thesis topic?

## 6. Key Figures (for Slides)

Extracted figures from the papers, ready to be referenced in slides.

| Filename | Source Paper | Visual Description | Suggested Slide Usage |
|----------|-------------|-------------------|----------------------|
| `quick-review/figures/2301.07041_fig1.png` | Attention Is All You Need (2017) | Multi-head attention architecture diagram | Slide 3: illustrate core concept |
| `quick-review/figures/2301.07041_fig2.png` | Attention Is All You Need (2017) | BLEU score comparison table vs prior methods | Slide 6: results comparison |
| ... | ... | ... | ... |

**Selection criteria for inclusion:**
- Must illustrate a **core concept** (method architecture, problem setup) or **main result**
- Must be **legible** when projected (labels readable, contrast sufficient)
- Prefer **vector-friendly diagrams** over dense tables
- Include caption or one-sentence description for each

If no figures were successfully extracted, note: `<!-- Figures not extracted; check quick-review/papers/ for source files -->`

## 7. Suggested Slide Outline
## 6. Key Figures (for Slides)

Extracted figures from the papers, ready to be referenced in slides.

| Filename | Source Paper | Page | Visual Description | Suggested Slide Usage |
|----------|-------------|------|-------------------|----------------------|
| `quick-review/figures/nerf_p3_img1.png` | NeRF (2020) | p.3 | 3D volumetric rendering diagram: camera rays → sampling along ray → color/opacity integration | Slide 3: illustrate core concept |
| `quick-review/figures/nerf_p7_img1.png` | NeRF (2020) | p.7 | PSNR comparison table vs prior methods | Slide 6: results comparison |
| ... | ... | ... | ... | ... |

**Selection criteria for inclusion:**
- Must illustrate a **core concept** (method architecture, problem setup) or **main result**
- Must be **legible** when projected (labels readable, contrast sufficient)
- Prefer **vector-friendly diagrams** over dense tables
- Include caption or one-sentence description for each

If no figures were successfully extracted, note: `<!-- Figures not extracted; manually screenshot from PDFs in quick-review/papers/ -->`

## 7. Suggested Slide Outline
- Slide 1: Title + motivation
- Slide 2: Background / problem setup
- Slide 3: Timeline / evolution
- Slide 4-5: Key methods (2-3 representative)
- Slide 6: Results comparison
- Slide 7: Takeaways
- Slide 8: Open questions / discussion

## References
[Full citation list in any consistent format]
```

**Critical rules for synthesis:**
- Use **bullet points** and **tables** — easy to scan, easy to convert to slides
- Every claim must be traceable to a specific paper
- Highlight **surprising results** and **intuitive insights** — these make good presentation content
- Keep total length to **2-4 pages** (lite) or **4-6 pages** (standard)

### Phase 3: Self-Check (Optional)

If `depth >= standard`, do a quick sanity check:

```yaml
Agent:
  subagent_type: "research-reviewer"
  prompt: |
    I am preparing a quick literature review on [keyword] for a presentation.
    Please read quick-review/QUICK_REVIEW.md and check:
    1. Are there any obvious missing seminal papers?
    2. Is the method comparison fair and accurate?
    3. Are the takeaways actually supported by the cited papers?
    4. Is anything stated as fact that is actually debated?

    Return: PASS / MINOR_ISSUES / MAJOR_ISSUES with specific fixes.
```

If reviewer finds issues, apply fixes and rewrite `QUICK_REVIEW.md`.

## Output Artifacts

| File | Purpose |
|------|---------|
| `quick-review/QUICK_REVIEW.md` | Main structured digest |
| `quick-review/search_log.md` | List of searched queries and found papers (for transparency) |
| `quick-review/figures/` | Extracted figures from papers (PNG/JPG) |
| `quick-review/papers/` | Downloaded PDFs (source for re-extraction if needed) |
| `quick-review/figures/` | Extracted figures from papers (PNG/JPG/PDF/EPS from TeX source) |
| `quick-review/papers/` | Downloaded TeX source tarballs or PDFs (source for re-extraction if needed) |
| `quick-review/REFERENCES.bib` | Optional BibTeX entries (if user needs LaTeX) |

## Recovery / Resume

If session is interrupted, read existing `quick-review/QUICK_REVIEW.md` and continue from the incomplete section.

## Example Invocation

```bash
> /skill:quick-lit-review "diffusion models for text-to-speech" --depth standard --focus methods

> /skill:quick-lit-review "neural radiance fields" --ref arxiv:2003.08934 arxiv:2202.02688 --depth lite

> /skill:quick-lit-review "in-context learning theory"
```

## Related Skills

| Skill | When to use instead |
|-------|---------------------|
| `/skill:research-lit` | Deep, comprehensive literature review for a paper's Related Work section |
| `/skill:arxiv` | Bulk download or metadata-only search |
| `/skill:semantic-scholar` | Need citation counts and venue metadata |
| `/skill:idea-discovery` | You want to generate research ideas, not just summarize |
| `/skill:presentation-slides` | Next step: turn this review into actual slides |
