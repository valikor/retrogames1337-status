# Operator setup checklist

The orchestrator handled `gh repo create` + the initial push. Everything
below is a manual GitHub UI / DNS step that the CLI can't fully
automate. Do them in order.

## 1. Repo creation

Already done by orchestrator (`gh repo create valikor/retrogames1337-status
--public --source=. --push`). No action needed.

## 2. Enable GitHub Pages

1. Open the repo Settings → **Pages**.
2. Under **Build and deployment** → **Source**, choose
   **"Deploy from a branch"**.
3. **Branch:** `gh-pages` — this branch does NOT exist yet at first
   push. It is auto-created by the first successful run of the
   **Static Site CI** workflow (or by manually triggering **Uptime CI**
   once, then **Static Site CI**). Refresh after ~5–10 minutes if you
   don't see `gh-pages` in the dropdown.
4. **Folder:** `/ (root)`.
5. Save.

## 3. Custom domain — `status.retrogames1337.com`

1. In **Settings → Pages → Custom domain**, enter
   `status.retrogames1337.com` and save. GitHub will write a `CNAME`
   file to the `gh-pages` branch automatically.
2. In **Route 53** for the `retrogames1337.com` hosted zone, add:

   | Record | Type | Value | TTL |
   | --- | --- | --- | --- |
   | `status.retrogames1337.com` | `CNAME` | `valikor.github.io` | 300 |

3. Wait for GitHub's DNS check (a green check appears under the custom
   domain field — usually 1–5 min after DNS propagates).
4. Tick **"Enforce HTTPS"** once GitHub finishes provisioning the
   Let's Encrypt cert (can take up to ~30 min after DNS check passes).

## 4. (Optional) Discord notifications

1. In your Discord server, create a webhook for the channel you want
   incident pings in (Server Settings → Integrations → Webhooks → New).
   Copy the webhook URL.
2. In repo **Settings → Secrets and variables → Actions →
   New repository secret**:
   - **Name:** `DISCORD_WEBHOOK_URL`
   - **Value:** the webhook URL you copied
3. Done — `notifications` block in `.upptimerc.yml` already references
   `$DISCORD_WEBHOOK_URL`, so the next status change will fire a ping.

## 5. (Optional) Personal Access Token

The default `GITHUB_TOKEN` is fine for almost every upptime feature. If
the first run fails with a permissions error on commits to `gh-pages`,
generate a classic PAT with `repo` + `workflow` scopes and add it as
secret **`GH_PAT`**. The workflows already prefer `GH_PAT` over
`GITHUB_TOKEN` when present.

## 6. Kick off the first run

1. Go to the **Actions** tab. If GitHub asks you to enable workflows
   for a forked-style template, click **Enable**.
2. Open **Uptime CI** → **Run workflow** → select `main` → run.
3. Once it finishes (≈30 s), open **Static Site CI** → **Run workflow**.
   This creates the `gh-pages` branch.
4. Return to **Settings → Pages** and finish step 2 above (selecting
   the `gh-pages` branch) if you weren't able to before.
5. The status site goes live at the custom domain within ~5–10 minutes
   after the first `gh-pages` deploy.

## 7. Sanity check

- Visit https://status.retrogames1337.com — should render a green
  "All systems operational" page once first uptime check passes.
- Visit the repo Issues tab — no `down` issues open means everything
  is healthy.
- Open `history/` on the `master` branch — you should see one JSON
  file per monitored endpoint, updated by the bot every 5 min.

## Troubleshooting

- **`gh-pages` never appears**: the **Uptime CI** workflow must run
  once successfully first. Check the Actions tab for failures.
- **DNS check fails**: confirm the CNAME record by running
  `dig +short status.retrogames1337.com CNAME` — it should print
  `valikor.github.io.`.
- **Discord pings not firing**: confirm the secret name is exactly
  `DISCORD_WEBHOOK_URL` (no trailing space) and that the webhook URL
  is for a channel you can post to.
