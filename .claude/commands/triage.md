---
description: Triage my Trello cards against this repo and Figma, then build the card map
argument-hint: "[optional board name]"
---

# Triage my Trello cards

Find the Trello cards assigned to me, decide for each one whether it is **ready** for
frontend work, **needs backend** work, or is **blocked**, and write the results for the
card map at `.claude/card-board/board.html`.

## Hard rules

- **Trello is read-only in this command.** Never call a Trello tool that creates, updates,
  moves, archives, labels, or comments on anything. If a Trello change would help, list it
  under "Suggested Trello changes" in the final summary. Do not apply it.
- **Figma is read-only.** Never call a tool that edits a Figma file.
- **Don't touch source code.** The only file you may write is `.claude/card-board/triage-data.js`.
- **Never create, edit, or regenerate any HTML file.** The map (`.claude/card-board/board.html`)
  is already provided and reads `triage-data.js`. If `board.html` is missing, write the data
  file anyway, then tell me to copy `board.html` from the kit. Don't build your own page.

## Steps

1. **Config.** Read `.claude/card-board/config.md`. If something you need is blank, ask me,
   then save my answer into that file.

2. **Collect cards.** Get my Trello member, then every open card assigned to me on the
   configured boards (or only on the board named "$ARGUMENTS", if I passed one).
   - Include **only cards where I am a member**. Unassigned cards and cards assigned only to
     other people are not mine.
   - Include **only cards in the lists named under "Lists to include"** in config. If that
     field is empty, include every list except the done ones.
   - Skip cards in the lists that count as done.
   - Skip Trello's own onboarding/template cards (for example, anything in a "Trello Starter
     Guide" list or board). They aren't work items. For each card read the description, checklists,
   comments (newest first), attachments and links, labels, due date, and linked cards.

3. **Map the repo once, before judging any card.**
   - Find the API layer: the folders in config, plus a search for fetch/axios/ky clients,
     React Query / SWR / RTK Query hooks, GraphQL operations, OpenAPI/Swagger files, and
     mock handlers (e.g. MSW).
   - Build a list of the endpoints/operations the frontend can already call, with the file
     each one lives in.
   - Note where shared components and design tokens live.

4. **Judge each card.**
   - **Needs backend**: the card needs data or an action that no existing endpoint or
     operation provides. Describe the missing endpoint as precisely as you can (method,
     path, fields) and mention where you looked.
   - **Blocked**: any of these is true:
     - it depends on a card that isn't done,
     - a question in the comments hasn't been answered,
     - it has a "blocked" (or similar) label,
     - it changes UI but has no design linked,
     - the requirements are unclear or contradict each other.
   - **Ready**: none of the above.
   - If a card is both blocked and needs backend, mark it **blocked** and still list the
     backend needs.
   - Reasons must be concrete: the file where an endpoint was found, the commenter and
     date of an open question, the name and list of a blocking card.

5. **Design check (light).** If a card has a Figma link, make one metadata read to confirm
   the frame exists and capture its name. Don't pull full design context here;
   `/work-card` does that.

6. **Connections.**
   - `dependsOn`: the card explicitly links to, mentions, or clearly builds on another card.
   - `related`: the card shares an endpoint or a major component with another of my cards.
   - Cards that block mine but aren't assigned to me go in `external`.

7. **Write `.claude/card-board/triage-data.js`** (overwrite it). It must be valid
   JavaScript in exactly this shape:

   ```js
   window.TRIAGE_DATA = {
     generatedAt: "2026-09-10T09:30:00Z",   // ISO time of this run
     member: "Your name",
     cards: [
       {
         id: "a1B2c3D4",                    // Trello short link
         name: "Order history page",
         url: "https://trello.com/c/a1B2c3D4",
         board: "Storefront",
         list: "To do",
         labels: ["frontend"],
         due: null,                         // ISO date or null
         estimate: "M",                     // S, M, L, or null
         status: "ready",                   // "ready" | "needs_backend" | "blocked"
         summary: "One or two plain sentences on what the card asks for.",
         reasons: ["GET /api/orders already exists in src/api/orders.ts"],
         backendNeeds: [                    // empty if none
           { method: "GET", endpoint: "/api/orders/:id/timeline", note: "Not found in src/api" }
         ],
         blockers: [                        // empty if none
           { type: "card", detail: "Waiting on 'Payment methods API' (Backend, In progress)", cardId: "x9Y8z7W6" }
           // type is one of: "card", "question", "design", "label", "unclear"
         ],
         dependsOn: [],                     // ids of cards this one needs first
         related: [],                       // ids of cards sharing endpoints/components
         figma: { url: "https://www.figma.com/design/...?node-id=12-340", name: "Orders / List" }, // or null
         likelyFiles: ["src/pages/orders/OrderHistory.tsx"]
       }
     ],
     external: [
       {
         id: "x9Y8z7W6",
         name: "Payment methods API",
         url: "https://trello.com/c/x9Y8z7W6",
         board: "Backend",
         list: "In progress",
         assignee: "Omar"
       }
     ]
   };
   ```

8. **Summary in chat.**
   - A short table: card, status, one-line reason.
   - "Suggested Trello changes": proposals only, each written as the exact change
     (e.g. *add comment on "Profile avatar upload": "@Lina what's the max file size?"*).
     Don't apply any of them.
   - Tell me to open (or refresh) `.claude/card-board/board.html`. Offer to open it for me.
