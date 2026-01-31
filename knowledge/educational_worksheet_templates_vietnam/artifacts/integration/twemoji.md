# Twemoji Integration in Educational Content

Visual aids (emojis) are used extensively in Grade 2 worksheets to represent math objects (apples, fish, flowers, etc.).

## Integration Method

The worksheets use the Twemoji CDN and parse the document on load to ensure high-quality SVG emojis that print well.

### 1. Script Inclusion
```html
<script src="https://cdn.jsdelivr.net/npm/@twemoji/api@latest/dist/twemoji.min.js"></script>
```

### 2. Initialization Script
Placed at the end of the `<body>`:
```html
<script>
    document.addEventListener('DOMContentLoaded', () => 
        twemoji.parse(document.body, { folder: 'svg', ext: '.svg' })
    );
</script>
```

## Emoji Styling

Since Twemoji replaces emoji characters with `<img>` tags, specific styling is needed to control their size.

```css
img.emoji {
    height: 1.1em;
    width: 1.1em;
    vertical-align: -0.1em;
}

/* Larger version for object groups */
.lg img.emoji {
    height: 1.6em;
    width: 1.6em;
}
```

## Usage Patterns
Emojis are often wrapped in `.obj` and `.lg` classes within `.grp` (group) containers to create visual representations of sets.
