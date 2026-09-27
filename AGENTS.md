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
   - **Author** — X username of the post/article author as a clickable link: `[@handle](https://x.com/handle)` (bare `@handle` is not a link in GitHub markdown)
   - **Article/Post** — canonical X URL (primary attribution). When both a status/share URL and an X article URL exist, this **must** be the `https://x.com/i/article/...` URL
   - **Status** — optional; the status/share URL (`https://x.com/.../status/...`) when it differs from Article/Post
   - **Date** — publication or extraction date (`YYYY-MM-DD`)
   - **Title** — title of the article/thread
2. **Long summary** — faithful overview of the source
3. **Key claims** — bullet list of the main points asserted
4. **Actionables** — concrete, source-derived follow-ups for a general reader (or `N/A`). Do not personalize for a named person.
5. **References** — destination URLs a reader can open for real content (docs, GitHub repos, Hugging Face, product pages, blogs, and canonical X URLs — status or `https://x.com/i/article/...` — when they belong in References). Group when documenting expected shape (e.g. X source / Docs / GitHub / HF / Other).

   **Do not** put in References:
   - X/Twitter shorteners (`t.co`, `https://t.co/...`)
   - Raw media / CDN asset URLs (`pbs.twimg.com`, `.jpg`, `.png`, `.gif`, `.webp`, video/media CDN paths)
   - Tracking or redirect junk that is not a readable page

   If the source has no useful references beyond what is already in **Source**, use `N/A` under `## References` (or omit empty groups). Do not pad with media or shorteners.

Do not invent claims, references, or actionables that are not supported by the source.

## Content rules

- Summarize; do not paste the full source verbatim unless needed for a short quote
- **Exactness:** copy names, numbers, and product/model labels exactly as in the source; do not substitute newer or guessed names
- No PII, credentials, API keys, session tokens, or private account details
- Stay faithful to the source; mark uncertainty instead of fabricating detail

## PR expectations

- One PR per note when practical (small related batches are OK)
- Title follows Conventional Commits (see below)
- PRs must be **ready for review** (not draft). After creating the PR, mark it ready if the platform opened it as draft.
- **Body must follow** [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md):
  - **Summary** — fill Source URL and note path
  - **Motivation** — optional; only when it adds useful context
  - **Checklist** — mark items honestly (`[x]` only when true; leave unchecked when not applicable or unmet)
- Do **not** put meta or agent chatter in the PR body — only what the template asks for.
- Keep the diff limited to the note file(s) plus any unavoidable metadata
- If extraction fails mid-work, close out without committing invented content

## Conventional Commits

Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages and PR titles.

For **new or updated notes**:

- `docs(article): <article title>` — source is an X article
- `docs(thread): <thread subject>` — source is a thread
- Do **not** put `@handle` in the title; author belongs only in the note Source block

For **other changes**:

- `docs(contract):` — note contract / agent guidance (`AGENTS.md`, section rules)
- `chore:` — templates, meta, tooling, and non-content scaffolding

Examples:

- `docs(article): Shipping Better Agents`
- `docs(thread): Notes on long-context evals`
- `docs(contract): put References after Actionables`
- `chore: add pull request and issue templates`

Keep the subject line short; put detail in the body when needed.
