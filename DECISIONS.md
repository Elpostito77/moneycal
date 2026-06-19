# MoneyCal — Build Decisions Log

Running log of implementation choices, patterns, and things to stay consistent on across all calculators.

---

## Input Clamping (added 2026-06-14)

**Pattern:** Every `<input type="number">` has both an HTML `max`/`min` attribute AND a JS snap-back on the `input` event. The HTML attribute handles arrow-key controls; the JS handles typed values (which bypass `max`).

**Standard snippet — paste into every new calculator's `<script>` block before the initial `run()` call:**

```javascript
// Snap typed values back within min/max bounds immediately
(function() {
  document.querySelectorAll('input[type="number"]').forEach(function(el) {
    el.addEventListener('input', function() {
      var v = parseFloat(el.value);
      if (isNaN(v)) return;
      var mx = parseFloat(el.getAttribute('max'));
      var mn = parseFloat(el.getAttribute('min')) || 0;
      if (!isNaN(mx) && v > mx) el.value = mx;
      if (!isNaN(mn) && v < mn) el.value = mn;
    });
  });
})();
```

**Standard field limits used across calculators:**

| Field type | min | max |
|-----------|-----|-----|
| Dollar amounts (large) | 0 | 100,000,000 |
| Dollar amounts (income) | 0 | 10,000,000 |
| Dollar amounts (contributions) | 0 | 1,000,000 |
| Percentage rates | 0 | 50 (interest) / 100 (contrib %) / 300 (match %) |
| Years (retirement horizon) | 1 | 60 |
| Years (loan term) | 1 | 30 |
| Years (comparison) | 1 | 30 |

---

## Page Template (established 2026-06-12)

All pages share the same structure:

1. GA4 snippet (G-0LKD1BP2HB) — first thing in `<head>`
2. Favicon set: `/favicon.svg`, `/favicon-32.png`, `/apple-touch-icon.png`, `/manifest.json`
3. `theme-color: #1565c0`
4. Title format: `[Tool Name] — [Brief keyword phrase] | MoneyCal`
5. Meta description: 1–2 sentences, under 160 chars, keyword-natural
6. Canonical URL: `https://moneycal.co/finance/[slug].html`
7. Shared CSS: `../css/main.css`
8. Three JSON-LD blocks: SoftwareApplication, BreadcrumbList, FAQPage

---

## Schema Markup (established 2026-06-12)

Every calculator page gets three JSON-LD blocks:

- **SoftwareApplication** — `applicationCategory: "FinanceApplication"`, price 0 USD
- **BreadcrumbList** — Home → Finance → [Tool Name]
- **FAQPage** — 5 Q&As matching the visible `<details>` FAQ section

The visible FAQ `<details>` section and the FAQPage schema must match exactly (same questions, same answers).

---

## Content Structure (established 2026-06-12)

Each calculator page body:

1. `<div class="ad-slot lead">` — above the fold, before breadcrumb
2. Breadcrumb nav
3. `<h1>` with emoji + tool name
4. `.lede` paragraph (1–2 sentences, what the tool does)
5. `<section class="tool">` — the calculator inputs + result
6. `<div class="ad-slot">` — after tool, before content
7. `<section class="content">` containing:
   - How to use (h2 + ol or p)
   - How the math works (h2 + p)
   - Example with real numbers bolded (h2 + p)
   - `.note` disclaimer box
   - FAQ section (h2 + `.faq` with `<details>` elements)
   - `<div class="ad-slot">` inside content
   - Related tools list (h2 + `.related` ul)
   - Closing `<div class="ad-slot">`
8. Footer with disclaimer

**Content length target:** 250–400 words in the `.content` section (not counting FAQ). More text = better SEO; less text = better UX. Aim for the middle.

---

## Ad Slots (established 2026-06-12)

Three slots per page:
- `.ad-slot.lead` — very top, before breadcrumb
- `.ad-slot` — between tool and content
- `.ad-slot` — inside content section, after FAQ, before Related tools
- `.ad-slot` — end of content section

Slots contain the text "Advertisement" as placeholder until AdSense is configured.

---

## Calculator Math Approach (established 2026-06-12)

- All math runs client-side, no server calls
- Use period-by-period simulation (loop) where closed-form is hard to explain or debug
- Use closed-form formulas where standard and verifiable (loan amortization, FV)
- Always verify with Node.js before shipping — `node -e "..."` sanity checks against known values
- Return `r === 0` edge case explicitly (avoid divide-by-zero)

---

## Emoji Icons per Calculator (established 2026-06-12)

| Calculator | Emoji |
|-----------|-------|
| Compound Interest | 📈 |
| Debt Payoff | 💳 |
| Savings Goal | 🎯 |
| FIRE | 🔥 |
| Car Loan | 🚗 |
| Mortgage Payoff | 🏠 |
| 401(k) | 🏦 |
| Rent vs. Buy | 🔑 |
| Net Worth | 📊 |
| Emergency Fund | 🛟 |
| Savings Rate | 💰 |

---

## Deferred / Decided Against (updated as decisions are made)

| Feature | Decision | Reason |
|---------|---------|--------|
| PDF export (Net Worth) | **No** — use print CSS + shareable URL instead | PDF is a one-way door: user exports once, never returns. AdSense requires repeat visits. Print CSS gives "good enough" export with zero engineering cost. Shareable URL drives both return visits and sharing. |
| Shareable URL for Net Worth | **Yes — future** | Generate `?cash=X&investments=Y...` query params so users can bookmark their exact inputs. Drives return visits. Low effort. |
| Print CSS | **Yes — future** | 10-line CSS block that hides ads and nav when printing. Lets users self-export cleanly without a download button. |
| Account / login | **No** | Kills zero-friction value prop. No sign-up = fast tool. |

---

## Sitemap

Update `sitemap.xml` every time a new page is added. Format:

```xml
<url>
  <loc>https://moneycal.co/finance/[slug].html</loc>
  <changefreq>monthly</changefreq>
  <priority>0.8</priority>
</url>
```

Homepage uses `priority 1.0` + `changefreq weekly`. About page uses `priority 0.3`.

---

## GitHub / Deploy

- Repo: `Elpostito77/moneycal` (GitHub)
- Deploy: Vercel auto-deploys on push to `main`
- PAT stored in `~/.github_pat` (expires 2026-09-11)
- GA4 property: G-0LKD1BP2HB
- AdSense: apply ~2026-07-01 (2–3 weeks after launch traffic builds)
- GSC: domain property verified, sitemap submitted 2026-06-14
