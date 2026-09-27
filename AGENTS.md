# Agent contract: adding notes

This repo stores knowledge notes extracted from X (Twitter) articles and long threads. Agents may open pull requests that add or update notes. Follow this contract exactly.

## When to create a PR

Create a PR when you successfully extract a note from a concrete X article or thread URL and have something worth committing.

Do **not** create a PR when:

- The source URL cannot be fetched or parsed
- Extraction fails or content is incomplete/ambiguous
- You would need to invent, guess, or pad the note

**Fail closed:** if the source cannot be extracted, do nothing. Leave no empty PR, placeholder note, or invented content.

## One note per source

- One markdown file per X article or thread
- Do not merge unrelated sources into a single note
- If updating an existing note for the same source URL, edit that file in place rather than creating a duplicate

## File naming

Place notes under `notes/`:

```
notes/YYYY-MM-DD-short-slug.md
```

Rules:

- `YYYY-MM-DD` is the source publication date when known; otherwise use the extraction date
- `short-slug` is lowercase kebab-case derived from the title (ASCII, no spaces)
- Keep the slug short and stable; do not rename casually after merge
- Never put secrets, tokens, or personal identifiers in filenames

Example: `notes/2026-09-15-shipping-better-agents.md`

## Required markdown sections

Use `notes/_template.md` as the structure. Every committed note must include these sections (use `N/A` only when a section truly does not apply after successful extraction):

1. **Source** — single section with:
   - **Author** — X username of the post/article author (`@handle`)
   - **Article/Post** — canonical X URL (primary attribution)
   - **Date** — publication or extraction date (`YYYY-MM-DD`)
   - **Title** — title of the article/thread
2. **Long summary** — faithful overview of the source
3. **Key claims** — bullet list of the main points asserted
4. **Links** — URLs referenced in the source (if any)
5. **Actionables** — concrete follow-ups implied by the source (if any)

Do not invent claims, links, or actionables that are not supported by the source.

## Content rules

- Summarize; do not paste the full source verbatim unless needed for a short quote
- No PII, credentials, API keys, session tokens, or private account details
- Stay faithful to the source; mark uncertainty instead of fabricating detail

## PR expectations

- One PR per note when practical (small related batches are OK)
- Title/body should name the source and say whether this is a new note or an update
- Keep the diff limited to the note file(s) plus any unavoidable metadata
- If extraction fails mid-work, close out without committing invented content
