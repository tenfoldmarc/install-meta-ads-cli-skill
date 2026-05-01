---
name: install-meta-ads-cli
description: Install Meta's official Ads CLI (the `meta` command) on a beginner's machine. Walks the user through installing Python 3.12+ via uv, installing the meta-ads package, creating a Meta Developer App, creating a System User in Meta Business Suite, assigning assets, adding the missing app permissions, generating a never-expiring system user access token, saving credentials to .env, and verifying the install with `meta auth status` and a real campaign list call. Use when the user asks to install the Meta Ads CLI, set up the meta command, or connect Claude Code to Meta Ads via CLI.
---

# Install Meta Ads CLI — Beginner Walkthrough

This skill installs Meta's official **Ads CLI** (https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli) on the user's machine and gets it authenticated against their Meta ad account.

**Audience:** Total beginners. Assume nothing is installed. Assume they've never created a Meta Developer App or a System User. Assume they don't know what `pip` or `uv` are. Walk them through everything.

**Your job as Claude:** Do as much as possible yourself via Bash. Only ask the user to do things you can't do (browser clicks in Meta Business Suite, copying tokens). Verify each step works before moving on.

---

## How to run this skill

Run the phases **in order**. After each phase, briefly tell the user what just happened, then move to the next. Don't dump all phases at once — the user will get lost. Only show browser instructions when the user actually needs to act.

If the user has already done some of this (e.g., they already have a Meta Developer App, or `meta` is already installed), detect it via Bash checks and skip ahead.

---

## Prerequisites the user needs

Tell them up front:

1. A Mac, Linux, or Windows machine (this skill is written for macOS — adapt commands for other OSes if needed).
2. A **Meta Business account** with at least one ad account inside it. They can confirm by going to https://business.facebook.com — if it loads with their business name, they're good.
3. About **15–20 minutes**.

If they don't have a Meta Business account, stop and tell them to create one at https://business.facebook.com first.

---

## Phase 0 — Detect existing state

Before doing anything, run these checks silently and decide where to start:

```bash
which meta                                    # is the CLI already installed?
which uv                                      # is uv already installed?
python3 --version; python3.12 --version       # Python versions available
test -f .env && grep -E "^(ACCESS_TOKEN|AD_ACCOUNT_ID)=" .env  # existing creds
```

- If `meta --version` prints a version AND `meta auth status` says authenticated → skill is done, just confirm with the user.
- If `meta` is installed but `meta auth status` fails → skip to Phase 4 (token generation).
- Otherwise → start at Phase 1.

---

## Phase 1 — Install Python 3.12+, uv, and the CLI

Tell the user: "I'm going to install three things: Python 3.12, the `uv` package manager, and the `meta-ads` CLI itself. Sit back — I'll handle this."

### 1a. Check Python

```bash
python3.12 --version 2>&1 || echo "MISSING"
```

If missing, install via Homebrew (macOS) or direct download. On macOS:

```bash
brew install python@3.12
```

If they don't have Homebrew, walk them through installing it from https://brew.sh first.

### 1b. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
export PATH="$HOME/.local/bin:$PATH"
uv --version
```

### 1c. Install meta-ads

```bash
uv tool install meta-ads --python "$(which python3.12)"
```

⚠️ **Common gotcha:** If `pip install meta-ads` is attempted directly on a uv-managed Python, it fails with `error: externally-managed-environment`. Always use `uv tool install` instead — it handles the virtual environment automatically.

### 1d. Verify

```bash
meta --version
```

Should print `meta, version 1.0.x`. If `meta` is not found, the user needs `~/.local/bin` in their PATH. Add to their shell profile:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

(Or `~/.bashrc` if they use bash.)

---

## Phase 2 — Create a Meta Developer App

This is something **only the user can do** — it's a browser flow with their Facebook login. Walk them through it click-by-click.

Send them this:

> ## 📍 Phase 2 — Create a Meta Developer App
>
> 1. Go to **https://developers.facebook.com/apps/**
> 2. Click the green **"Create App"** button (top right).
> 3. **Use case** → select **"Create & manage ads with Marketing API"** → click **Next**. (Picking this use case auto-adds the Marketing API product and pre-enables the right permissions — much less to configure later.)
> 4. **App type** → if asked, select **"Business"** → click **Next**.
> 5. **Details:**
>    - **App name:** `Ads CLI` (or whatever you want)
>    - **App contact email:** your email
>    - **Business account:** pick your business
> 6. Click **"Create App"**. May ask for your Facebook password.
> 7. You'll land on the app dashboard. Look at the URL — it'll look like `developers.facebook.com/apps/1234567890/dashboard/`. **Copy the App ID** (the number) and paste it to me.
>
> ### 🚨 Now do this BEFORE leaving the app dashboard — saves you regenerating the token later
>
> 8. In the left sidebar click **App Settings → Basic**.
> 9. Scroll down to **Privacy Policy URL** and paste a working URL (any public privacy policy page on a domain you control). If you don't have one, the fastest path is `https://[your-domain].com/privacy` — make sure the page actually loads in a fresh tab, otherwise Meta will block it. (No domain? Use a free Notion page set to public, or a GitHub Pages doc — Meta just needs the URL to load.)
> 10. Scroll down further to **Category** and pick something like "Business and Pages" (required to flip the app to Live).
> 11. Click **Save Changes** at the bottom.
> 12. Now look at the **top of the page** — there's a toggle that says **"In development"**. Click it and confirm to flip the app to **"Live"**.
>
> **Why this matters:** If you skip steps 8–12, the token you generate in Phase 5 will work for the install check but will silently fail the moment you try to create real ad creatives — Meta returns "app is in development mode" and you'll have to come back, flip the app, AND regenerate a new token (the old one is permanently stuck in dev-mode). Doing it now = token works for everything from day one.

When they give you the App ID, save it for later (you'll reference it in Phase 4 when they pick the app to generate the token from).

---

## Phase 3 — Create a System User and assign assets

Same deal — only the user can do this in the browser. Send them this:

> ## 📍 Phase 3 — Create a System User
>
> A "system user" is like a robot account inside your business that holds API tokens. It's better than personal tokens because it doesn't expire when you change your password.
>
> 1. Go to **https://business.facebook.com/settings/system-users**
>    - If asked, pick the business that holds your ad accounts.
> 2. Click the blue **"Add"** button (top right).
> 3. In the dialog:
>    - **Username:** `Ads CLI`
>    - **Role:** **Admin** (NOT Employee — Admin is required)
>    - Accept the agreement
>    - Click **"Create system user"**
> 4. You'll see `Ads CLI` in the list. **Click on it** so it's selected.
> 5. Click **"Assign assets"** on the right side.
> 6. **Assign these one at a time** — click "Assign assets" for each:
>    - **Ad accounts** → check the ad account you want to manage → toggle ON **"Manage ad account"** → Save
>    - **Pages** → check the Facebook Page you'll use as ad identity → toggle ON **"Manage Page"** → Save
>    - **Apps** → check the app you just created in Phase 2 → toggle ON **"Manage app"** → Save
>    - (Optional) **Catalogs** and **Datasets** if you'll use catalog ads or Pixels
>    - (Optional) **Instagram accounts** if you'll run Instagram placements
>
> Tell me **"done"** when these are all assigned.

---

## Phase 4 — Add missing app permissions

Tell the user: "When you generate the token, Meta will probably complain that some permissions aren't enabled on your app. Let's pre-empt that."

Send them this:

> ## 📍 Phase 4 — Enable permissions on your app
>
> 1. Open https://developers.facebook.com/apps/{APP_ID}/ (replace `{APP_ID}` with the App ID from Phase 2).
> 2. In the **left sidebar**, find **"App Review"** → click to expand → click **"Permissions and Features"**.
> 3. For **each** of these permissions, search for it → click **"Get advanced access"** → confirm:
>    - `business_management`
>    - `ads_management`
>    - `pages_show_list`
>    - `pages_read_engagement`
>    - `pages_manage_ads`
>    - `catalog_management`
>    - `read_insights`
>
> Tell me **"done"** when you've added them all.

⚠️ **Note:** Meta may say "Advanced Access" requires App Review for production. Since they're using their **own** system user on their **own** app, the permissions usually work immediately for self-use without going through review. If Meta blocks them and demands business verification, they'll need to complete it (usually requires a business document) before generating the token.

---

## Phase 5 — Generate the access token

Send them this:

> ## 📍 Phase 5 — Generate the token
>
> 1. Go back to https://business.facebook.com/settings/system-users
> 2. Click on the **Ads CLI** system user.
> 3. Click **"Generate new token"** on the right.
> 4. In the dialog:
>    - **Select app:** pick **Ads CLI** (the app from Phase 2)
>    - **Token expiration:** **"Never"**
>    - **Permissions/Scopes:** check ALL of these 7 boxes:
>      - ☑ `business_management`
>      - ☑ `ads_management`
>      - ☑ `pages_show_list`
>      - ☑ `pages_read_engagement`
>      - ☑ `pages_manage_ads`
>      - ☑ `catalog_management`
>      - ☑ `read_insights`
>    - ⚠️ If Meta auto-checks `threads_business_basic`, **uncheck it** — not needed.
> 5. Click **"Generate token"**.
> 6. **Copy the token immediately** — you only see it once. It starts with `EAA...`.
>
> ⚠️ **Security:** Don't paste the token into chat. I'll open your `.env` file and you'll paste it directly there.

---

## Phase 6 — Save credentials to .env

Once they've copied the token:

### 6a. Find their working directory's .env

Decide where to put it:
- If working in a specific project, use that project's `.env`.
- Otherwise default to `~/.env` or ask them where they want it.

### 6b. Get their ad account ID

If they don't know it, walk them: Go to https://business.facebook.com/settings/ad-accounts → click their ad account → the ID looks like `act_1234567890`. (CLI requires the `act_` prefix.)

### 6c. Add placeholder lines to .env and open it

```bash
# Append the block (DON'T overwrite existing .env)
printf '\n# Meta Ads CLI\nAD_ACCOUNT_ID=act_THEIR_ID_HERE\nACCESS_TOKEN=\n' >> /path/to/.env
open -a TextEdit /path/to/.env   # macOS — use appropriate editor for other OSes
```

### 6d. Tell them what to do

> The file is open in TextEdit. Scroll to the bottom. You'll see:
> ```
> # Meta Ads CLI
> AD_ACCOUNT_ID=act_...
> ACCESS_TOKEN=
> ```
> Click after `ACCESS_TOKEN=` and paste your token. No quotes, no spaces. Save (`Cmd+S`) and close.
> Tell me **"saved"** when done.

### 6e. Verify (without printing the token)

```bash
tok=$(grep "^ACCESS_TOKEN=" /path/to/.env | cut -d= -f2-)
if [ -z "$tok" ]; then
  echo "EMPTY — token not saved"
else
  echo "Token present: ${#tok} chars, starts ${tok:0:6}..., ends ...${tok: -4}"
fi
```

---

## Phase 7 — Verify the install works

```bash
# Load .env and check auth
set -a; source /path/to/.env; set +a
meta auth status
meta ads adaccount current
meta ads campaign list
```

Expected:
- `meta auth status` → `Authenticated (token: EAA...XXXX)`
- `meta ads adaccount current` → prints their `act_...` ID
- `meta ads campaign list` → table of their campaigns

If `meta auth status` fails:
- Re-check `.env` formatting (no quotes around the token, no spaces)
- Make sure the user generated the token from the right app
- Check the system user was assigned to the ad account in Phase 3

If `campaign list` returns "permission denied":
- The system user wasn't assigned the ad account → go back to Phase 3, step 6
- Or the token doesn't have `ads_management` scope → regenerate with all 7 scopes

---

## Phase 8 — Run a real query to celebrate

Show them what they can do now:

```bash
# Last 7 days account performance
meta ads insights get --date-preset last_7d --fields spend,impressions,ctr,cpc

# List active campaigns
meta ads campaign list

# List Pages they can use as ad identity
meta ads page list
```

Then tell them:

> 🎉 **You're set up.** Try these:
>
> - `meta --help` → see every command
> - `meta ads insights get --date-preset last_30d --fields spend,impressions,clicks,ctr` → last 30d numbers
> - `meta ads campaign list` → all campaigns
> - Output formats: append `--json` or `--plain` for scripting (NOT `--output`)
>
> **Gotchas:**
> - Budgets are in **cents** (`--daily-budget 5000` = $50.00)
> - Everything created **PAUSED** by default — must `--status ACTIVE` to go live
> - Cascading deletes: deleting a campaign kills its ad sets + ads

---

## Reference: Common errors and fixes

| Error | Cause | Fix |
|---|---|---|
| `error: externally-managed-environment` | Tried `pip install` on uv-managed Python | Use `uv tool install meta-ads` instead |
| `meta: command not found` after install | `~/.local/bin` not in PATH | Add `export PATH="$HOME/.local/bin:$PATH"` to `~/.zshrc` |
| `Authentication failed` from `meta auth status` | Wrong/missing token in `.env` | Re-check `.env`, regenerate token if needed |
| `(#100) The parameter ad_account_id is required` | `AD_ACCOUNT_ID` not loaded | `source .env` before running, or pass `--ad-account-id act_...` |
| `(#200) Requires extended permission: ads_management` | Token missing scope | Regenerate token with all 7 scopes |
| `Error: No such option: --output` on `ads campaign list` | `--output` is a top-level flag, not a subcommand flag | Use `meta --output json ads campaign list` (top-level) |
| `Error: No such option: --level` on `insights get` | CLI's insights only operate at scope of filter (`--ad-id`, `--adset-id`, `--campaign-id`) | Filter by ad/adset/campaign ID for per-entity insights — not `--level` |
| Permissions check shows `threads_business_basic` selected | Meta auto-suggests it | Uncheck — not needed |
| `Ads creative post was created by an app that is in development mode` (subcode 1885183) | App was in Dev mode when token was minted, OR Privacy Policy URL is missing | Phase 2 fix: add Privacy Policy URL in App Settings → Basic, save, then flip toggle to "Live" at top of dashboard. **Then regenerate the token** (the old one is permanently stuck in dev-mode). |

---

## Phase 9 — Save to memory (optional, only if user has auto-memory)

If the user has a memory system at `~/.claude/projects/.../memory/`, save:

- App ID, ad account ID, system user name
- Confirmation that CLI is installed and working
- Date of install

This way future Claude sessions know it's already set up.
