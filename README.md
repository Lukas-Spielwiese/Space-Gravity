# Space Gravity – Vercel Build (Supabase REST)

**Domain:** https://space-gravity-two.vercel.app  
**Änderungen:**
- Share-Links/Copy-URL zeigen auf `https://space-gravity-two.vercel.app`.
- Inline CSS/JS (keine externen Pfade), dadurch problemlos als `index.html` auf Vercel nutzbar.
- Supabase REST bleibt aktiv (gleiche `supabaseUrl` + `anon`-Key).

## Deploy (bekannter Weg)
1. Datei `index.html` ins **public GitHub Repo** (Root).
2. Vercel → **Add New Project** → Repo importieren.  
   - Framework: **Other** · Build: leer · Output: `/` (Root).
3. Nach Deploy Domain in Project **Settings → Domains** setzen/prüfen.

## Supabase (Tabelle + RLS)
Falls noch nicht vorhanden, im SQL-Editor ausführen:
```sql
create table if not exists public.leaderboard (
  id uuid primary key default gen_random_uuid(),
  created_at timestamp with time zone default now(),
  name text not null check (char_length(name) between 1 and 12),
  score integer not null check (score >= 0),
  wave integer not null default 1,
  duration integer not null default 60,
  date text not null,
  mode text not null check (mode in ('classic','passplay','daily'))
);

alter table public.leaderboard enable row level security;

create policy public_can_select on public.leaderboard
  for select using (true);

create policy public_can_insert on public.leaderboard
  for insert with check (true);
```
> Für Produktion kannst du Insert/Select über Edge Functions härten.

## Test der DB-Anbindung
- Starte ein Spiel, beende es → Prompt für Namen → Score wird via `POST /rest/v1/leaderboard` geschrieben.
- Öffne im Spiel „Bestenliste“ → lädt **Top 10** für den aktuellen Modus via `GET /rest/v1/leaderboard?mode=eq.<mode>&order=score.desc&limit=10`.

## Hinweise
- Mobile Touch: `touch-action: none` verhindert Scrollen während der Steuerung.
- Wenn du die Supabase-Projekt-URL/Keys änderst, passe sie oben im Script an.
