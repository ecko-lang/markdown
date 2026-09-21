# Markdown - Ecko Std Lib Package

Parse Markdown into a block list, render it to HTML, and pull out the headings,
code blocks and tables a docs pipeline or a RAG ingest actually wants. Pure - no
capabilities.

## Install

```bash
ecko get github.com/ecko-lang/markdown
```

## Usage

```ecko
import markdown

doc = fs.read("guide.md")

# Walking is a for loop over blocks. There is no visitor to learn.
for b in markdown.parse(doc) {
    print(b.kind)
}

# A table of contents.
for h in markdown.headings(doc) {
    print(h.level, h.text)
}

# Every shell command in the document.
for c in markdown.code_blocks(doc) {
    if c.lang == "sh" { print(c.code) }
}

# Tables as data.
for t in markdown.tables(doc) {
    for row in t.rows { print(row[0], row[1]) }
}

# Chunks for embedding: each one carries the heading it sits under, so a
# retrieved chunk still says what it is about.
for s in markdown.sections(doc) {
    embed(s.heading + "\n\n" + s.body)
}

markdown.to_html(doc)
```

## API

| function | returns |
|---|---|
| `parse(src)` | every block, in document order |
| `to_html(src)` | HTML, with the source escaped |
| `headings(src)` | `{ level, text }` per heading |
| `code_blocks(src)` | `{ lang, code }` per fenced block |
| `tables(src)` | `{ headers, rows }` per table |
| `text(src)` | plain text, markup removed, code dropped |
| `sections(src)` | `{ heading, level, body }` per heading |

A block from `parse` always has a `kind`, plus the fields that go with it:

| kind | fields |
|---|---|
| `heading` | `level` (1-6), `text` |
| `paragraph` | `text` |
| `code` | `lang` (may be `""`), `code` |
| `list` | `ordered`, `items` |
| `table` | `headers`, `rows` |
| `quote` | `text` |
| `rule` | - |

## Notes

**This is not CommonMark.** It is a block-level subset chosen for what
documentation tooling does: find the structure, pull out the code and tables,
produce clean text to embed. If you need a spec-compliant parser for arbitrary
Markdown from the internet, this is the wrong package and you will find the
disagreements quickly.

What it handles: ATX headings (`#` through `######`), fenced code with a
language tag, bullet and ordered lists, pipe tables, block quotes, thematic
breaks, paragraphs, and the inline spans `**bold**`, `*italic*`, `` `code` ``
and `[text](url)`.

What it does not: setext headings (`===` underlines), nested or lazy lists,
reference links, HTML blocks, footnotes, and inline spans that straddle a
line break. A link whose target contains brackets or unbalanced parentheses
will end at the first `)` - `[a](http://x/(y))` does not round-trip.

**Rendering someone else's Markdown is safe by default.** Text is escaped
before any tag is added, so `<script>` in the source comes out as
`&lt;script&gt;` - in headings, paragraphs, link text and code blocks alike.
Link targets are checked too: `http`, `https`, `mailto` and anything without a
scheme become an `href`, and everything else - `javascript:`, `data:`,
`vbscript:` - renders as plain text, keeping the words and dropping the link.

Unmatched emphasis stays literal. `a **b` renders as `a **b`, not as an empty
`<strong>`, because inventing a tag the author did not write is worse than
leaving their characters alone.

## Testing

```bash
ecko test
```

Offline and deterministic - no network, no clock, no filesystem.

## License

MIT
