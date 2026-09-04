# Confluence sync

## Live page

- **URL:** https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/4394024967/Mobile+app+UI+performance+improving+ideas
- **Page ID:** `4394024967`
- **GitHub notes repo:** https://github.com/rafal-tt/MobileUiPerformance

## Mirror files (for diffs)

| File | Role |
| --- | --- |
| [`4394024967-body.html`](4394024967-body.html) | Exact HTML last published (or intended) to Confluence |
| [`4394024967-Mobile-app-UI-performance-improving-ideas.md`](4394024967-Mobile-app-UI-performance-improving-ideas.md) | Human-readable markdown mirror of the same sections |

## Sync markers

Sections are bounded by plain text markers (survive Confluence HTML round-trip better than HTML comments):

```text
sync-begin:<section-id>
...
sync-end:<section-id>
```

| Section id | Content |
| --- | --- |
| `scope` | In/out of scope |
| `constraints` | App constraints |
| `problems` | Problems to solve |
| `options-considered` | MAUI-hosted options + pros/cons |
| `options-parking` | Full-migration parking lot |
| `recommendation` | Recommendation + next steps |
| `footer` | Repo / related links |

## Workflow

1. **Edit in Confluence** → fetch page → update mirror files → `git diff confluence/`
2. **Edit in repo** → update both mirrors (md + html) → publish with `updateConfluencePage` using `4394024967-body.html`
3. Never remove `sync-begin` / `sync-end` markers when editing either side
4. Do not put local filesystem paths in the published page; link the GitHub repo instead
