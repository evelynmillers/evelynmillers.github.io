# evelynmillers.github.io

Static site for Evelyn Millers' books. No build step, no dependencies, no
framework. Every page is a single self-contained HTML file with its CSS and
JS inline. To deploy, put the files in the repo root — GitHub Pages serves
them as-is.

```
index.html                  store / landing page
devotion/index.html         free devotional opt-in (MailerLite) + the book
devotion/sent/index.html    thank-you page + timed offer
assets/                     all images
```

---

## Products and their links

| Product | Price | Selar | Used on |
|---|---|---|---|
| What Your Adult Child Wishes You Knew | $26.99 | `h9v900180y` | store, and the expired state of the thank-you page |
| ...same book, thank-you price | $16.99 | `051220678u` | thank-you page, first hour only |
| God Heals | $9.99 | `16176qb598` | store |
| Feel Younger Again | $14.99 | `feel_younger_again` | store |
| All three | $34.99 (was $51.97) | `8tr2aw35i7` | store bundle block |

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

## Voice

**The whole site is Evelyn speaking in the first person.** "My newest book",
"my three books", "all my books on Selar". Never "her" or "she" about
Evelyn — that reads as somebody writing *about* her on her own site, and it
breaks the only thing the page is selling.

The two correct uses of "her" are in the gift section ("Pick one for her" —
the mother being bought for) and the reader's own daughter. Both refer to
the reader's family, not to Evelyn.

The footer's advertisement disclosure is deliberately third person and
should stay that way.

---

## Things that are easy to break

These are all bugs that have already happened here. Each cost real debugging.

**`.f-front` and `.f-back` are sized by one shared rule.** The base is
`.f-front,.f-back{width:228px;height:332px}`. If you resize a book, restate
*both*. Overriding only `.f-back` leaves the cover at the old size inside a
bigger frame and it detaches from the page edges.

**Do not swap HTML blocks with a regex.** A non-greedy `.*?</div>` match ate
the characters either side of the bundle block and left a stray `v>` printing
on the page. Edit by exact string, and run a parser over the result.

**The devotional hero shows both books above the fold.** Free devotional and
the paid book sit side by side, each with its own button, and both clear the
fold at 375x667 as well as 390x844. The hero used to be 1493px tall with the
email field at 1157px — nobody saw the signup without scrolling. Anything
added above the tiles pushes them off again; measure, don't eyeball.

**`.stage` centres vertically, not horizontally.** It is
`flex-direction: column`, so `justify-content: center` does nothing for
horizontal placement. Inside the hero grid its column held it; standing on
its own in `.stagewrap` it needed `align-items: center` or it sat 368px left
of centre on desktop.

**Bind every jump link, not the first.** The devotional page has more than
one `a[href="#em"]` now (the book section and the closing section). The
handler uses `querySelectorAll(...).forEach`. A `querySelector` there binds
only the first and the others silently lose their focus behaviour.

**The thank-you page offer must stay above the fold.** The cover, title,
price, clock and button all fit on the first screen at 390x844 — the button
lands at ~633px. Adding anything above the offer, or letting the cover go
full width on mobile again, pushes the button off-screen; it used to sit at
1768px, two screens down. Measure after changing that page, don't eyeball it.

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
  (`evelyn.m_us`) and the contact address (`evelynmiller3716@gmail.com`)
  say *Miller*, while the Threads handle (`evelyn.millers.us`) says
  *Millers*. The handles no longer agree with each other either. Unresolved.
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
