# guild-app-template

Next.js 15 template for Movement Guild apps. Pre-wired with Discord OAuth, Movement wallet connection, and Supabase.

## What's Included

- **Discord OAuth** via NextAuth — guild membership verified on sign-in, roles cached in JWT
- **Movement Wallet** — @aptos-labs/wallet-adapter-react pre-configured
- **Supabase** — per-app Postgres schema, service role server actions
- **Wallet + Discord linking** — opt-in via `REQUIRE_WALLET_LINK=true`
- **Shared config** — ESLint, Prettier, TypeScript, Tailwind via `@movement-guilds/shared-config`

---

## Getting Started

### Prerequisites

Complete these **before** running the bootstrap script:

#### 1. Install required CLIs

```bash
# GitHub CLI
brew install gh
gh auth login

# Supabase CLI
brew install supabase/tap/supabase
supabase login

# Vercel CLI
npm i -g vercel
vercel login
```

#### 2. Create an empty GitHub repo

1. Go to https://github.com/organizations/movement-guilds/repositories/new
2. Name it your app name (e.g., `guild-leaderboard`)
3. **Leave it completely empty** — no README, no .gitignore, no license
4. Click "Create repository"

---

### Bootstrap Your App

From your terminal:

```bash
git clone https://github.com/movement-guilds/.github
cd .github
./scripts/new-app.sh <your-app-name> <schema-prefix>
```

**Example:**
```bash
./scripts/new-app.sh guild-leaderboard gl
```

#### What is `schema-prefix`?

Supabase uses PostgreSQL schemas to organize tables. Each guild app gets its own schema to avoid table name conflicts. The prefix is a short identifier (2-8 lowercase letters) for your app's database tables.

| App | Schema Prefix | Tables created as |
|-----|---------------|-------------------|
| guild-leaderboard | `gl` | `gl.users`, `gl.scores`, etc. |
| guild-chess | `gc` | `gc.games`, `gc.moves`, etc. |

**Reserved names you can't use:** `public`, `auth`, `storage`, `extensions`

---

### Post-Bootstrap Setup

After the script completes, follow these steps in order:

#### Step 1: Create Discord OAuth App

1. Go to https://discord.com/developers/applications
2. Click "New Application" → name it your app name
3. Go to **OAuth2** in the sidebar
4. Copy the **Client ID** and **Client Secret** (you'll need these later)
5. Add redirect URL: `http://localhost:3000/api/auth/callback/discord`

#### Step 2: Link to Vercel

```bash
cd <your-app-name>
vercel link
```

Follow the prompts to connect to your Vercel account.

#### Step 3: Add GitHub Actions Secrets

Go to: `https://github.com/movement-guilds/<your-app-name>/settings/secrets/actions`

Add these secrets:

| Secret Name | Where to get it |
|-------------|-----------------|
| `DISCORD_CLIENT_ID` | Discord Developer Portal → OAuth2 |
| `DISCORD_CLIENT_SECRET` | Discord Developer Portal → OAuth2 |
| `NEXTAUTH_SECRET` | Run: `openssl rand -base64 32` |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase Dashboard → Settings → API |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase Dashboard → Settings → API |
| `DISCORD_GUILD_ID` | Discord → Server Settings → Widget → Server ID |
| `DISCORD_ADMIN_ROLE_ID` | Discord → Server Settings → Roles → Right-click role → Copy ID |
| `DISCORD_MOD_ROLE_ID` | Discord → Server Settings → Roles → Right-click role → Copy ID |
| `MOVEMENT_RPC_MAINNET` | `https://mainnet.movementnetwork.xyz/v1` |
| `MOVEMENT_RPC_TESTNET` | `https://testnet.movementnetwork.xyz/v1` |

#### Step 4: Add Vercel Environment Variables

Go to: Vercel Dashboard → Your Project → Settings → Environment Variables

Add the same variables as above, **plus**:

| Variable | Value |
|----------|-------|
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase Dashboard → Settings → API (keep secret!) |
| `UPSTASH_REDIS_REST_URL` | Upstash Console (optional, for rate limiting) |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Console (optional) |

#### Step 5: Push Database Migrations

```bash
# Set your Supabase project ref in .env.local
echo "SUPABASE_PROJECT_REF=your-project-ref" >> .env.local

# Link and push migrations
supabase link --project-ref $SUPABASE_PROJECT_REF
supabase db push
```

Find your project ref in: Supabase Dashboard → Settings → General → Reference ID

#### Step 6: Add Production Discord Redirect

After your first Vercel deploy:

1. Go back to Discord Developer Portal → your app → OAuth2
2. Add redirect: `https://<your-vercel-domain>/api/auth/callback/discord`

#### Step 7: Set Branch Protection (Optional)

Go to: `https://github.com/movement-guilds/<your-app-name>/settings/branches`

Recommended rules for `main` and `staging`:
- Require pull request reviews
- Require status checks to pass
- Require branches to be up to date

---

## Local Development

```bash
npm install
cp .env.example .env.local
# Fill in .env.local with your values (see .env.example for descriptions)
npm run dev
```

Open http://localhost:3000

---

## Environment Variables Reference

See [.env.example](./.env.example) for all available variables with descriptions.

---

## Contributing

See [CONTRIBUTING.md](https://github.com/movement-guilds/.github/blob/main/CONTRIBUTING.md).
