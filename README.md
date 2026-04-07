# Fruit Madness 2026 🍉🏆

A single-page bracket voting app for 32 fruits. No build step — just `index.html` + Supabase.

## Supabase Setup

1. Create a [Supabase](https://supabase.com) project.
2. Go to **SQL Editor** and run:

```sql
create table submissions (
  id uuid default gen_random_uuid() primary key,
  ip text not null unique,
  bracket jsonb not null,
  champion text not null,
  created_at timestamptz default now()
);
```

3. Open `index.html` and fill in your credentials at the top:

```js
const SUPABASE_URL = 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = 'your-anon-key';
```

4. In Supabase **Authentication → Policies**, add a policy on `submissions`:
   - **INSERT**: allow for `anon` role (so visitors can submit)
   - **SELECT**: allow for `anon` role (so results page can read all submissions)

## Deploy to GitHub Pages

1. Create a new GitHub repo (e.g. `fruit-madness`).
2. Push `index.html` to the `main` branch.
3. Go to **Settings → Pages** → set source to `main` branch, root `/`.
4. Your app will be live at `https://<username>.github.io/fruit-madness/`.

## How It Works

- **Bracket page**: Users click fruits to advance them through 5 rounds (Round of 32 → Championship). All 31 picks required.
- **IP dedup**: On submit, the user's IP is fetched via `api.ipify.org` and stored. The `unique` constraint on `ip` prevents double submissions.
- **Results page**: Aggregates all submissions to compute:
  - Power Rankings (Elo-style scoring)
  - Most Likely Final Four
  - Biggest Upsets (by seed gap × frequency)
  - Most Overrated / Underrated
  - Champion distribution chart

## Bracket Format (stored in `bracket` JSONB column)

```json
{
  "rounds": [
    [[1,32],[16,17],...],
    ...
  ],
  "champion": 1
}
```

Each round is an array of `[winner_seed, loser_seed]` pairs. Seeds map to fruits client-side.
