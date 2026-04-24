---
name: lecture-script
description: "Generate a spoken lecture script from slides or a review document. Produces page-by-page talking notes with timing, transitions, and audience interaction cues. Use when user says '写演讲稿', 'lecture script', 'talk script', 'speaker notes', 'presentation script', or wants a full spoken narrative to accompany slides."
argument-hint: "[slides-file-or-review-file] [--minutes M] [--language zh|en]"
---

# Lecture Script: From Slides to Spoken Narrative

Generate lecture script from: **$ARGUMENTS**

## Context

This skill produces a **complete spoken script** that accompanies a slide deck or review document. It is designed for:
- Class presentations (student presents a keyword/paper to classmates)
- Seminar talks
- Thesis defenses
- Lightning talks

The script is **page-by-page** (or section-by-section), with explicit timing, transition phrases, and audience interaction cues. It reads like a teleprompter but feels natural.

## Constants

- **SOURCE** = auto-detect from `$ARGUMENTS` (slides.tex, slides.pdf, QUICK_REVIEW.md, etc.)
- **MINUTES = 15** — Talk duration. Script length is calibrated to this.
- **LANGUAGE = `zh`** — Default spoken language. Options: `zh` (中文), `en` (English). Override with `--language en`.
- **STYLE = `academic`** — Tone. Options: `academic` (formal, precise), `casual` (conversational, engaging), `teaching` (Socratic, asks questions).
- **OUTPUT_DIR = `presentation/`**
- **WORDS_PER_MINUTE = 130** (zh) / 160 (en) — Speaking rate for script length estimation.

## Workflow (Step-by-Step, Human Interruptible)

### Phase 1: Ingest Source

Detect source type:

```bash
# Case A: Beamer .tex file
if [ -f "$SOURCE" ] && [[ "$SOURCE" == *.tex ]]; then
    # Extract frame titles and content
    grep -A 20 '\\begin{frame}' "$SOURCE" > /tmp/slide_content.txt
fi

# Case B: PDF slides (fallback)
if [ -f "$SOURCE" ] && [[ "$SOURCE" == *.pdf ]]; then
    pdftotext "$SOURCE" /tmp/slide_content.txt 2>/dev/null || echo "PDF text extraction failed"
fi

# Case C: Markdown review / outline
if [ -f "$SOURCE" ] && [[ "$SOURCE" == *.md ]]; then
    cat "$SOURCE" > /tmp/slide_content.txt
fi

# Case D: Directory containing presentation/
if [ -d "$SOURCE" ]; then
    cat "$SOURCE/"*.tex "$SOURCE/"*.md > /tmp/slide_content.txt 2>/dev/null
fi
```

If source is ambiguous, **ask user**.

### Phase 2: Parse Structure

Identify the narrative structure from the source:

```markdown
## Parsed Structure

### Section 1: Opening (Slide 1)
- Title: [...]
- Key points: [...]
- Estimated time: [X min]

### Section 2: Motivation (Slides 2-3)
- ...

### Section N: Conclusion (Slide N)
- ...
```

Write this to `presentation/SCRIPT_STRUCTURE.md` as an intermediate artifact.

**Checkpoint**: Show structure to user. Wait for approval.

### Phase 3: Write Spoken Script

Write `presentation/LECTURE_SCRIPT.md` with the following format:

```markdown
# Lecture Script: [Title]

## Meta
- Total time: [M] minutes
- Language: [zh/en]
- Style: [academic/casual/teaching]
- Source: [file]

---

## [00:00 - 00:30] Slide 1: Title

**What to say:**
"大家好，今天我要介绍的是 [Title]。这个方向最近几年在 [venue/field] 引起了广泛关注，因为它解决了 [core problem]..."

**Transition cue:**
→ "在开始具体方法之前，我先简要说明一下这个问题的背景。"

**Audience interaction:**
[teaching style only] "大家在之前的学习中，有没有接触过 [related concept]?"

**Timing note:**
- Target: 30 seconds
- Words: ~65 (zh) / ~80 (en)
- Do NOT read the title verbatim — add context

---

## [00:30 - 02:00] Slide 2: Motivation

**What to say:**
"为什么这个问题重要？我们可以从三个角度来看..."

**Transition cue:**
→ "了解了问题的动机之后，我们来看一下现有的方法是怎么解决这个问题的。"

**Timing note:**
- Target: 90 seconds
- Words: ~195 (zh) / ~240 (en)
- Emphasize the gap — this is the hook

---

## [MM:SS - MM:SS] Slide N: Conclusion & Q&A

**What to say:**
"总结一下，今天我们讨论了..."

**Transition cue:**
→ "以上就是我今天报告的全部内容，欢迎大家提问。"

**Audience interaction:**
- Prepare 2-3 anticipated questions and brief answers
- Mark "if asked about X, say Y"

---

## Appendix: Anticipated Q&A

### Q1: [Common question 1]
**A:** [2-3 sentence answer]

### Q2: [Common question 2]
**A:** [2-3 sentence answer]

### Q3: [Technical depth question]
**A:** [Brief answer + "happy to discuss offline"]
```

**Writing rules:**
- **Page-by-page**: Every slide gets its own section with `[start - end]` timestamp
- **Natural spoken language**: contractions, colloquialisms, filler words allowed ("那么", "其实", "you know", "so")
- **Never read bullets verbatim**: Expand each bullet into 2-3 spoken sentences
- **Transition cues**: Explicit phrase to bridge to next slide (prevents awkward silence)
- **Timing notes**: Per-slide target time and word count — user can adjust speed
- **Audience interaction** (teaching style): Embed questions, polls, or "think-pair-share" moments
- **Emphasis markers**: Use **bold** for words to stress, [pause] for deliberate pauses
- **Appendix Q&A**: Prepare 3 likely questions with concise answers

### Phase 4: Calibrate Length

Calculate total script words:
```bash
wc -w presentation/LECTURE_SCRIPT.md
```

Compare against target:
- Target words = MINUTES × WORDS_PER_MINUTE
- If actual > 1.2 × target: mark sections to trim (suggest which slides to shorten)
- If actual < 0.8 × target: mark sections to expand (suggest where to add detail or examples)

Write calibration note to `presentation/SCRIPT_CALIBRATION.md`:

```markdown
# Script Calibration

- Target duration: [M] min
- Target words: [N]
- Actual words: [A]
- Ratio: [A/N]

## If running long, trim these sections:
- [Slide X]: reduce example detail
- [Slide Y]: skip optional background

## If running short, expand these sections:
- [Slide Z]: add concrete example
- [Slide W]: add comparison with baseline
```

### Phase 5: Review (Optional)

```yaml
Agent:
  subagent_type: "paper-editor"
  prompt: |
    Review the lecture script in presentation/LECTURE_SCRIPT.md.
    Check:
    1. Is the language natural and spoken (not written prose)?
    2. Are transitions smooth between slides?
    3. Is the timing realistic?
    4. Does the script match the slide content (no phantom claims)?
    5. Are the Q&A anticipations realistic?

    Return: PASS / FIX with specific line numbers.
```

## Output Artifacts

| File | Purpose |
|------|---------|
| `presentation/LECTURE_SCRIPT.md` | Main spoken script (page-by-page) |
| `presentation/SCRIPT_STRUCTURE.md` | Parsed narrative structure |
| `presentation/SCRIPT_CALIBRATION.md` | Timing calibration & trim/expand notes |

## Recovery / Resume

If interrupted, read existing `presentation/LECTURE_SCRIPT.md` and append from the next slide.

## Example Invocation

```bash
> /skill:lecture-script "presentation/slides.tex" --minutes 15 --language zh

> /skill:lecture-script "quick-review/QUICK_REVIEW.md" --minutes 10 --language en --style casual

> /skill:lecture-script "presentation/" --minutes 20
```

## Related Skills

| Skill | When to use instead |
|-------|---------------------|
| `/skill:presentation-slides` | Previous step: generate the slide deck |
| `/skill:quick-lit-review` | Two steps back: generate the source material |
| `/skill:paper-slides` | You need conference-grade slides from a full paper |
| `/skill:rebuttal` | You are responding to reviewer comments, not giving a talk |
