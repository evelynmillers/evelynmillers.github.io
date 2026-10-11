# Evelyn Miller — landing page

Single-page site for the two thirty-day challenges, selling through Selar.

## Files

- `index.html` — the whole page (HTML, CSS and JS in one file)
- `assets/` — hero photo and the two book covers
- `.nojekyll` — tells GitHub Pages to serve the files as-is

## Before you publish

Open `index.html` and replace these three placeholders:

| Find | Replace with |
|---|---|
| `SITE_URL` | your live domain, e.g. `evelynmiller.com` or `yourname.github.io/evelyn` |
| `[YOUR BRAND NAME]` | the business name that sells the books |
| `support@example.com` | the real support address |

`SITE_URL` appears in the social-share tags. Until it is set, links shared on
Facebook or WhatsApp will not show a preview image.

## Checkout links

All buttons go to Selar:

- Feel Younger Again — `selar.com/feel_younger_again`
- God Heals — `selar.com/16176qb598`
- Both, $20 — `selar.com/8tr2aw35i7`

## Swapping an image

Replace the file in `assets/` keeping the same filename. No HTML edit needed.

---

## The free devotional funnel

Two extra pages:

- `devotion/index.html` → `https://evelynmillers.github.io/devotion/` — email capture
- `devotion/sent/index.html` → `https://evelynmillers.github.io/devotion/sent/` — confirmation + one-hour offer

### To make it work

1. **Pick an email service** (MailerLite or Kit are both free at this volume).
2. In `devotion/index.html`, replace `EMAIL_SERVICE_FORM_URL` with that service's
   form POST endpoint. The field is already named `email`.
3. In the service, set the automation to send the devotional PDF on signup, and
   set the post-signup redirect to `https://evelynmillers.github.io/devotion/sent/`.
4. In `devotion/sent/index.html`, set the real offer: the price (`$6.99`), the
   struck price, and the coupon code (`DEVOTION30`).
5. **Create that coupon on Selar with a real expiry or a capped number of uses.**
   The page tells people the price ends in an hour. If the code still works a week
   later, the page is lying.

### The countdown

The deadline is stamped in `localStorage` the first time the page is opened and
then held, so refreshing does not restart it. When it runs out the discount is
genuinely withdrawn and the full price is shown instead.

A different browser or device starts a fresh hour. That is the limit of a static
page with no login, and it is why the Selar coupon needs its own real expiry.
