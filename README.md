# x-article-wiki

Notes from X articles and threads.

This repo is a small public wiki of knowledge notes extracted from X (Twitter) articles and long threads. Each note captures one source so the content stays easy to review, search, and update over time.

## What lives here

- One markdown note per X article or thread
- Notes are added and updated through pull requests
- Agents open those PRs after extracting content from a source URL

## Layout

```
notes/           # committed notes (one file per source)
notes/_template.md
AGENTS.md        # contract for agents that add notes
```

## Contributing

Humans and agents should follow `AGENTS.md`. Prefer a PR per note (or a small related set). Do not invent content when a source cannot be extracted.
