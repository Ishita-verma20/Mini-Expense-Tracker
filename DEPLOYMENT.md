Vercel (frontend + Next API) + Render (Postgres) deployment steps

Overview
- We'll deploy the Next.js app (frontend + API routes) to Vercel.
- We'll provision a managed Postgres database on Render and point `DATABASE_URL` at it.
- For local dev you can continue using SQLite (`file:./dev.db`). The project now supports switching via `DATABASE_URL`.

High-level steps
1) Provision a Postgres database on Render
   - Sign into https://render.com
   - Create -> Database -> PostgreSQL
   - Choose plan (free/managed) and create the DB
   - After creation, copy the DATABASE_URL connection string (format: `postgres://user:pass@host:port/dbname`)

2) Configure Vercel
   - In your Vercel dashboard, import the GitHub repo and set project to the `expense-tracker` repo
   - In Project Settings -> Environment Variables, add `DATABASE_URL` and paste the Render Postgres connection string
   - Ensure `NODE_ENV` is set to `production` (Vercel sets this automatically on production deploys).

3) Update Prisma provider before pushing production schema
   - Edit `prisma/schema.prisma` and ensure `provider = "postgresql"`.
   - Commit that change.
   - Set `DATABASE_URL` in your local `.env` to your Render Postgres connection string when testing locally.

4) Push and deploy on Vercel
   - Push your branch to GitHub; Vercel will build and deploy the app automatically.
   - The build uses `prisma generate && prisma db push && next build` (see `package.json`).
   - Because `DATABASE_URL` points to Postgres and the provider is `postgresql`, Prisma will create the schema in your Render DB.

5) Migrating data from local SQLite (optional)
   - If you have data in `dev.db` you can migrate it manually.
   - Quick approach: export rows from SQLite and import them into Postgres via a small Node script or using `pg`/`psql`.
   - A simple Node script approach:
     - Run locally with `DATABASE_URL=file:./dev.db` to read existing rows, then run again with `DATABASE_URL` pointed to Postgres and insert rows.
     - Beware of primary key collisions; you may need to preserve IDs or let Prisma generate new ones and re-map relations.

6) Verify the deployed app
   - Open the Vercel URL in an incognito window and test flows (list/add expenses, budgets, analytics).

Notes & troubleshooting
- `better-sqlite3` is only required for local SQLite usage. Production with Postgres does not need it.
- If the build fails because Prisma client was generated for a different provider, run `prisma generate` locally after switching `provider` and commit the generated client if you prefer, or let the build generate it on Vercel.
- If you want a separate backend API (Express) hosted on Render instead of Next.js API routes, I can scaffold a lightweight `server/` and deployment config. That will require adding `server` npm scripts and a small Dockerfile or Render service settings.

Commands you will run locally (example)

# Switch schema to Postgres (edit file and commit)
# Generate client and push schema to Postgres
npx prisma generate
npx prisma db push --preview-feature

# Optional: revert to SQLite locally
# set DATABASE_URL=file:./dev.db (or unset) and run:
npx prisma generate

If you want me to scaffold an Express backend on `server/` and prepare Render service files, say so and I'll create the files and commit them.