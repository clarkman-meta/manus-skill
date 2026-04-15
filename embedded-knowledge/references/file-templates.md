# File Templates

Key source files for the knowledge base. Copy and adapt these when building a new KB site.

---

## `client/index.html`

```html
<!doctype html>
<html lang="zh-TW">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1" />
    <title>Knowledge Base</title>
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

---

## `client/src/lib/kb-data.ts` — Data Schema

```typescript
export interface Article {
  id: string;
  title: string;
  description: string;
  tags: string[];
  content: string; // HTML string rendered with dangerouslySetInnerHTML
  lastUpdated: string; // "YYYY-MM-DD"
}

export interface SubCategory {
  id: string;
  title: string;
  description: string;
  icon: string;       // lucide-react icon name string
  articles: Article[];
}

export interface Category {
  id: string;
  title: string;
  description: string;
  icon: string;       // lucide-react icon name string
  color: string;      // "green" | "blue"
  subcategories: SubCategory[];
}

export const categories: Category[] = [ /* ... */ ];

// Helper functions
export function getCategoryById(id: string): Category | undefined { ... }
export function getSubCategoryById(catId: string, subId: string): SubCategory | undefined { ... }
export function getArticleById(catId: string, subId: string, artId: string): Article | undefined { ... }
export function getAllArticles(): Array<Article & { categoryId: string; subcategoryId: string }> { ... }
```

---

## `client/src/App.tsx` — Routes

```tsx
import { Route, Switch } from "wouter";
import Home from "./pages/Home";
import ArticlePage from "./pages/ArticlePage";
import CategoryPage from "./pages/CategoryPage";

function Router() {
  return (
    <Switch>
      <Route path="/" component={Home} />
      <Route path="/category/:catId" component={CategoryPage} />
      <Route path="/article/:catId/:subId/:artId" component={ArticlePage} />
      <Route component={NotFound} />
    </Switch>
  );
}
```

---

## `client/src/index.css` — Key CSS Classes

The `.kb-prose` class drives all article typography. Add to `@layer components`:

```css
.kb-prose h1 { @apply text-2xl font-bold mb-4 mt-0 text-slate-900; font-family: 'Space Grotesk', sans-serif; }
.kb-prose h2 { @apply text-xl font-semibold mb-3 mt-8 text-slate-800 border-b border-slate-200 pb-2; font-family: 'Space Grotesk', sans-serif; }
.kb-prose h3 { @apply text-base font-semibold mb-2 mt-6 text-slate-700; font-family: 'Space Grotesk', sans-serif; }
.kb-prose p  { @apply text-sm leading-7 text-slate-600 mb-4; }
.kb-prose ul { @apply list-disc list-inside text-sm text-slate-600 mb-4 space-y-1 pl-2; }
.kb-prose code { @apply bg-slate-100 text-green-700 px-1.5 py-0.5 rounded text-xs; font-family: 'JetBrains Mono', monospace; }
.kb-prose pre  { @apply bg-slate-900 text-green-300 p-4 rounded-lg text-xs overflow-x-auto mb-4; font-family: 'JetBrains Mono', monospace; }
.kb-prose table { @apply w-full text-sm mb-4 border-collapse; }
.kb-prose th { @apply bg-slate-800 text-slate-100 px-3 py-2 text-left font-medium text-xs uppercase tracking-wide; }
.kb-prose td { @apply px-3 py-2 border-b border-slate-100 text-slate-600 align-top; }
.kb-prose tr:nth-child(even) td { @apply bg-slate-50; }
.kb-prose .callout-warning { @apply bg-amber-50 border border-amber-200 rounded-lg p-4 mb-4 text-sm text-amber-800; }
.kb-prose .callout-info    { @apply bg-blue-50 border border-blue-200 rounded-lg p-4 mb-4 text-sm text-blue-800; }
.kb-prose .callout-tip     { @apply bg-green-50 border border-green-200 rounded-lg p-4 mb-4 text-sm text-green-800; }
```

---

## `ArticlePage.tsx` — TOC Extraction

```typescript
// Use Array.from to avoid TS downlevelIteration error
function extractHeadings(html: string) {
  const matches = Array.from(html.matchAll(/<h([123])[^>]*>(.*?)<\/h[123]>/gi));
  return matches.map((m, i) => ({
    level: parseInt(m[1]),
    text: m[2].replace(/<[^>]+>/g, ""),
    id: `heading-${i}`,
  }));
}

function injectIds(html: string) {
  let i = 0;
  return html.replace(/<h([123])([^>]*)>/gi, (_match, level, attrs) => {
    return `<h${level}${attrs} id="heading-${i++}">`;
  });
}
```
