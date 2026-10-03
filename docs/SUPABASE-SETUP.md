# Setting up the backend

Two accounts, both free, about twenty minutes. Everything that could be written
without your login is already in this repository; this file is the part only you
can do.

**A note on keys before anything else.** Supabase gives two. The **anon** key is
meant to be public — it sits in the browser and can only do what the row-level
policies allow. The **service_role** key bypasses every policy. Never paste it
into a file here, a workflow, or the browser. If it ever leaks, rotate it in the
dashboard immediately.

---

> **Done already:** the project exists (`podium`, Mumbai, free plan) and all
> three SQL files have been applied to it, with a fourth hardening migration on
> top. `config.js` already points at it. What is left for you is step 3, Google
> sign-in, and step 4, the R2 bucket — R2 needs enabling in the dashboard first,
> which is a billing verification only you can do.

## 1. Supabase project

1. Sign in at https://supabase.com with GitHub.
2. **New project.** Name it `podium`. Pick the region closest to your campus —
   `ap-south-1` (Mumbai) for Chennai. Set a database password and keep it in a
   password manager; you will rarely need it.
3. Wait for the project to finish building, about two minutes.

## 2. Run the three SQL files, in order

Open **SQL Editor** in the Supabase dashboard, then for each file: paste the
whole thing, run it, check it says success.

| Order | File | What it makes |
|---|---|---|
| 1 | [`supabase/01-schema.sql`](../supabase/01-schema.sql) | Tables, types, constraints, indexes, the view that counts votes, and the trigger that refuses to change a vote |
| 2 | [`supabase/02-policies.sql`](../supabase/02-policies.sql) | Row-level security: who may read and write what |
| 3 | [`supabase/03-keepalive.sql`](../supabase/03-keepalive.sql) | The single `heartbeat` row the scheduled job reads |

Every statement is guarded, so running a file twice is harmless.

## 3. Turn on sign-in

1. **Authentication, Providers, Google.** Enable it.
2. You need a Google OAuth client: https://console.cloud.google.com, APIs and
   services, Credentials, Create OAuth client ID, Web application.
3. Authorised redirect URI — copy the callback shown on the Supabase provider
   page, which looks like
   `https://<project-ref>.supabase.co/auth/v1/callback`.
4. Paste the client id and secret into Supabase and save.
5. **Authentication, URL Configuration.** Site URL is
   `https://rmagdaleena2508-01.github.io/podium/`. Add
   `http://localhost:8793` as an additional redirect URL so local work keeps
   working.

## 4. Cloudflare R2 bucket

1. Sign in at https://dash.cloudflare.com, then **R2**. The free tier needs a
   card on file for verification but does not charge inside 10 GB.
2. **Create bucket**, name it `podium-files`, location closest to you.
3. **Settings, Public access.** Enable the public r2.dev URL for now, or attach a
   custom domain later. Only rendered page images are read from it.
4. **Lifecycle rules.** Add one: delete objects under the prefix `originals/`
   after **90 days**. That is the retention promise in the FAQ, enforced by the
   bucket rather than by anyone remembering.
5. **Manage R2 API tokens.** Create a token with **Object Read and Write**,
   scoped to this bucket. Keep the id and secret in your password manager — they
   go into the upload worker later, never into this repository.

The bucket layout the code expects:

```
originals/<event-id>/<deck-id>.pdf      private, deleted at 90 days
pages/<deck-id>/<n>.webp                what the wall shows
posters/<event-id>.jpg
banners/<event-id>.jpg
```

## 5. Point the site at both

```bash
cp config.example.js config.js
```

Fill in the project URL, the **anon** key (Project Settings, API) and the R2
public base URL. `config.js` is in `.gitignore`, so your keys stay on your
machine. For the live site on GitHub Pages, the same file is committed with the
anon key only — that key is public by design.

## 6. Keep the project awake

The workflow is already in the repository:
[`.github/workflows/keep-supabase-awake.yml`](../.github/workflows/keep-supabase-awake.yml).
It reads one row every third day, which is enough to stop the seven-day pause.

Add two repository secrets under **Settings, Secrets and variables, Actions**:

| Secret | Value |
|---|---|
| `SUPABASE_URL` | `https://<project-ref>.supabase.co` |
| `SUPABASE_ANON_KEY` | the anon public key |

Then open the **Actions** tab and run **Keep Supabase awake** once by hand. A
green tick means the project answers and the schedule will hold it open.

If it ever goes red with a 404 or a timeout, the project is already paused:
restore it from the Supabase dashboard — the data is intact on disk — and the
job goes green again.

---

## What is still to write, in order

1. `supabase.js` — one small module that creates the client from `config.js`,
   signs people in with Google, and creates a profile row on first sign-in.
2. Read paths: events by code, decks for an event, counts after close.
3. Write paths: create event, join event, upload deck, cast vote — each of them
   a single insert that the policies either allow or refuse.
4. A worker that puts uploads into R2 with a signed URL, renders the pages with
   `pdf.js`, and writes the `deck_pages` rows.
5. Swap the browser-storage reads in `app.js` for these calls, one screen at a
   time, wall first.

Until step 5 lands, the site keeps working exactly as it does now, from browser
storage. Nothing breaks while the backend is being wired up.
