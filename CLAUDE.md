# Members & Guests — Project Context

Read this first. It captures everything needed to pick up where the last session left off.

---

## Project Goal

A private, invite-only network for country club members to host one another at clubs across the country. The marketing promise is: apply → get verified → browse verified members → reach out and arrange a round at their club.

**Live at:** [membersandguests.com](https://membersandguests.com) (GitHub Pages, custom domain)
**Repo:** [github.com/sqoax/membersandguests](https://github.com/sqoax/membersandguests) — **public**
**Owner:** single solo founder, **complete novice at code** — explain things plainly, avoid jargon unless you define it. They will push back if you dump too much.

---

## Stack / Files

Static HTML only. **No build system, no server, no package.json.** Four standalone HTML files, each with its own inline `<style>` and `<script>`. Supabase is called directly from the browser with the JS SDK loaded from a CDN.

```
├── CLAUDE.md              ← this file
├── CNAME                  → membersandguests.com
├── index.html             Landing page
├── apply/index.html       Application form
├── admin/index.html       Admin dashboard (login + approve/delete/invite)
└── members/index.html     Member login + directory
```

**Backend:** Supabase project at `upohsbvsycwjsjbtwsll.supabase.co`
- Table: `applications` (all applicant + member data lives here, distinguished by `status` column: `pending` vs `verified`)
- Supabase Auth is used for member login only
- Discord webhook receives new-application pings (URL base64-encoded in `apply/index.html`)

**Design system** (duplicated across all four files):
- Colors: navy `#1a2744`, cream `#f8f5f0`, forest `#2d4a2d`, gold `#c9a96e`
- Fonts: Cormorant Garamond (serif, display) + Inter (sans, body)
- Tone: understated, editorial, traditional-country-club. No emojis, no bright colors, no heavy shadows.
- Common patterns: IntersectionObserver reveal animations, noise-texture SVG overlays, gold divider ornaments (line–diamond–line)

---

## What's Been Completed

### End-to-end flow works
Stranger applies → admin reviews → admin approves → admin clicks Invite → Supabase emails member → member sets password → member sees directory. All functioning.

### Recent commits (this session)
1. **[fa2cde3](https://github.com/sqoax/membersandguests/commit/fa2cde3)** — Added "Sign in" link to landing-page nav (next to "Apply", reveals on scroll, points to `/members/`)
2. **[477af6a](https://github.com/sqoax/membersandguests/commit/477af6a)** — Quick-win cleanup batch:
   - `apply/`: removed 3-second pageload check and 30-second localStorage throttle that silently dropped real submissions. Honeypot kept for bots.
   - `admin/`: added `window.confirm()` before delete.
   - `admin/`: humanized `membership_type`, `willing_to_host`, `how_heard` via label maps.
   - `admin/`: displays `referral_code` on cards (was collected but unused).
   - `members/`: location now shows only `club_location` (was awkwardly concatenating with applicant's city).
   - `index.html`: deleted 111 lines of dead email-capture CSS.

### Git workflow setup
- Local folder at `/Users/hiattgafnea/Downloads/m&g/` is now a proper clone of the repo (was loose files before)
- Old loose-files backup still exists at `/Users/hiattgafnea/Downloads/m&g-backup/` — safe to delete once user confirms
- `gh` CLI installed and authed as `sqoax` (scopes: `repo`, `read:org`, `gist`)

---

## 🚨 Bugs & Blockers (critical first)

### Security — CRITICAL but deprioritized
The user knows, has acknowledged, and chose to defer until closer to launch because the project has zero public awareness.

1. **Supabase service role key is publicly committed** in `admin/index.html` (look for `sb_secret_`). This is a god-mode key. Rotate before any real launch.
2. **Admin password is client-side plaintext** (`mg-admin-2026` in same file). Trivially bypassable by setting `sessionStorage.mg_admin_unlocked = 'true'`.
3. **Anon role likely has UPDATE/DELETE on `applications`** (the admin page uses the anon client for these). Any visitor could approve themselves or wipe the table.
4. **Discord webhook URL exposed** in base64 in `apply/index.html`.
5. **RLS on `applications` table unknown** — could be leaking PII of pending applicants to authenticated members.

Full security plan lives in session history as "Phase 0–4." Phase 0 (rotate + lock RLS) is the emergency minimum.

### Known functional bugs not yet fixed
- **Invite button state doesn't persist.** Click Invite → refresh → button says "Invite" again. Requires adding `invited_at` column to `applications` table (needs user to run SQL in Supabase dashboard).
- **Login error is ambiguous.** "Wrong password" vs "account not yet activated" look identical to member. Non-trivial to distinguish via Supabase Auth errors.
- **Footer Instagram link is `href="#"`** on all four pages. User hasn't decided whether they want an IG account or want the icon hidden.

### Missing functionality the marketing page implies but doesn't deliver
- No member-to-member messaging or intro request
- No hosting calendar or booking flow
- No club verification (applicants are trusted at their word)
- No password reset for members
- No applicant confirmation email
- No profile detail pages — directory is read-only cards

---

## Exact Next Priorities

User will pick one of these when they return. Don't assume — ask.

1. **Password reset for members.** Supabase has this built in. Needed before the "Sign in" nav link gets real traffic.
2. **Applicant confirmation email.** After submitting the application, send the applicant a "we got your application" email. Closes the "submit into a void" UX gap.
3. **Phase 0 security cleanup.** Rotate service role key, rotate Discord webhook, lock down RLS. User has to do the dashboard parts themselves; I write the code changes.
4. **Member-to-member intro flow.** The actual product. Simplest MVP: a "Request intro" button on each member card that emails the host via an Edge Function.
5. **Footer Instagram** — waiting on user decision (real handle vs hide).
6. **Invite persistence** — add `invited_at` column, update logic.

---

## Design Preferences (observed)

- **Understated, editorial tone.** Avoid marketing-speak, exclamation points, or generic SaaS voice.
- **Serif + sans combo** is sacred — don't swap fonts without asking.
- **No emojis in copy.** (Only exception so far: a single 📋 emoji in the Discord webhook embed.)
- **Buttons are thin-bordered** with uppercase letter-spaced labels, navy or gold. Not rounded, not filled-pill.
- **Gold (`#c9a96e`) is the accent** — use sparingly, as highlights and dividers.
- **Keep inline styles** for now. User hasn't asked for a build system and doesn't need one yet.
- **Copy is short and confident**, not apologetic. Example good phrasing: "Membership is reviewed, not automatic." Example bad phrasing: "Please apply and we'll try our best to get back to you!"

---

## Key Commands / Workflow

### Git (user doesn't know these — just do them)
```bash
cd "/Users/hiattgafnea/Downloads/m&g"
git add .
git commit -m "..."
git push
```
Always push to `main`. No branches, no PRs — user works solo and the site auto-deploys from `main` via GitHub Pages.

### GitHub CLI
Authed as `sqoax`. Use for repo inspection, issue creation, gist sharing, etc.

### Verifying deployed changes
After `git push`, GitHub Pages takes 30–90 seconds to redeploy. `membersandguests.com` reflects `main` branch directly.

---

## Things to Know Before Continuing

- **User wants direct, honest answers.** They specifically asked earlier: "Be direct and thorough. Don't sugarcoat issues." Respect that. No false reassurance.
- **User is a novice.** Explain what you're doing, especially around git / Supabase / deployment concepts. They've never cloned a repo before this week.
- **Repo is public.** Anything committed is world-readable, including the known secrets. User is aware and has chosen to defer.
- **Site is live.** `membersandguests.com` is not a staging environment. Every `git push origin main` goes to production immediately. Treat accordingly.
- **No tests, no CI.** Test by loading the live site or opening the HTML in a browser.
- **When context gets full again:** write an updated `CLAUDE.md` before `/clear`. The user asked explicitly for this pattern.
- **Supabase schema is not checked in.** If you need to know the table structure, check the `insert` call in `apply/index.html` around line 1120, or query Supabase directly.

### Shortcut phrases the user uses
- "push that to github" → do all of git add/commit/push
- "fix X" → make the change, push it, tell them what shipped
- "what do you want to tackle" → user is handing you the decision; propose a concrete batch with rationale, get their go-ahead, execute

---

_Last updated: 2026-04-22, end of session 2._
