# 代码高亮

```bash
npm install markdown-it-highlightjs --save
```

```typescript
import HighLight from 'markdown-it-highlightjs'
let md = new MarkdownIt({
  breaks: true,
  html: true,
  linkify: true,
  typographer: true
});
HighLight(md, {});
```

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/highlightjs/cdn-release@11.5.1/build/styles/atom-one-dark.min.css"/>
```



# Emoji

```bash
npm install markdown-it-emoji --save
```

```typescript
import Emoji from 'markdown-it-emoji'

Emoji(md, {});
```

```markdown
# 即将Unicode的值 中的 U+ 替换为 &#x，并加;结尾
U+1F607 -> &#x1F607
```



# Katex

```bash
npm install katex --save
npm install @traptitech/markdown-it-katex --save
```

```typescript
import Katex from '@traptitech/markdown-it-katex'
Katex(md, {});
```

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex/dist/katex.min.css"/>
```

