---
name: presentation-slides
description: "Generate presentation slides (beamer LaTeX → PDF + editable PPTX) from any text source: a quick review, paper abstracts, bullet-point outline, or compiled paper. Use when user says '做PPT', '做幻灯片', 'make slides', 'presentation', '从综述生成slides', '生成slides', or wants to turn notes/review into a talk deck."
argument-hint: "[source-file-or-topic] [--pages N] [--minutes M] [--style academic|minimal|bold]"
---

# Presentation Slides: From Text to Talk Deck

Generate presentation slides from: **$ARGUMENTS**

## Context

This skill turns **any text source** into a slide deck:
- `quick-review/QUICK_REVIEW.md` from `/skill:quick-lit-review`
- A paper's abstract + key sections
- A bullet-point outline the user wrote
- A compiled paper (falls back to `/skill:paper-slides` behavior)

Unlike `/skill:paper-slides` (which requires a full compiled LaTeX paper), this skill is **source-agnostic** and optimized for rapid preparation.

## Constants

- **SOURCE** = auto-detect from `$ARGUMENTS` (file path, directory, or raw text topic)
- **PAGES = 12** — Target slide count. Override with `--pages N`.
- **MINUTES = 15** — Talk duration. Override with `--minutes M`.
- **STYLE = `academic`** — Visual style. Options: `academic` (clean, beamer metropolis), `minimal` (bare text, max content), `bold` (strong colors, big fonts, teaching style).
- **ASPECT_RATIO = `16:9`**
- **OUTPUT_DIR = `presentation/`**
- **SPEAKER_NOTES = true** — Include `\note{}` blocks in beamer.
- **AUTO_PROCEED = false** — Wait for user confirmation at each checkpoint.

## Workflow (Step-by-Step, Human Interruptible)

### Phase 1: Ingest Source

Parse `$ARGUMENTS` to identify the source:

```bash
# Case A: User provided a file path
if [ -f "$SOURCE" ]; then
    cat "$SOURCE" > /tmp/slide_source.txt
fi

# Case B: User provided a directory (e.g., quick-review/)
if [ -d "$SOURCE" ]; then
    cat "$SOURCE/QUICK_REVIEW.md" > /tmp/slide_source.txt 2>/dev/null || \
    cat "$SOURCE"/*.md > /tmp/slide_source.txt
fi

# Case C: Raw topic (no file) — call quick-lit-review internally or ask user
```

If source is ambiguous, **ask user** before proceeding.

### Phase 2: Design Slide Structure

Analyze the source and design a slide outline. Write to `presentation/SLIDE_OUTLINE.md`:

```markdown
# Slide Outline: [Title]

## Meta
- Target pages: [N]
- Talk minutes: [M]
- Style: [style]
- Source: [source file]

## Slide-by-Slide Plan

### Slide 1: Title
- Content: [Title, author, date, affiliation]
- Visual: [none / logo]

### Slide 2: Motivation & Problem
- Content: [2-3 bullets]
- Visual: [diagram / figure reference]

### Slide 3: Background
- ...

### Slide N: Conclusion & Discussion
- Content: [takeaways + open questions]
```

**Design principles:**
- **1 slide ≈ 1 minute** for oral talks; **1.5 slides/minute** for spotlights
- No slide with > 5 bullets or > 80 words
- Every slide must have a **visual element** (diagram, table, figure, or at least a strong title)
- Placeholder slides for key figures from papers — mark `[FIGURE: cite paper X, fig Y]`

**Checkpoint**: Present outline to user. Wait for approval or edits before proceeding.

### Phase 3: Generate Beamer LaTeX

Write `presentation/slides.tex` using beamer (metropolis theme for academic, default beamer for minimal, custom colors for bold).

**Template structure:**

```latex
\documentclass[aspectratio=169]{beamer}
\usetheme{metropolis}  % or default / custom
\metroset{sectionpage=progressbar, numbering=fraction}

\title{[Title]}
\author{[Author]}
\date{[Date]}

\begin{document}
\maketitle

\begin{frame}{Motivation}
  \begin{itemize}
    \item ...
  \end{itemize}
  \note{Speaker notes...}
\end{frame}

% ... more frames ...

\begin{frame}{Conclusion}
  \begin{enumerate}
    \item Key takeaway 1
    \item Key takeaway 2
    \item Open question
  \end{enumerate}
  \note{Transition to Q&A...}
\end{frame}

\end{document}
```

**Content rules:**
- Use `\columns` for side-by-side comparisons
- Use `\begin{block}` / `\begin{alertblock}` for emphasis
- Use `\begin{table}` for method comparisons
- Use `\includegraphics` for figures (placeholders if files not available)
- Use `\note{}` for speaker notes
- Add `\usepackage[backend=biber,style=numeric]{biblatex}` + `\printbibliography` if citations exist

**Inserting figures from `quick-lit-review` (if available):**

If the source is `quick-review/QUICK_REVIEW.md` and `quick-review/figures/` exists:
1. Copy relevant figures to `presentation/figures/`
2. In the LaTeX source, use `\includegraphics[width=0.8\textwidth]{figures/filename.png}`
3. Add a `\caption{...}` or footnote citing the original paper

Example:
```latex
\begin{frame}{Method Overview}
  \centering
  \includegraphics[width=0.75\textwidth]{figures/nerf_p3_img1.png}
  \vspace{0.3em}
  \small Source: Mildenhall et al., ECCV 2020
\end{frame}
```

**TikZ diagrams:** For method architectures or flowcharts that lack a source image, draw them with TikZ directly in the `.tex` file. Prefer TikZ over bitmap images for crisp projection quality.

### Phase 4: Compile & Fix

```bash
cd presentation
latexmk -pdf -interaction=nonstopmode slides.tex 2>&1 | tee compile.log
```

If compilation fails:
1. Read `compile.log`
2. Fix error (missing package, typo, figure path)
3. Recompile (max 3 attempts)
4. If stuck, invoke `Agent: subagent_type: "coder"` for diagnosis

**Stuck after 2 attempts?** Manually inspect the error and apply targeted fix.

### Phase 5: Generate PPTX (Editable Fallback)

After PDF succeeds, generate an editable PPTX:

```bash
# Option A: pandoc (if installed)
pandoc -o presentation/slides.pptx presentation/slides.md 2>/dev/null || true

# Option B: python-pptx (if available)
# Generate a simple PPTX with python-pptx as fallback
```

If neither tool is available, write `presentation/slides.md` (Marp-compatible markdown) as a lightweight editable format:

```markdown
---
marp: true
theme: default
paginate: true
---

# Title

---

## Motivation
- Bullet 1
- Bullet 2
```

### Phase 6: Visual Review by PDF (Recommended)

After compiling the PDF, have a reviewer inspect the **actual visual output** to catch layout issues, density problems, and readability flaws that are invisible in the LaTeX source.

```yaml
Agent:
  subagent_type: "research-reviewer"
  prompt: |
    You are a presentation design reviewer. Please read the following files:
    - presentation/slides.pdf (the compiled slide deck — view it directly)
    - presentation/slides.tex (the LaTeX source — for reference)

    Inspect the PDF visually and evaluate:

    1. **Layout & Alignment**: Any overflow, misalignment, or clipping?
       - Text running off the slide edges
       - Figures/tables truncated or poorly positioned
       - Uneven spacing between blocks
    2. **Content Density**: Is any slide too crowded?
       - > 5 bullet points on one slide
       - > 80 words of body text
       - Tiny font sizes to fit content
    3. **Visual Hierarchy**: Can the audience grasp the key point in 3 seconds?
       - Title clearly states the slide's message
       - Emphasis (bold, color, block) guides the eye
       - No competing visual elements
    4. **Figure/Table Quality**: Are they readable from the back row?
       - Axis labels, legends, and captions legible
       - Colors distinguishable in grayscale (projectors vary)
       - No pixelation or compression artifacts
    5. **Flow & Narrative**: Does the story build logically?
       - motivation → background → core content → results → conclusion
       - Each slide has a clear transition to the next
    6. **Illustrations & TikZ (CRITICAL — focus here)**: 
       - **\includegraphics**: Are inserted figures properly scaled? Do they overflow the frame? Are they pixelated or misaligned?
       - **TikZ diagrams**: Are lines/edges crisp? Are node labels readable at projection size? Do arrows overlap with text? Is the layout balanced (not lopsided)?
       - **Missing illustrations**: Are there slides that are pure text but should have a figure? (e.g., "Method Overview" without a diagram, "Results" without a plot)
       - **Placeholder abuse**: Are there `[FIGURE]` placeholders that should have been replaced with actual `\includegraphics` or TikZ code?
    7. **Beamer-Specific Issues**:
       - Overfull/underfull hbox warnings visible in output
       - Block titles overlapping with body
       - Column ratios causing imbalance
       - `\includegraphics` using wrong path or missing file extension

    Return your assessment in this format:
    - **Verdict**: PASS / MINOR_FIXES / MAJOR_REDESIGN
    - **Slide-by-slide issues** (if any): list slide number + problem + suggested fix
    - **Figure/TikZ specific fixes**: if any illustration is broken, missing, or poorly rendered, suggest exact LaTeX fix or TikZ code replacement
    - **Global suggestions**: typography, color, spacing, or structure improvements

    Be specific: quote the problematic line from slides.tex if possible.
```

**How to apply fixes:**
1. Parse the reviewer's feedback
2. Edit `presentation/slides.tex` to fix each issue
3. Recompile with `latexmk -pdf slides.tex`
4. If major redesign is suggested, return to **Phase 2** (revise outline) before rewriting tex

**Iterate up to 2 review rounds.** After the second round, accept remaining minor issues and ship.

> 💡 Tip: If the reviewer points out a specific slide is too dense, the fastest fix is often to split it into two slides or replace bullets with a figure/table.

## Output Artifacts

| File | Purpose |
|------|---------|
| `presentation/slides.tex` | Beamer LaTeX source |
| `presentation/slides.pdf` | Compiled PDF |
| `presentation/slides.md` | Marp markdown (editable fallback) |
| `presentation/slides.pptx` | PowerPoint (if pandoc available) |
| `presentation/SLIDE_OUTLINE.md` | Structured outline |

## Recovery / Resume

If interrupted, check which files exist in `presentation/` and resume from the next phase.

## Example Invocation

```bash
> /skill:presentation-slides "quick-review/QUICK_REVIEW.md" --pages 10 --minutes 15

> /skill:presentation-slides "paper/main.tex" --style minimal --pages 8

> /skill:presentation-slides "Neural Radiance Fields: A Survey" --pages 12
```

## Related Skills

| Skill | When to use instead |
|-------|---------------------|
| `/skill:paper-slides` | You have a **complete compiled paper** and want conference-grade beamer slides |
| `/skill:paper-poster` | You need a poster (A0/A1), not slides |
| `/skill:lecture-script` | Next step: write the spoken script for these slides |
| `/skill:quick-lit-review` | Previous step: generate the source material for the slides |
| `/skill:mermaid-diagram` | Need a custom diagram/flowchart for a specific slide |
