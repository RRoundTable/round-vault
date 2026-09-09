# round-vault

Personal Obsidian vault.

## Structure

| Folder | Purpose |
| --- | --- |
| `00-Inbox` | Quick capture and daily notes. Nothing stays here long. |
| `10-Notes` | Permanent, atomic notes — the actual knowledge base. |
| `20-Projects` | Active work with an outcome and an end date. |
| `30-References` | Source material: papers, docs, book and article notes. |
| `90-Attachments` | Images and files. Obsidian saves attachments here automatically. |
| `Templates` | Note templates used by the core Templates plugin. |

## Opening the vault

Obsidian → *Open folder as vault* → select this directory.

## Syncing

Plain git. `.obsidian/workspace.json` and other per-machine state are gitignored,
so open panes and window layout stay local while settings and plugins sync.

```bash
git pull --rebase
git add -A && git commit -m "notes: ..." && git push
```
