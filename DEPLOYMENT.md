# Deployment: GitHub → Cloudflare Pages → danielu.xyz

End-to-end walkthrough. You'll do three things:
1. Push this folder to a new GitHub repo.
2. Connect that repo to Cloudflare Pages (zero-config static deploy).
3. Point `danielu.xyz` at Cloudflare Pages and remove the Bolt mapping.

Total time: about 15 minutes.

---

## Part 1 — Push to GitHub

### 1.1 Create the repo on GitHub

1. Go to https://github.com/new
2. Repository name: **`danielu-portfolio`** (or any name you prefer)
3. Visibility: your choice — **Public** is fine; **Private** also works with Cloudflare Pages.
4. Do **NOT** initialize with a README, .gitignore, or license (this folder already has them).
5. Click **Create repository**.

### 1.2 Push this folder from your Mac

Open Terminal and run:

```bash
cd "/Users/admin/Library/CloudStorage/Dropbox/Professional_Development Branding Jobs/2026 Job Search/danielu-portfolio"

git init
git add .
git commit -m "Initial portfolio site"
git branch -M main
git remote add origin https://github.com/YOUR-GITHUB-USERNAME/danielu-portfolio.git
git push -u origin main
```

Replace `YOUR-GITHUB-USERNAME` with your actual GitHub username.

If you've never used git on this machine before, you'll be prompted to sign in to GitHub. Easiest flow: install **GitHub Desktop** (https://desktop.github.com), sign in once, and either use the GUI or let it set up your credentials for the command line.

---

## Part 2 — Deploy to Cloudflare Pages

### 2.1 Create the Pages project

1. Log into Cloudflare → https://dash.cloudflare.com
2. In the left sidebar, go to **Workers & Pages** → **Create** → **Pages** → **Connect to Git**.
3. Click **Connect GitHub**. Authorize Cloudflare's GitHub app for your account. If you made the repo Private, you'll need to grant Cloudflare access to just this repo (recommended).
4. Pick the **`danielu-portfolio`** repo.
5. On the **Set up builds and deployments** screen, use:
   - **Project name**: `danielu-portfolio` (this becomes `danielu-portfolio.pages.dev`)
   - **Production branch**: `main`
   - **Framework preset**: **None**
   - **Build command**: *(leave empty)*
   - **Build output directory**: `/` *(or leave empty)*
   - **Root directory**: *(leave empty)*
6. Click **Save and Deploy**.

Cloudflare will deploy in ~30 seconds. You'll get a live URL like `https://danielu-portfolio.pages.dev`. Open it and confirm the site renders correctly.

From now on, every `git push` to `main` auto-deploys.

---

## Part 3 — Point danielu.xyz at Cloudflare Pages

How this works depends on whether **danielu.xyz** is currently using Cloudflare for DNS.

### Check first: is danielu.xyz already on Cloudflare?

In the Cloudflare dashboard main page, do you see `danielu.xyz` listed as one of your sites?

- **Yes** → follow **Path A** below.
- **No** → follow **Path B** below.

---

### Path A — danielu.xyz is already on Cloudflare

This is the easy path.

1. In the Pages project dashboard, go to **Custom domains** → **Set up a custom domain**.
2. Enter `danielu.xyz` (apex) and click **Continue**.
3. Cloudflare will offer to add the DNS records automatically. Accept.
4. Repeat with `www.danielu.xyz` if you want the `www` subdomain to also work.
5. **Remove the old Bolt mapping**. In Cloudflare DNS for `danielu.xyz`, find any `CNAME` record pointing to `*.bolt.host` and delete it. The new Pages records should be the only ones for the root domain.

The site should be live on `https://danielu.xyz` within a minute or two.

---

### Path B — danielu.xyz is NOT on Cloudflare yet

You need to move DNS to Cloudflare first. Two sub-steps:

#### B.1 Add the domain to Cloudflare

1. In the Cloudflare dashboard main page, click **Add a site**.
2. Enter `danielu.xyz` → Continue.
3. Choose the **Free** plan.
4. Cloudflare will scan your existing DNS records. Verify the list — importantly, if there's a CNAME pointing to anything `.bolt.host`, you can leave it for now; we'll remove it after Pages is attached.
5. Cloudflare gives you two **nameservers** (e.g., `ada.ns.cloudflare.com` and `kai.ns.cloudflare.com`). Copy them.

#### B.2 Update nameservers at your registrar

Find out where `danielu.xyz` is registered. Common `.xyz` registrars include:
- **Namecheap** — Dashboard → Domain List → Manage → Nameservers → Custom DNS
- **Porkbun** — Account → Domains → the domain → Authoritative Nameservers
- **Cloudflare Registrar** — already on Cloudflare, so skip
- **GoDaddy** — My Domains → DNS → Nameservers → Change

Replace the current nameservers with the two Cloudflare ones. DNS propagation usually takes 10–60 minutes. Cloudflare will email you when it's active.

#### B.3 Attach to Pages (after nameservers are live)

Follow **Path A** above — now that the domain is on Cloudflare, the custom-domain flow in Pages will just work.

---

## Part 4 — Confirm and clean up

1. Open `https://danielu.xyz` in an incognito window. Confirm you see the new Swiss portfolio.
2. Open `https://danielu-portfolio.pages.dev` — confirm it also works (this is your deploy-status URL and should always match `danielu.xyz`).
3. In Bolt, the old `bolt.host` site will still exist but will no longer be mapped to your custom domain. You can leave it as a backup or delete the project.
4. Double-check the SSL padlock on `https://danielu.xyz`. Cloudflare provisions a free cert automatically; it should show up within a few minutes. If it doesn't, in Cloudflare → SSL/TLS → set mode to **Full** or **Full (strict)**.

---

## Future edits

Any change you make locally, then:

```bash
cd "/Users/admin/Library/CloudStorage/Dropbox/Professional_Development Branding Jobs/2026 Job Search/danielu-portfolio"
git add .
git commit -m "describe the change"
git push
```

Within about 30 seconds, `danielu.xyz` reflects the change. No build step required.

If you'd rather edit in a browser: the GitHub web editor (press `.` on the repo page) is great for quick copy tweaks.

---

## Troubleshooting

**Site deploys but fonts look wrong.** Cloudflare's Rocket Loader can sometimes interfere with Google Fonts. In your Cloudflare dashboard → Speed → Optimization → turn Rocket Loader **OFF**.

**Site says "Error 526" or SSL certificate issue.** In Cloudflare → SSL/TLS → set mode to **Full**. The free Cloudflare cert needs a minute to issue.

**danielu.xyz still shows the old Bolt site after DNS change.** Clear your browser cache / try an incognito window. If still stuck after 30 minutes, confirm the old `.bolt.host` CNAME was actually removed from the DNS table.
