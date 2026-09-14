# Card Pilot

Claude reads the Trello cards assigned to you, checks each one against your frontend code
and your Figma designs, and shows them as a bubble map: **ready to build**, **needs backend**,
or **blocked**. Pick a ready card, press **Start working**, and Claude builds it from the design.

**Claude never changes anything in Trello unless you say yes.**

---

## What it does

| Command | What happens |
|---|---|
| `/triage` | Reads your cards, searches your repo for the API calls each one needs, checks the Figma links, and finds blockers. Writes the results for the map. |
| `board.html` | The bubble map. Click a card to see why it got its status. Press **Start working** to copy its command. |
| `/work-card <id>` | Builds that one card: reads the Figma frame, makes a branch, shows you a plan, waits for your OK, then writes the code. |

A card is **ready** when the data it needs already exists in your code and a design is linked.
It **needs backend** when an endpoint is missing. It's **blocked** when another card, an
unanswered question, or a missing design is in the way.

---

## Requirements

- [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed
- A Trello account that is a **full member** (not a guest) of the workspace with your board
- A Figma account, if your cards link to designs
- Your frontend project, in a git repo

---

## Install

**1. Copy the files into your project**

Put the `.claude` folder in the root of your project.

Already have a `.claude` folder? Don't replace it. Copy the pieces in:

- `commands/triage.md` and `commands/work-card.md` → your `.claude/commands/`
- the whole `card-board/` folder → your `.claude/`
- `settings.json` → save it as `.claude/settings.local.json` (**don't overwrite your own
  `settings.json`**). If you already have a `settings.local.json`, copy the `allow`, `ask`,
  and `deny` lists into it.

**2. Keep it out of git** (optional, but these are personal files)

```bash
echo ".claude/commands/triage.md"      >> .git/info/exclude
echo ".claude/commands/work-card.md"   >> .git/info/exclude
echo ".claude/card-board/"             >> .git/info/exclude
echo ".claude/settings.local.json"     >> .git/info/exclude
```

`.git/info/exclude` works like `.gitignore` but is never pushed, so your teammates see nothing.

**3. Connect Trello and Figma**

```bash
claude mcp add --transport http trello https://mcp.trello.com/v1
claude mcp add --transport http figma  https://mcp.figma.com/mcp
```

Keep the names `trello` and `figma`; the permission rules depend on them.

Then start Claude Code (`claude`), type `/mcp`, pick each server, and sign in in the browser.
On the Trello screen, the dropdown lists **workspaces, not boards** — pick the workspace your
board lives in.

**4. Fill in `.claude/card-board/config.md`**

Your board name, which lists to scan, where your API code lives, and your lint/test commands.
Anything you leave blank, `/triage` asks about once and saves.

> Filling this in is the difference between a triage that takes minutes and one that takes
> seconds. Without it, Claude searches your whole project every run.

---

## Daily use

1. `/triage` — or `/triage <board name>` for one board
2. Open `.claude/card-board/board.html` in your browser (or ask Claude Code to open it).
   Refresh it after each triage.
3. Click a bubble → see why it's ready or blocked, what backend work is missing, the design
   link, and the files it will likely touch.
4. Press **Start working** → copies `/work-card <id>` → paste it into Claude Code.
5. Claude shows a plan and waits. Say go, and it builds on a new branch.
6. At the end, it lists proposed Trello changes. Only the ones you approve get applied.

### Reading the map

- **Glowing indigo** = ready to build · **yellow** = needs backend · **red** = blocked
- **Dashed grey** = a card that blocks yours but belongs to someone else
- **Arrows** point from a card to the cards it unlocks. Bigger bubble = unlocks more, so it's
  usually the best place to start.
- **Dashed lines** connect related cards (shared endpoint or component).
- Chips at the top filter by status. Bubbles can be dragged.

---

## How your approval is protected

Two separate layers, so it doesn't rely on Claude behaving:

1. **The commands** tell Claude that Trello is read-only in `/triage`, and that `/work-card`
   must show every proposed Trello change and wait for a yes.
2. **The permission rules** in `settings.local.json` allow only Trello's *read* tools to run
   freely. Any tool that changes Trello triggers a prompt. Figma edit tools are denied
   outright, and `git push` / opening a PR always asks.

When a Trello prompt appears, answer **Yes** for that one change — don't pick "don't ask
again". And don't run Claude Code with `--dangerously-skip-permissions` for this workflow.

Want it stricter? Uncheck **Write** when authorizing Trello, or move the Trello write tools
into the `deny` list. Then Claude can only suggest changes, and you apply them yourself.

---

## Troubleshooting

**My board isn't in the Trello dropdown.**
That list shows workspaces, not boards. Pick the workspace that contains your board — open
the board in Trello and check the name in the top left. If it's still missing, you're likely
a *guest* on that board; ask an admin to make you a full workspace member.

**Triage is slow / uses a lot of tokens.**
It's scanning your repo to find your API layer. Fill in `config.md`, run `/clear` before
`/triage`, and run it once a morning rather than after every change. `/work-card` is much
lighter.

**It showed me a plain list instead of the bubble map.**
Claude generated its own page. Make sure `.claude/commands/triage.md` is the version from
this kit (it forbids creating HTML) and that `board.html` from the kit is in
`.claude/card-board/`.

**I see a yellow "sample cards" box.**
`/triage` hasn't written its data yet. Check that `triage-data.js` sits next to `board.html`.

**Trello auth broke.**
`/mcp` → trello → authenticate. Or reconnect:
`claude mcp remove trello` then the `claude mcp add` line above.

**Onboarding cards showed up as blocked.**
Those are Trello's starter-guide templates. The current `triage.md` skips them and anything
not assigned to you.

---

## Files

```
.claude/
  settings.json          → copy as settings.local.json (permission rules)
  commands/
    triage.md            /triage
    work-card.md         /work-card
  card-board/
    board.html           the bubble map — open this in a browser
    config.md            your boards, paths, and commands
    triage-data.js       written by /triage (your card data — keep it out of git)
```

---

## Sharing it

Send someone the whole folder. Each person connects their own Trello and Figma accounts, so
nothing of yours is shared. You can pass along your filled-in `config.md` if you work on the
same project — but not `triage-data.js`, which is your cards.

---

## Notes

- Tested with a Vue frontend; it reads whatever patterns your repo already uses, so React,
  Svelte, and others work the same way.
- The map supports Arabic and other right-to-left card titles.
- Until you run `/triage`, the map shows sample cards so you can try the interface.
