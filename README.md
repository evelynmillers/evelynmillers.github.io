# evelynmillers.github.io

Static site for Evelyn Millers' books. No build step, no dependencies, no
framework. Every page is a single self-contained HTML file with its CSS and
JS inline. To deploy, put the files in the repo root — GitHub Pages serves
them as-is.

```
index.html                  store / landing page
devotion/index.html         free devotional opt-in (MailerLite)
devotion/sent/index.html    thank-you page + timed offer
assets/                     all images
```

---

## Products and their links

| Product | Price | Selar | Used on |
|---|---|---|---|
| What Your Adult Child Wishes You Knew | $27 | `h9v900180y` | store, and the expired state of the thank-you page |
| ...same book, thank-you price | $17 | `051220678u` | thank-you page, first hour only |
| God Heals | $9.99 | `16176qb598` | store |
| Feel Younger Again | $14.99 | `feel_younger_again` | store |
| Both challenges | $19.99 | `8tr2aw35i7` | store |

Full form: `https://selar.com/<id>?add_to_cart=1`

**A product link appears in more than one place.** The $27 book link is in
the shelf button *and* in the picker's `copy` map down in the page script.
Change both together or the picker will quietly send people to a dead link.

---

## The timed offer on the thank-you page

The discount is real and the page enforces it.

- The deadline is stamped on first visit and stored in `localStorage` under
  `em_adultchild_deadline`.
- Refreshing does **not** restart it. This is the whole point — a clock that
  resets on reload is a lie the visitor can catch.
- When it runs out, `body.expired` is set: the `.live` block hides, the
  `.gone` block shows, and the page switches to $27 and the regular link.
- If `localStorage` is unavailable (private browsing), the window silently
  restarts. Failing in the visitor's favour is the right direction to fail.

**Known limit:** Selar does not enforce the deadline. Anyone who saved the
$17 URL can still use it afterwards. The page keeps its promise; the payment
processor doesn't know about it.

To change the window, edit `WINDOW_MS` in `devotion/sent/index.html`. If you
change the length, **also change the storage key**, or returning visitors
will still be on their old deadline.

---

## The email signup

`devotion/index.html` posts to MailerLite:

```
https://assets.mailerlite.com/jsonp/2723691/forms/201725173163885813/subscribe
```

It posts with `mode:'no-cors'`, so the response is opaque and cannot be read.
The page redirects to `sent/` either way and lets MailerLite's own email be
the confirmation. There is a 6-second timeout as a safety net.

The visible input is `name="email"` and a hidden `fields[email]` is filled in
at submit time. That split exists on purpose — see below.

---

## Things that are easy to break

These are all bugs that have already happened here. Each cost real debugging.

**Do not add `autocorrect="off"` or `spellcheck="false"` to the email field.**
On iOS the AutoFill email suggestion rides in the QuickType bar above the
keyboard, and those attributes hide that bar. `type="email"` already disables
autocorrect. There is a comment in the file saying this; leave it there.

**Focus must happen inside the tap.** The "send me the devotional" link calls
`focus()` synchronously and scrolls afterwards. Move the `focus()` into a
`setTimeout` and iOS refuses to open the keyboard — no keyboard, no AutoFill.

**Never `transform: scale()` a 3D book.** The rotating covers use
`transform-style: preserve-3d`. A scale anywhere up the ancestor chain
flattens the 3D context and the faces shear apart. The lead book on the
store page is bigger because every face has its own recomputed geometry
(`.lead .f-spine`, `.lead .f-top`, etc.), not because it's scaled.

**Spine text collides with the author line.** `.f-spine span` and
`.f-spine em` are both absolutely positioned. A long title needs the span
given an explicit bottom inset so it centres above the author, as
`.lead .f-spine span` does.

**Android `intent://` must not carry `S.browser_fallback_url` pointing back
at the same page.** That made a failed hand-off look like a page reload.
There is no programmatic escape from an iOS in-app browser at all — Apple
removed `x-safari-` and Instagram blocks the rest — so iOS copies the link
and tells the user where the `⋯` menu is.

**HTML `width`/`height` attributes beat CSS `aspect-ratio`.** Portrait images
need `height:auto` in CSS or they render as a sliver.

---

## Claims on the page

Everything stated as a fact should stay checkable.

- **"5,000+ followers across Instagram, Facebook and YouTube"** — a sum of
  real follower counts, rounded *down*, naming the three platforms it's
  summed from so a sceptic can click through and verify. Do not round up and
  do not drop the platform names.
- **No star ratings and no reviews** appear anywhere, because there are no
  real ones yet. When real reviews exist, the aggregate can go up. Inventing
  either is illegal in the US market this sells into (16 CFR Part 465).
- **"One year to change your mind"** is a real refund promise. It is the
  strongest trust asset on the site and no competitor offers it.

---

## Open items

- **Miller vs Millers.** The new book's cover and the thank-you page say
  *Millers*. The rest of the site, the Instagram handle
  (`evelynmiller_us`) and the contact address (`evelynmiller3716@gmail.com`)
  say *Miller*. Both spellings are currently visible on the same shelf.
  Unresolved.
- **No three-book bundle.** "Take both" at $19.99 now sits below a $27 single
  book. A three-book Selar product around $34.99 against $51.97 would fix the
  ladder.
- **Unused assets:** `evelyn-door.jpg`, `evelyn-window.jpg`,
  `evelyn-living.jpg` are in `assets/` but not referenced by any page.

---

## Local preview

Opening `index.html` by double-clicking it will show every image broken —
the paths are relative and there's no `assets/` folder next to it in your
Downloads. Serve the folder instead:

```sh
python3 -m http.server 8000
```

Then open `http://localhost:8000`.
