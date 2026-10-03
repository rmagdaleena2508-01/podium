# v1 audit — what lies, what is missing, what to decide

A pass through every page as a user, then as the person who has to maintain it.
Grouped by how much it matters. Nothing here is fixed yet.

---

## Part 1 — Things that say one thing and do another — **all ten fixed**

Fixed. What each one does now is in the last column.

| # | Where | It said | It did | It now does |
|---|---|---|---|---|
| 1.1 | Landing header, closing line, FAQ, onboarding page | "Sign in **with Google**" | Signs nobody in at all, and you asked for the words to be just "Sign in" | Says "Sign in", and the onboarding button says "Continue". The provider stays unnamed until it works |
| 1.2 | FAQ, "Who can vote?" | "you get one vote in each group" | The rule changed: back as many decks as you like, once each, and a vote is final | States the real rule: back as many decks as you like, once each, a vote cannot be taken back, and you cannot vote for your own team |
| 1.3 | Hero, "Open event" with a code | Implies the code opens **that** event | The code's shape is checked and then thrown away. Any six valid characters land you in the same single event | Matches the code against the event and refuses anything else: "No event with the code ZZZZZZ." With no event set up it says so and offers to run one |
| 1.4 | Landing brand mark, top left and footer | Podium home | `href="/"`, which on GitHub Pages is the **domain root**, not this project — a 404 for every visitor who clicks the logo | Both point at `index.html`, like the other five pages |
| 1.5 | Onboarding, "Open your organiser invite" | An invite link | `href="#"` with a click handler that jumps straight to the setup screen. No invite exists, so anyone can become an organiser by clicking a sentence | Reads "Set up an event" and links straight to the setup screen. No pretend invite |
| 1.6 | Menu, "Voting" | The voting rules | Scrolls to the three role tabs, which is the "One pitching event, three jobs" section | Renamed "Who does what", which is what that section is |
| 1.7 | Footer, "What we store" and "How to enter" | Two different pages | Both jump to the same FAQ block | Jump to their own answers, `#faq-deck` and `#faq-enter` |
| 1.8 | Footer, `hello@podium.app` | A mailbox | The domain does not exist, so the mail bounces | Replaced with "Report a problem", pointing at the repository issues |
| 1.9 | Wall, "See the poster" | Always available | Appears only if a poster image was uploaded on the setup screen, which most organisers will skip | Stays visible, disabled, reading "No poster added" with a tooltip saying the organiser has not uploaded one |
| 1.10 | Deck viewer, "Slide 3 of 15" | The deck has 15 slides | 15 is the event's slide **limit**, not the file's page count | New uploads store no count, and the viewer says "Page 3" with no total. A count is only shown when one is known |

---

## Part 2 — Things that are open doors — **front end closed, four wait on the server**

Not "bugs" exactly. The interface asks politely and the data does not enforce.

| # | Issue | What happens today | What it needs |
|---|---|---|---|
| 2.1 | **Anyone can vote without signing in** | The wall lets a visitor with no profile vote. `isMine()` compares the deck owner to `profile?.name`, so with no profile nothing is "mine" and even your own deck is votable | **Done.** Reading the wall needs nothing; voting needs a profile. The button reads "Sign in to vote" and goes to sign-in, and `castVote` re-checks every condition before it writes |
| 2.2 | **Anyone can upload without signing in** | The upload sheet works with no account. "One deck per team" is not enforced at all | **Done in the front end.** Upload needs a profile, refuses once uploads close, and refuses a second deck from the same account. The database has to enforce the same, since a browser check is a courtesy |
| 2.3 | **Identity is a string** | Ownership and self-vote checks compare **display names**. Two teams called "Team Kestrel" are the same person as far as the code knows | **Done.** A profile now carries a generated id, decks store `ownerId`, and ownership and self-vote checks compare ids. Names are used only for decks uploaded before ids existed |
| 2.4 | **Roles are a browser value** | `podium.profile.role` decides what the header shows. The console can make anyone an organiser | Still open. Needs the server |
| 2.5 | **Team names go into the page as HTML** | A team called `<img src=x onerror=...>` runs script for every viewer, organiser included | **Done.** Every value that reaches the page goes through `esc()` first. Verified with a team named `<img src=x onerror=...>`: it renders as text, no element is created, no script runs |
| 2.6 | **Votes are a list in the browser** | Clearing site data returns every vote. Counts are per device | Still open, but votes are now stored per voter id rather than in one shared list, so the shape matches the table it is going into |

---

## Part 3 — Unfinished screens and states

| # | Missing | Why it is felt |
|---|---|---|
| 3.1 | **Event page** | There is no page for an event that is not yet open: no poster, dates, rules, sign-up link or countdown. A team with a code has nowhere to land before uploads open |
| 3.2 | **A deck has no address** | Decks open in a dialog, so a team cannot send anyone a link to their own deck |
| 3.3 | **Slides are not rendered** | The viewer shows an empty frame per slide. This is the biggest hole in the product: the wall exists to be read |
| 3.4 | **Teams and Voters lists** | You asked for both on the dashboard. Not built |
| 3.5 | **Ties** | The results page states a tie honestly but the organiser cannot resolve it anywhere |
| 3.6 | **No profile screen** | A name or college typed once cannot be changed, and a team cannot delete its own deck, which the FAQ promises |
| 3.7 | **Nothing for a second event** | Every screen assumes one event in storage. No list, no switching, no history |
| 3.8 | **No "not found" states** | Opening the wall or the dashboard with no event shows an empty shell rather than "no event yet, set one up" |
| 3.9 | **No Terms or Privacy page** | Student work and names are being stored. These need writing before a real fest |
| 3.10 | **Share card** | Planned, not built |

---

## Part 4 — Smaller cracks worth closing

- **Sample content still on the landing page.** The podium shows Team Kestrel,
  Team Monsoon and Team Nightshift; the counter shows 1 / 3 / 6. Both are
  illustration, and neither is labelled as such.
- **"Trusted by" claims three colleges** that have not used it yet.
- **Dead CSS.** `.frame-bar`, `.card-visual`, `.deck-thumb`, `.fab-demo`,
  `.grid-2` are left over from mock cards that are gone.
- **Two upload paths disagree.** The wall's sheet caps at 25 MB; the setup
  screen's banner and poster cap at 5 MB with a different message.
- **Poster and banner images are base64 in `localStorage`.** A 4 MB poster can
  fill the 5 MB quota and quietly break every save after it.
- **The event chip is hidden under 720px**, so on a phone no screen says which
  event you are in.
- **No focus trap in the dialogs.** Tab can walk out of the upload sheet into
  the page behind it.
- **`aria-live` is missing** on the vote button, so a screen reader is not told
  the vote was counted.
- **No page title changes per event**, so several open tabs all read "Podium".

---

## Part 5 — Decisions needed from you

Each one changes what gets built, so none of them are mine to make.

1. **Hosting.** Supabase (Postgres, sign-in, storage, row-level security in one
   place) or Firebase (faster to start, weaker at constraints)? My recommendation
   is Supabase, because the vote rules are unique constraints and policies.
2. **Who may vote** — anyone with an account, or only people the organiser
   admitted? Open voting is how a fest feels; admitted-only is how it stays
   clean.
3. **Ties.** Shared places with no automatic tiebreak and an organiser decision
   on the record, as proposed — or an automatic rule such as earliest upload?
4. **Judges.** Does v2 need a separate jury score alongside the room's vote?
   Most fests award both.
5. **One event or many.** Should an organiser be able to run two events at once,
   and should the home page list open events publicly?
6. **Retention.** Confirm 30 days public, 90 days before the file is deleted,
   winners kept for good.
7. **Money.** Confirmed off the platform for good, or is a fee-on-Podium ever on
   the table? Everything in the copy currently promises never.
8. **Domain and name.** The repo is `podium`, the product is
   Podium, and the footer mail is `hello@podium.app`. Three names, one product.

---

## Part 6 — The order I would work in

**Now, front end only, no decisions blocked (half a day)**

1. Escape every interpolated value — the script injection hole (2.5).
2. Fix the six lying links and labels (1.1, 1.2, 1.4, 1.5, 1.6, 1.8).
3. Gate voting and uploading behind a signed-in profile, even the browser-only
   one, so the rule is visible before it is real (2.1, 2.2).
4. Label the illustration: mark the sample podium and the counter as examples.
5. Delete the dead CSS and unify the upload limits.

**Then, still front end (a day)**

6. Event page with poster, dates, rules and countdown (3.1).
7. Deck addresses, so `event.html#deck-123` opens that deck (3.2).
8. Empty and not-found states everywhere (3.8).
9. Tie resolution on the dashboard (3.5).

**Then the server, in this order (the real work)**

10. Supabase project, schema, row-level security.
11. Sign-in and profiles; role from membership.
12. Decks and votes in the database, with the unique constraint.
13. `pdf.js` on upload for real page counts and rendered slides (3.3).
14. Storage for decks, posters and banners; signed URLs.
15. Teams and Voters lists with CSV (3.4).
16. Terms and Privacy (3.9), then the share card (3.10).

Steps 1 to 5 are worth doing today whatever you decide, because they make the
site stop claiming things that are not true.
