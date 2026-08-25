# Recipes — setup

A single static `index.html`. No build step, no dependencies, no JS framework.
It talks to Supabase directly over its REST API using plain `fetch`.

---

## 1. Create the Supabase project

1. Go to supabase.com, sign in, **New project**. The free tier is fine.
2. Once it finishes provisioning, open **SQL Editor** and run this:

```sql
create table if not exists recipes (
  id         uuid primary key default gen_random_uuid(),
  title      text not null,
  steps      jsonb not null default '[]'::jsonb,
  created_at timestamptz not null default now()
);

alter table recipes enable row level security;

create policy "anon read"   on recipes for select to anon using (true);
create policy "anon insert" on recipes for insert to anon with check (true);
create policy "anon delete" on recipes for delete to anon using (true);
```

A `steps` row looks like:

```json
[
  { "ing": "200g flour\n1 tsp salt", "ins": "Mix the dry ingredients." },
  { "ing": "2 eggs", "ins": "Beat the eggs and fold them in." }
]
```

One object per screen. `ing` is free text, one ingredient per line.

## 2. Get your keys

**Project Settings → API**. You need two values:

- **Project URL** — `https://xxxxxxxx.supabase.co`
- **anon / publishable key** — the long `eyJ...` string (the *anon* one, **not** `service_role`)

## 3. Connect the app

Either:

- **Open `index.html` and paste them into the setup screen.** They're saved in that browser's localStorage. You'd repeat this once per device.
- **Or hardcode them** — open `index.html` and fill in the two constants near the top of the `<script>`:

```js
var BUILTIN_URL = "https://xxxxxxxx.supabase.co";
var BUILTIN_KEY = "eyJhbGciOi...";
```

Hardcoding is the better option once you host it, so it just works on any device.

## 4. Put it on your phone

The file needs to be served over `https://` to feel like an app. Any of these, all free:

- **Netlify Drop** — netlify.com/drop, drag the folder in, done in about ten seconds.
- **GitHub Pages** — push the folder to a repo, Settings → Pages → deploy from `main`.
- **Vercel** — `vercel deploy` in the folder.

Then on your phone open the URL in Safari → Share → **Add to Home Screen**. It launches
fullscreen with no browser chrome, so it reads like a native app.

---

## A note on security

The anon key is visible to anyone who opens the page, and the policies above let anyone
holding it read, add, and delete recipes. That's the normal trade-off for a keyless personal
app and it's fine for recipes — but don't put anything private in this table.

If you later want it locked to just you, the change is to turn on Supabase Auth (email magic
link is about 20 lines) and swap the policies to `to authenticated using (auth.uid() = user_id)`.
Say the word and I'll wire it up.

---

## How the app is built

| Screen | What it does |
|---|---|
| **Setup** | Only appears if the URL and key aren't set. Test-fetches before saving. |
| **Menu** | Two tabs: Browse recipes, Add new recipe. |
| **Browse** | List of recipes with step counts. **Manage** reveals per-row Delete. |
| **Reader** | One step per screen. Ingredients for that step on top, instructions below. Tap the **left 35%** to go back, anywhere right of that to go forward. A progress bar sits under the header. Arrow keys work on desktop. |
| **Add** | Recipe name, then a stack of step cards — ingredients box and instructions box each, Quizlet style. Add step / Remove per card. |

Colours are three shades of terracotta plus white text, defined as CSS variables at the top
of the file (`--bg`, `--bg-deep`, `--bg-lift`). Change those three to restyle the whole app.
