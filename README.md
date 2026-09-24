# Wardrobe Ledger

A personal closet inventory that runs entirely in the cloud as a Claude artifact. It works from any browser signed in to claude.ai, and nothing runs on a local machine.

- **My live copy:** https://claude.ai/artifact/7xQC1kFbFuVqE2jUba7kKY (private, so only I can open it)
- **Source:** `index.html` (published to the artifact with the capabilities below)

## Use it yourself

The page runs as a Claude artifact, so there's no server to host and nothing to install. You need a Claude account, and for the weekly Gmail check you also need Claude Code with the Gmail connector.

1. **Publish the page.** In Claude Code, give Claude `index.html` from this repo and ask it to publish the file as an artifact with these capabilities: `db`, `assets` and `sample` (see the JSON below). You'll get a private link that works from any browser where you're signed in to Claude.
2. **Add your clothes.** Use **+ Add item** on the page, or give Claude a spreadsheet or list and ask it to write each item into the artifact's `items` collection, using the data model below.
3. **Turn on the weekly Gmail check (optional).** Create a Claude Code routine that runs once a week with the Gmail connector attached. Use the prompt in [`routine-prompt.md`](routine-prompt.md) with your artifact link in place of `YOUR_ARTIFACT_URL`.

Your data stays in your own artifact. Nobody else can see it unless you share the page.

## What it does

- **Closet**: a photo grid grouped by category, with filters for brand, occasion and color family, plus sorting by name, newest or price. Tap an item to see its details, edit it, add or replace a photo, mark it listed for sale, mark it as gone, or delete it.
- **Add item**: a form with an optional photo. "Fill in details from photo" asks Claude to identify the brand, type, category and colors.
- **For sale**: items currently listed on a resale site, with where they're listed and the asking price.
- **Review**: a tray where the weekly Gmail routine drops new purchases and "your item sold" notices. Nothing reaches the closet until you tap Add to closet or Mark as gone.
- **Gone**: sold, donated, gifted, returned or tossed items, with a resale earnings total. They can be restored for a year and are then deleted for good by the weekly routine.
- **Bulk select**: select several items and mark them listed or gone in one step.

## Capabilities declared at publish

```json
{
  "db": { "rules": [{ "path": "", "read": "view", "write": "admin" }] },
  "assets": {},
  "sample": {}
}
```

- `db`: the shared document store holding the `items` and `pending` collections.
- `assets`: photo storage. Photos are shrunk to 1400px JPEG before upload and referenced as `/_blob/<id>`.
- `sample`: lets the page ask Claude to read a photo when you add an item.

## Data model

### `items/<item-NNNN>`

| Field | Notes |
|---|---|
| `itemDescription` | Display name |
| `category` | Clothing, Shoes, Accessories, Handbags, Intimates |
| `item` | Sub-type (Sneakers, Blazer, Tote…) |
| `brand`, `colors`, `size`, `attire`, `approxPrice`, `links`, `notes`, `notes2` | Free text |
| `imageUrl` | `/_blob/<32-hex asset id>`, optional |
| `addedAt`, `addedVia` | ISO timestamp; `page`, `weekly-task`, `migration`, `manual` |
| `status` | `active` (default when missing), `listed`, `gone` |
| `listedAt`, `listing` | `{ where, askingPrice }` |
| `goneAt`, `gone` | `{ reason, date, salePrice, where }`; reason is Sold / Donated / Gave away / Returned / Tossed |

### `pending/<p-…>`

- `kind: "new"`: `{ source, emailSubject, foundAt, data: { …item fields } }`
- `kind: "sold"`: `{ source, emailSubject, foundAt, targetId, summary, gone: { reason, date, salePrice, where } }`

## Automation

The weekly routine ([`routine-prompt.md`](routine-prompt.md)) runs every Sunday in a fresh cloud session with the Gmail connector. It:

1. Adds new purchase confirmations to `pending` as `kind: "new"`.
2. Adds resale "item sold" emails to `pending` as `kind: "sold"`, matched to an item when the match is confident.
3. Deletes items whose `goneAt` is more than 365 days old, along with their photos.

It never writes to `items` except for that one-year cleanup.

## Editing from chat

From any Claude chat that has the ArtifactData tool, you can say things like:

- "Mark the Gucci loafers as sold on Poshmark for $400."
- "Tag all Veronica Beard blazers as Business."

Claude updates the `items` documents on the artifact above.
