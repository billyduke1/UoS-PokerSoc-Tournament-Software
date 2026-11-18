# UOS Poker

The website and tournament management software for **UOS Poker — the University of Sheffield Poker Society**: the society's 
information, two public leaderboards, and a real-time **play-money** online poker room with a 
No-Limit Hold'em engine, tiered AI bots, and a rated Elo ladder.

> **Play-money only.** There is no real-money gambling anywhere on this site.

---

## Stack

- **Monorepo** (pnpm workspaces)
  - `packages/engine` — pure, deterministic TypeScript poker engine (rules, evaluator, side
    pots, bots). Zero I/O.
  - `packages/shared` — zod schemas, constants, socket contracts, Elo math, profanity filter.
  - `apps/server` — Fastify + Socket.IO + Better Auth + Prisma/Postgres + Resend/React Email.
    Serves the built SPA in production.
  - `apps/web` — React + Vite + Tailwind v4 + Framer Motion SPA.
- One deploy target: a single **Railway** service + Postgres. External vendors: **Resend**
  (email) and **Google OAuth** (optional sign-in).

See `CLAUDE.md` for architecture conventions and `NOTES.md` for deliberate simplifications.

---

## Local development

**Prerequisites:** Node 22+, `pnpm` (`npm i -g pnpm`). No local Postgres needed — dev uses an
embedded Postgres 17 (downloaded on first run).

```bash
pnpm install

# 1. Start the dev database (real Postgres 17, project-local data dir, port 5433).
#    Leave this running in its own terminal.
pnpm --filter @uos-poker/server db:dev

# 2. In another terminal, create the local .env and apply migrations.
cp .env.example .env            # then fill in secrets (see below)
pnpm --filter @uos-poker/server db:migrate

# 3. Run server + web together.
pnpm dev
#   web  → http://localhost:5173
#   api  → http://localhost:3001
```

In development you can play immediately via the **dev door** on the lobby (pick a nickname;
no account needed). Email verification links are captured in an in-memory mailbox at
`GET /api/dev/mailbox?to=<email>` instead of being sent.

In **production** visitors can also "Play as guest" from the lobby — an ephemeral identity
(no account) sandboxed to **practice tables vs bots + spectating**. Rated play, a saved
bankroll, and the leaderboards still require a verified account. This keeps the live demo
one click away for portfolio/society visitors without exposing rated tables to throwaway
identities.

### Useful commands

```bash
pnpm typecheck         # tsc across the workspace
pnpm lint              # eslint
pnpm test              # vitest (engine + shared + server)
pnpm exec playwright test   # E2E (boots server+web automatically)
pnpm --filter @uos-poker/server db:studio   # Prisma Studio
```

---

## Environment variables

Copy `.env.example` to `.env` for local dev; set the same keys in Railway for production.

| Variable | Required | Notes |
|---|---|---|
| `DATABASE_URL` | yes | Local: the embedded PG URL in `.env.example`. Railway: the Postgres plugin's URL. |
| `BETTER_AUTH_SECRET` | yes | `node -e "console.log(crypto.randomBytes(32).toString('hex'))"` |
| `BETTER_AUTH_URL` | yes (prod) | The server's public URL, e.g. `https://uospoker.up.railway.app`. |
| `SITE_URL` | yes (prod) | Public site URL (same as above for the single-service deploy). |
| `PORT` | no | Defaults to 3001; Railway injects its own. |
| `RESEND_API_KEY` | prod | Without it, email is captured to the dev mailbox (dev) or logged (prod). |
| `EMAIL_FROM` | prod | e.g. `UOS Poker <noreply@yourdomain>`; the domain drives the DNS print. |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | optional | Enables "Continue with Google". |
| `ADMIN_EMAIL` | recommended | The account that signs up with this email is auto-promoted to admin. |


---


## System description

This repo has two purposes - to provide a promotional and informational website for the society, and to house the necessary software for running tournament sessions and storing persistent weekly performance data

**Promotional website:** 
This website supplements the society's other social media pages and helps to capture a wider audience, containing both students and potential sponsors. For students, it acts as both a source of information about the society and as a learning tool for students to improve their poker game. For sponsors, it acts as a portfolio and introduction to what the society does, how it runs, and what it can provide to comapanies.

**Tournament management system:**
The key part of this repo is the tournament backend - a custom-built piece of software to assist the society in their running of tournament session. It features in-built blind timers, adjustable blind levels, an autonomous sign-in flow, and an account system that allows for leaderboard data to be stored and presented as public leaderboards for the society.
