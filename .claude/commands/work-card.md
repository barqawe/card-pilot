---
description: Start building a Trello card in this repo, matched to its Figma design
argument-hint: "<trello short link>"
---

# Work on card $ARGUMENTS

## Hard rules

- **No Trello changes without my explicit yes.** Before any Trello write (moving a card,
  commenting, labels, checklists, attachments, members), show me the exact change and wait.
  One yes covers one change. If I say no or don't answer, leave Trello untouched.
- **Never push, force-push, open a PR, or merge** without asking me first.
- **Don't edit Figma.**
- **Stay in scope.** If you spot unrelated problems, list them at the end instead of fixing them.

## Steps

1. **Re-check the card.** Read card `$ARGUMENTS` from Trello again (description, checklists,
   comments, attachments). Compare with `.claude/card-board/triage-data.js`. If the card is
   marked blocked or needs backend, or something changed since triage (new comments, new
   links, moved lists), tell me what and ask whether to continue.

2. **Read the design.** From the card's Figma link, get the design context for that node,
   the variables/tokens it uses, and a screenshot. Check Code Connect mappings if any exist.
   Map every element in the design to an existing component or token in this repo, and list
   anything with no match.

3. **Branch.** Read `.claude/card-board/config.md` for the default branch and commands. If
   the working tree isn't clean, stop and ask me. Otherwise create
   `feature/$ARGUMENTS-<short-slug>` from the default branch.

4. **Plan, then wait.** Show me a short plan: files to add or change, components reused,
   API calls used, states covered (loading, empty, error), and open questions. Wait for my
   go-ahead before writing code.

5. **Build** the way this repo already works. Read neighboring code first, then:
   - use the existing API client and data-fetching patterns,
   - reuse existing components; use tokens instead of hard-coded colors, spacing, and fonts,
   - follow the repo's i18n and RTL setup if it has one,
   - cover loading, empty, and error states,
   - keep it accessible: labels, focus order, keyboard use, contrast.

6. **Verify.**
   - Run the typecheck, lint, and test commands from config.
   - If a dev server and browser tooling are available, screenshot the result at the
     design's frame size, compare it with the Figma screenshot, and fix differences in
     spacing, type, color, and alignment.
   - If you can't take screenshots, give me a short list of things to check by eye.

7. **Wrap up.**
   - Summarize what changed, what you verified, and anything left undone.
   - Then list proposed Trello changes, one per line, for example:
     - Move "Order history page" from **To do** to **In progress**
     - Comment: "Started on branch feature/a1B2c3D4-order-history"
   - Ask which ones to apply, and apply only those I approve.
