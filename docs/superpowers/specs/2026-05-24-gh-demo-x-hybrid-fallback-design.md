# gh-demo-x Hybrid Fallback Design

## Goal

When Hermes returns partially useful judgment content but the formatter cannot reliably convert it into the 9-section decision card, avoid both:

- empty-shell structured output
- over-minimal fallback like "模型未返回可用判断内容"

Instead, preserve as much useful model output as possible while still keeping the final Markdown readable.

## User Requirement

Approved fallback strategy: **B**

- extract whatever structured fields can be extracted safely
- preserve the remaining useful raw judgment text
- do not discard model output just because full formatting failed

## Current Problem

The current formatter assumes that all 9 sections are present in a stable numbered format.

If Hermes output deviates, the formatter may still emit the 9 headers but with empty bodies like:

```md
### 1. 选题结论
****
```

This is worse than raw output because it hides potentially useful content.

## Proposed Behavior

### Success Path

If the formatter can extract enough non-empty sections, keep the current pretty decision-card layout.

### Hybrid Fallback Path

If the formatter cannot produce a reliable full decision card:

1. Keep any sections that were extracted successfully and contain meaningful content.
2. Add a final section:

```md
### 原始判断片段
```

3. Preserve the remaining cleaned Hermes output there, excluding:
   - obvious boilerplate
   - lines already consumed by extracted sections
   - noise like `clarify timed out...`

### Full Raw Fallback Path

If almost nothing can be extracted structurally, render:

```md
## 判断结果

以下内容为 Hermes 原始判断片段，因结构化解析失败，保留原文供人工阅读：

...
```

## Parsing Rules

- A section counts as usable only if its body has meaningful content after stripping Markdown decoration.
- Placeholder-only values like `****` do not count.
- The formatter should prefer losing layout before losing information.

## Non-Goals

- Do not attempt aggressive semantic reconstruction.
- Do not invent missing sections.
- Do not summarize or compress raw fallback content beyond minimal cleanup.

## Acceptance Criteria

For problematic outputs like `mattpocock/skills`:

- no empty-shell decision card
- no silent data loss
- final `.md` contains either:
  - valid structured sections plus raw fragment appendix, or
  - raw judgment fallback block

