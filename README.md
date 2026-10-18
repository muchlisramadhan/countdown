# countdown

A static date-countdown calendar. It works offline in a browser and stores events on that device by default.

## Install on Android

The site is an installable Progressive Web App (PWA). Publish the repository on GitHub Pages or another HTTPS host, then open its URL in Chrome on Android. Open Chrome's menu and choose **Install app** or **Add to Home screen**. The app shell is cached for offline use after the first visit; events remain stored on the device.

For local testing, serve the repository over `http://localhost` (for example, `python -m http.server 3000`) and open it in Chrome. Service workers require HTTPS on published sites; `localhost` is allowed for local development. Sync across devices requires an internet connection.

## Sync across devices

The app can sync events privately between devices using Supabase and email sign-in. GitHub Pages does not provide a shared database, so configure a Supabase project once:

1. Create a project at [supabase.com](https://supabase.com/).
2. In the Supabase SQL editor, run:

   ```sql
   create table public.countdown_data (
     user_id uuid primary key references auth.users(id) on delete cascade,
     data jsonb not null,
     updated_at timestamptz not null default now()
   );

   alter table public.countdown_data enable row level security;

   create policy "Users can read their own countdown data"
     on public.countdown_data for select
     using (auth.uid() = user_id);

   create policy "Users can insert their own countdown data"
     on public.countdown_data for insert
     with check (auth.uid() = user_id);

   create policy "Users can update their own countdown data"
     on public.countdown_data for update
     using (auth.uid() = user_id)
     with check (auth.uid() = user_id);
   ```

3. In Supabase project settings, copy the Project URL and publishable key (or legacy `anon` key). In `index.html`, set `SUPABASE_URL` and `SUPABASE_ANON_KEY` near the start of the script to those values. These are public client settings; do not put a service-role key in the page.
4. Set the Supabase Auth URL configuration to include your GitHub Pages site URL in the allowed redirect URLs. For local testing, also allow `http://localhost:3000/**`. Set the site URL to your GitHub Pages address.
5. Publish the updated `index.html`. On each device, open the same site and sign in with the same email address using the emailed link.

Sign-in links expire and can only be used once. Request and open the latest link in the same browser and device. For cross-device use, open the published GitHub Pages site rather than `localhost`; localhost is only reachable on the device running the local server.

When an account has no synced data yet, the current device's events are uploaded. When data already exists for the account, that synced copy is loaded on the device. Changes are automatically saved locally and synced after a short delay.