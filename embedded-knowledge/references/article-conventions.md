# Article Authoring Conventions

Guidelines for writing article HTML content in `kb-data.ts`.

---

## Structure Pattern

Every article should follow this structure:

```html
<h1>Article Title</h1>

<p>Opening paragraph: one sentence explaining what this article covers and why it matters.</p>

<!-- Optional: correct a common misconception immediately -->
<div class="callout-warning">
  <strong>⚠️ 常見誤解：</strong>Describe what people get wrong and what the truth is.
</div>

<h2>Core Concept</h2>
<p>...</p>

<h2>Comparison Table (if applicable)</h2>
<table>
  <thead><tr><th>Feature</th><th>Option A</th><th>Option B</th></tr></thead>
  <tbody>
    <tr><td>...</td><td>...</td><td>...</td></tr>
  </tbody>
</table>

<h2>Flow Diagram (if applicable)</h2>
<pre>
Component A
  ↓ action
Component B
  ↓ action
Component C
</pre>

<h2>Common Questions</h2>
<p><strong>Q: ...</strong></p>
<p>A: ...</p>
```

---

## Callout Box Usage

| Box Type | When to Use |
|---|---|
| `callout-warning` | Common misconceptions, dangerous pitfalls, security notes |
| `callout-info` | Background context, references, "good to know" notes |
| `callout-tip` | Best practices, recommended approaches, key takeaways |

---

## Tables

Use tables to compare two or more options side-by-side. Always include a `<thead>` row.

```html
<table>
  <thead><tr><th>特性</th><th>SLPI</th><th>ADSP</th></tr></thead>
  <tbody>
    <tr><td>功耗</td><td>&lt; 1 mW</td><td>數 mW</td></tr>
    <tr><td>感測器介面</td><td>✅ 直接連接</td><td>❌ 透過 IPC</td></tr>
  </tbody>
</table>
```

Use `&lt;` and `&gt;` for angle brackets inside table cells.

---

## Code and Flow Diagrams

For ASCII flow diagrams, use bare `<pre>` (no inner `<code>` tag):

```html
<pre>
PBL（ROM）
  ↓ 驗證
XBL（UEFI）
  ↓ 初始化 DDR
TrustZone
  ↓ 建立安全世界
HLOS（Android / Linux）
</pre>
```

For inline code references: `<code>slpi.mbn</code>`, `<code>adsp.img</code>`.

---

## Placeholder Articles

When a subcategory exists but content is not yet available:

```html
<h1>Topic Name（即將加入）</h1>

<p>此分類將整理以下主題：</p>

<ul>
  <li><strong>子主題 A</strong>：簡短說明</li>
  <li><strong>子主題 B</strong>：簡短說明</li>
</ul>

<div class="callout-info">
  <strong>ℹ️ 備註：</strong>請提供相關對話或資料，我將整理成完整文件加入此分類。
</div>
```

---

## Content Quality Checklist

Before finalizing an article:
- [ ] Does the title match `art.title` exactly?
- [ ] Is the opening `<h1>` the same as the article title?
- [ ] Are misconceptions addressed with `callout-warning`?
- [ ] Are comparison tables used where two or more options are discussed?
- [ ] Are all `<` and `>` inside table cells escaped as `&lt;` and `&gt;`?
- [ ] Is the article self-contained (no references to "the previous section")?
