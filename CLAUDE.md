# LLM Wiki

A personal knowledge base maintained by Claude Code.
Based on Andrej Karpathy's LLM Wiki pattern.

## Purpose

This wiki is a structured, interlinked knowledge base for planning a trip to Japan.
Claude maintains the wiki. The human curates sources, asks questions, and guides the analysis.

## Folder structure

```
raw/          -- source documents (immutable -- never modify these)
wiki/         -- markdown pages maintained by Claude (English)
wiki/index.md -- table of contents for the entire wiki
wiki/log.md   -- append-only record of all operations
wiki/el/      -- Greek translations of all wiki pages (mirrors wiki/)
wiki/el/index.md -- Greek table of contents
wiki/el/log.md   -- append-only record of translation operations
```

## Ingest workflow

When the user adds a new source to `raw/` and asks you to ingest it:

1. Read the full source document
2. Discuss key takeaways with the user before writing anything
3. Create a summary page in `wiki/` named after the source
4. Create or update concept pages for each major idea or entity
5. Add wiki-links ([[page-name]]) to connect related pages
6. Update `wiki/index.md` with new pages and one-line descriptions
7. Append an entry to `wiki/log.md` with the date, source name, and what changed

A single source may touch 10-15 wiki pages. That is normal.

## Page format

Every wiki page should follow this structure:

```markdown
# Page Title

**Summary**: One to two sentences describing this page.

**Sources**: List of raw source files this page draws from.

**Last updated**: Date of most recent update.

---

Main content goes here. Use clear headings and short paragraphs.

Link to related concepts using [[wiki-links]] throughout the text.

## Related pages

- [[related-concept-1]]
- [[related-concept-2]]
```

## Citation rules

- Every factual claim should reference its source file
- Use the format (source: filename.pdf) after the claim
- If two sources disagree, note the contradiction explicitly
- If a claim has no source, mark it as needing verification

## Question answering

When the user asks a question:

1. Read `wiki/index.md` first to find relevant pages
2. Read those pages and synthesize an answer
3. Cite specific wiki pages in your response
4. If the answer is not in the wiki, say so clearly
5. If the answer is valuable, offer to save it as a new wiki page

Good answers should be filed back into the wiki so they compound over time.

## Lint

When the user asks you to lint or audit the wiki:

- Check for contradictions between pages
- Find orphan pages (no inbound links from other pages)
- Identify concepts mentioned in pages that lack their own page
- Flag claims that may be outdated based on newer sources
- Check that all pages follow the page format above
- Report findings as a numbered list with suggested fixes

## Translation workflow (wiki/el/)

`wiki/el/` is a full Greek-language mirror of `wiki/`. When creating or updating any wiki page:

1. Write or update the English page in `wiki/` first
2. Create or update the corresponding Greek translation in `wiki/el/` with the same filename
3. Translate all prose, headings, table content, and callouts to Greek
4. Retain English technical acronyms where standard in Greek medical practice (e.g. ARDS, TBI, MAP, SpO₂, ATP, IV, IO, GCS), noting the Greek expansion on first use
5. Keep `[[wiki-links]]` pointing to the same filenames (links resolve language-agnostically)
6. Update `wiki/el/index.md` and append to `wiki/el/log.md` after each translation session

## Reading PDF files from raw/

`python` and `python3` are not available in this environment. Use Node.js with `pdfjs-dist` instead.

### Setup (one-time, already done)

```bash
npm install pdfjs-dist
```

`pdfjs-dist` is installed at the repo root (`node_modules/`). Do not reinstall it.

### Reading a page range

```js
node --input-type=module << 'EOF'
import { getDocument } from 'pdfjs-dist/legacy/build/pdf.mjs';
import { writeFileSync } from 'fs';

const PDF_PATH = 'C:/Users/KNOWLEDGE/Nextcloud2/llm-wiki/raw/YOUR_FILE.pdf';
const pdf = await (await getDocument(PDF_PATH).promise);
console.log('Total pages:', pdf.numPages);

let text = '';
for (let i = START_PAGE; i <= END_PAGE; i++) {
  const page = await pdf.getPage(i);
  const content = await page.getTextContent();
  const pageText = content.items.map(item => item.str).join(' ');
  text += `\n--- PDF PAGE ${i} ---\n` + pageText;
}

// Write to a temp file if the output is large, then read in chunks with bash
writeFileSync('C:/Users/KNOWLEDGE/Nextcloud2/llm-wiki/tmp_extract.txt', text);
console.log('Written, length:', text.length);
EOF
```

### Reading the temp file in chunks (when too large for Read tool)

```bash
head -c 15000 tmp_extract.txt        # first chunk
sed -n '20,40p' tmp_extract.txt      # specific lines
tail -c 8000 tmp_extract.txt         # last chunk
```

### Finding chapter page ranges

1. Read PDF pages 6–10 to find the table of contents
2. Note book-page numbers for each chapter from the TOC
3. Check the offset: for PHTLS 10th Ed., book page N = PDF page N+27
4. Extract the chapter's PDF page range using the script above

### Cleanup

Delete `tmp_extract.txt` when done. Do not commit it.

## Rules

- Never modify anything in the `raw/` folder
- Always update `wiki/index.md` and `wiki/log.md` after changes in `wiki/`
- Always update `wiki/el/index.md` and `wiki/el/log.md` after changes in `wiki/el/`
- Keep page names lowercase with hyphens (e.g. `machine-learning.md`) — same filenames in both `wiki/` and `wiki/el/`
- Write in clear, plain language
- When uncertain about how to categorize something, ask the user
