# Family dashboard — setup notes

Private dashboard for Dad's care team (family only). Static HTML served from `authorsite`; access gated by Cloudflare Access scoped to the `/family/*` path. Sensitive documents live on Google Drive, shared per-person.

- **Dashboard page**: `family/index.html` (this folder)
- **Target URL**: `https://adamplunkettauthor.com/family/`
- **Auth**: Cloudflare Access, email allowlist, path-scoped to `/family*`
- **Deep-read storage**: Google Drive folder, shared individually with family members
- **Rest of the site**: untouched — public, behaves exactly as today

---

## Your DNS situation

- **Registrar / DNS**: Squarespace Domains (confirmed from your DNS Settings screenshot)
- **Hosting**: GitHub Pages (CNAME file in repo points to `adamplunkettauthor.com`)
- **Change required**: move DNS authority from Squarespace to Cloudflare. Squarespace remains your registrar — this isn't a domain transfer, it's just switching which nameservers answer DNS queries.

---

## Step-by-step setup (order matters)

### 1. Sign up for Cloudflare and add your domain (~10 min)

1. Go to https://dash.cloudflare.com/sign-up and create a free account (if you don't have one).
2. Click **Add a site** and enter `adamplunkettauthor.com`. Pick the **Free** plan.
3. Cloudflare will auto-scan your existing DNS records at Squarespace and import them. Review them:
   - You should see the `_domainconnect` CNAME, the `google-site-verification` TXT, and likely a set of A records pointing at GitHub Pages (`185.199.108.153`, `.109.153`, `.110.153`, `.111.153`) or a CNAME to `adamplunkett.github.io`.
   - If the auto-scan misses anything, compare against the Squarespace DNS Settings page and add any missing records manually.
   - **Set every record's proxy status to "DNS only" (gray cloud)** for now. We'll turn on the orange cloud only where needed.
4. Cloudflare gives you two nameservers at the end of setup — something like `chloe.ns.cloudflare.com` and `fred.ns.cloudflare.com`. Copy these.

### 2. Change nameservers at Squarespace (~5 min; then wait)

1. In the Squarespace domain dashboard, left sidebar → **DNS** → **Domain Nameservers**.
2. Switch from Squarespace's default nameservers to **Custom nameservers**. Paste in the two Cloudflare nameservers.
3. Save.
4. Wait for propagation. Cloudflare will email you when it detects the change (usually 15 min – a few hours, up to 24 hrs worst-case). **Your site keeps resolving throughout** — there's no outage window.

### 3. Once Cloudflare says "active" — configure Access (~5 min)

1. In the Cloudflare dashboard, go to **Zero Trust** (left sidebar) → Accept the terms and pick the free plan if prompted → **Access** → **Applications** → **Add an application** → **Self-hosted**.
2. Configure:
   - **Application name**: `Dad — Family Dashboard`
   - **Session duration**: 1 month (or longer — family won't want to re-auth constantly)
   - **Application domain**: `adamplunkettauthor.com`, **Path**: `/family*`
3. **Policies** → **Add a policy**:
   - **Policy name**: `Family members`
   - **Action**: `Allow`
   - **Configure rules** → **Include** → **Emails** → paste the allowlist (you, Mom, Leah, David, Berit, Gabe, etc.)
4. **Identity providers**: enable **One-time PIN** (email magic code — no Google account required for family members).
5. Save.

**How it works for family**: They visit `https://adamplunkettauthor.com/family/`. Cloudflare prompts for their email, sends a 6-digit code, they enter it, they see the dashboard. Session lasts as long as you set. No passwords, no Google account required.

**Rest of your site**: Requests to `adamplunkettauthor.com`, `adamplunkettauthor.com/anything-else` — totally unaffected. Cloudflare passes them straight through.

### 4. Google Drive folder (~15 min)

1. In Google Drive (`adamplunkettauthor@gmail.com`), create folder `Dad — Family Health`.
2. Upload the source documents (see below for exact file list).
3. Share the folder with each family member **individually** via Drive's share UI (Viewer access). Add by email; they'll get a Drive notification.
4. **Get folder share link**: right-click folder → **Get link** → leave as **Restricted** (only people added can view) → **Copy link**. This is `DRIVE_FOLDER_URL` in the dashboard.
5. Also get share-links for individual files you want quick-linked from the dashboard (see below).

**Files to upload** (these live in `/Users/adamplunkett/Documents/Obsidian Vault/Dad Health/`):

**Overview**
- `family_summary_2026-04-21.MD` — long-form family summary
- `provider_outreach/00_clinical_brief.MD` — clinical brief

**Clinician outreach letters** (`provider_outreach/`)
- `01_current_psychiatrist.MD`
- `02_consult_psychiatrist_Gabe.MD`
- `03_GP.MD`
- `04_geriatrician.MD`
- `05_movement_disorder_neurologist.MD`
- `06_ketamine_clinic_Ann_Arbor.MD`
- `07_urologist.MD`

**Research syntheses** (`literature_reviews/` — consider creating a Drive subfolder)
- `01_sleep_movement_differential.md`
- `02_blind_spot_differential.md`
- `03_medication_safety_and_evidence.md`
- `04_heritability_family_history_treatment.md`
- `05_ketamine_anesthesia_biomarkers.md`
- `06_statin_neuropsych.md`
- `07_chronic_TRD_over_time.md`
- `08_guidelines_consensus_audit.md`
- `09_olanzapine_elderly_TRD_anxiety.md`
- `10_DIP_vs_unmasked_PD.md`
- `11_late_life_anxiety_TRD_alternatives.md`
- `12_ketamine_restart_maintenance.md`
- `13_lorazepam_elderly.md`
- `14_duloxetine_late_life_TRD.md`

**Explicitly NOT uploaded** (per your direction):
- `audit_2026-04-21.MD` and its v1 variant
- `probability_breakdown_2026-04-21.MD` and variants

> **Format tip**: Drive renders `.md` files as plain text. For family readability, you can either (a) leave as `.md` (fine for text-comfortable readers), or (b) open each in a text editor, paste into a new Google Doc, save. Google Docs renders markdown formatting cleanly. We can automate this later if it becomes a chore.

### 5. Paste Drive URLs into `index.html` (~5 min)

Open `family/index.html` and find the placeholders. Replace each with your actual Drive share URL:

- `DRIVE_FOLDER_URL` — whole folder share link
- `DRIVE_FAMILY_SUMMARY_URL` — family summary file link
- `DRIVE_CLINICAL_BRIEF_URL` — clinical brief file link
- `DRIVE_LETTER_PSYCH_URL` through `DRIVE_LETTER_UROLOGY_URL` — one per outreach letter
- `DRIVE_LITREVIEWS_URL` — literature reviews subfolder (or the main folder if you keep them flat)

### 6. Fill in the contact card

The contacts section at the bottom of the page has `[phone]`, `[Name]`, `[contact]` placeholders. Drop in actual values as providers come online. Keep updated.

### 7. Commit and push

```bash
cd /path/to/authorsite
git add family/
git commit -m "Add family health dashboard (Cloudflare Access-gated)"
git push
```

GitHub Pages rebuilds in 1–2 minutes. Visit `https://adamplunkettauthor.com/family/` — you should see the Cloudflare Access email prompt.

---

## Ongoing maintenance

- **Update the dashboard** as care decisions evolve. Edit `index.html`, push, bump the "Last updated" date in the masthead.
- **Add/remove family members** in the Cloudflare Access policy — changes apply on their next sign-in.
- **Add/remove Drive access** via the Drive share UI per-person.
- **Revoking access**: remove the email from the Cloudflare policy AND unshare from the Drive folder.

---

## Why this architecture

- **Static HTML**: zero backend, no DB, nothing to deploy beyond GitHub Pages.
- **Cloudflare Access**: real auth boundary — unauthenticated requests never see the HTML. Not a client-side JS polite gate.
- **Path-scoped policy**: everything outside `/family/` behaves exactly as today.
- **Per-email allowlist**: per-person revocation; no shared password to rotate.
- **Google Drive for deep docs**: leverages Drive's native per-user access control; no need to replicate it in the dashboard.

---

## Troubleshooting

- **Family member can't sign in**: check their email is in the Cloudflare Access policy (exact match, no typos). If Cloudflare sends a PIN but they never get it, check spam.
- **Page 404s at the path**: `/family/index.html` must be in the repo and pushed. Check GitHub Pages deploy status in the repo's Actions tab.
- **"This site can't be reached" during nameserver switchover**: DNS propagation in flight. Usually resolves within minutes to an hour. Don't panic.
- **Nothing changed after pushing**: GitHub Pages cache can take 1–5 min. Hard-refresh (Shift+Reload).

Questions → adamplunkett@gmail.com.
