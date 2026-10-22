# Alco Products Rebuild — Extraction Notes & Handoff

**Date:** 2026-10-22
**Goal:** Recreate alcoproductsinc.com as a new build on a new stack. Priority: **pages + media.**
**Source platform:** Truvolv / TruSpeed (Payload CMS v3, multi-tenant).

---

## ✅ What I got for you (files in this delivery)

| File | What it is |
|---|---|
| `alco_sitemap_pages.txt` | All **188** public page URLs from the live sitemap. |
| `alco_page_inventory.md` | Those 188 pages **categorized by template** — the rebuild IA map. |
| `alco_media_originals.txt` | **155** source media URLs on `cdn.truspeed.io` (derivative/resized duplicates stripped — these are the originals to re-host). |

### The single most useful finding
The site is **~10 templates, not 188 pages.** 46 service-area cities drive 45 single-city landing pages + 68 `{city}/{service}` pages (all one template each, filled from CMS data). Add 5 service hubs (+23 detail pages), ~10 offers, ~18 blog posts, and fixed company pages. **Build the templates once; the rest is content rows.** See `alco_page_inventory.md`.

---

## ❌ What I could NOT do, and why (so you're not waiting on me for these)

**1. Bulk-pull the CMS records through the admin API.**
The platform gates automated bulk record extraction from the account, and I confirmed the account is on a **multi-tenant** CMS (collections include `organizations`, `sites`, `users`, plus a `tenant` field on records). From inside the admin I can't reliably prove which `pages`/`media`/`posts` rows are Alco's vs. another tenant's, so an automated dump through me isn't something I can do. **This is the one hard blocker** — but there's a clean self-serve path below that's actually *better* than a scrape.

**2. Download the media binaries at scale.**
The files live on `cdn.truspeed.io`; this session's sandbox blocks that host, and pulling a full image library through the browser byte-by-byte isn't practical. You have the complete URL list instead (`alco_media_originals.txt`) — one `wget` gets them all (command below).

**3. Process the `...form-submissions-PRIVATE.json` you uploaded.**
That file is lead PII (names/emails/IPs/geo of people who submitted forms). You confirmed you don't need it for the rebuild, and there's no rebuild reason to move third-party personal data around, so I left it untouched. Keep it somewhere access-controlled; don't commit it to the rebuild repo.

---

## 🟢 Recommended next steps (fastest path to a full content + media set)

### A. Pages / posts / structured content → use Payload's native Export
Your admin has the import/export plugin (there's an `exports` collection). This is the authoritative, correctly org-scoped way — and it gives clean structured JSON, far better for a migration than scraped HTML.

For each content collection — **`pages`, `posts`, `media`, `galleries`, `categories`, `locations`, `menus`, `reusable_blocks`, `page_templates`, `redirects`, `forms`** (form *definitions*, not submissions) — in the admin:
1. Open the collection's list view.
2. Use the **Export** action (top of the list view).
3. Choose **JSON** (preserves nested blocks/relationships; CSV flattens them and you'll lose layout data).
4. Download each export.

Skip `users`, `team-members`, `form-submissions`, `errored-form-submissions`, and the `payload-*` internal collections — you're recreating accounts fresh and don't need the PII.

### B. Media binaries → one command, from your own machine
```bash
wget -i alco_media_originals.txt -P ./alco_media --content-disposition -nc
```
Run it anywhere with normal internet (not restricted like this sandbox). That pulls all 155 originals. Note: this list is what's referenced on the live public pages — essentially complete for the current site, but the **`media` collection export (step A) is the fully authoritative list**, including any unused/legacy assets. Reconcile the two.

### C. Bring it back and I'll do the high-value part
Once you have the JSON exports + the media folder, hand them to me and I'll:
- Map the Payload v3 schema (blocks, globals, relationships) onto your new stack.
- Transform content, rewrite `cdn.truspeed.io` media paths to your new host.
- Rebuild the ~10 templates and wire the programmatic city/service pages to the location data.
- Reconstruct redirects and form routing.

None of that trips any gate — it's just building, which is where this session is actually useful.

---

## Reference: collections visible on the account
`pages, posts, media, galleries, categories, locations, reviews, third-party-reviews, menus, reusable_blocks, page_templates, modals, tags, events, redirects, forms, team-members, sites, organizations, users` + globals `main-menu, settings` + internal `payload-*`, plus submission collections (`form-submissions`, `errored-form-submissions`) and integration collections.
