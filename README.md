# Install Meta Ads CLI
### A Claude Code Skill by [@tenfoldmarc](https://www.instagram.com/tenfoldmarc)

Installs Meta's official **Ads CLI** (the `meta` command) on your computer and connects it to your Meta ad account — without you having to figure out any of the technical setup yourself. Claude walks you through every step, does all the installs for you, and only asks you to click buttons in the browser when Meta requires you to (creating an app, generating a token, etc.).

By the end, you can run commands like `meta ads campaign list` or `meta ads insights get --date-preset last_7d --fields spend,impressions,ctr,cpc` directly in your terminal.

---

## What It Does

1. Checks what you already have installed (Python, uv, the CLI itself) and skips ahead if anything is already set up.
2. Installs Python 3.12, the `uv` package manager, and the `meta-ads` CLI for you automatically.
3. Walks you click-by-click through creating a Meta Developer App in your browser.
4. Walks you through creating a System User in Meta Business Suite and assigning your ad account, Facebook Page, and app to it.
5. Pre-empts the "missing permissions" error Meta throws by enabling the right permissions on your app first.
6. Walks you through generating a never-expiring access token.
7. Saves your token securely to a `.env` file (you paste it in directly — never into chat).
8. Verifies everything works by running a real `meta auth status` and `meta ads campaign list` against your account.

Total time: about 15–20 minutes.

---

## Requirements

- A Mac, Linux, or Windows computer
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed and working
- A **Meta Business account** — confirm by going to https://business.facebook.com (if it loads with your business name, you're good)
- At least one ad account inside that business
- A Facebook Page you can use as the ad identity (typically your business page)

Don't worry about installing Python, `uv`, or `meta-ads` manually — the skill installs all of those for you on first run.

---

## Install

### Step 1 — Open your terminal

**Mac:** Press `Command + Space`, type **Terminal**, hit Enter.
**Windows:** Press `Win + R`, type **cmd**, hit Enter. (If you have Git Bash installed, use that instead — it works better with this skill.)
**Linux:** Open your terminal app.

### Step 2 — Run this command

Copy-paste this entire line and hit Enter:

```bash
git clone https://github.com/tenfoldmarc/install-meta-ads-cli-skill ~/.claude/skills/install-meta-ads-cli
```

You'll see some text scroll by. Wait until it finishes (takes a few seconds).

### Step 3 — Open Claude Code

In the same terminal window, type:

```bash
claude
```

Hit Enter. Claude Code will open.

### Step 4 — Run the skill

Type:

```
/install-meta-ads-cli
```

Hit Enter. The skill kicks off. Claude does as much as possible automatically and only pauses when you need to click something in the browser. Just follow the prompts.

---

## Usage

Once installed, you can do things like:

**See last 7 days of account performance:**
```bash
meta ads insights get --date-preset last_7d --fields spend,impressions,ctr,cpc
```

**List all your campaigns:**
```bash
meta ads campaign list
```

**Get JSON output for piping into other tools:**
```bash
meta --output json ads campaign list | jq '.[] | select(.effective_status == "ACTIVE")'
```

**Create a paused campaign for testing:**
```bash
meta ads campaign create --name "Test Campaign" --objective OUTCOME_LEADS --daily-budget 5000
```

(Budgets are in cents — `5000` = $50.00. Everything is created **paused** by default — you have to explicitly set `--status ACTIVE` to go live.)

For the full command list, run `meta --help` or see the [official Meta docs](https://developers.facebook.com/documentation/ads-commerce/ads-ai-connectors/ads-cli).

---

## Updating

To get the latest version:

```bash
cd ~/.claude/skills/install-meta-ads-cli && git pull
```

---

## Built By

[@tenfoldmarc](https://www.instagram.com/tenfoldmarc) — Follow for daily AI automation walkthroughs. Real systems, not theory.
