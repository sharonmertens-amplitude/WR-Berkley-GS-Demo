# Lavalier Customer Center — WR Berkley Guides & Surveys demo

A fictional jewelry-insurance self-service portal, built to demo Amplitude **Guides & Surveys**, **Analytics**, and **Session Replay** together on a surface that resembles a real WR Berkley customer experience.

Single self-contained `index.html`. No build step, no dependencies. Same deploy pattern as the CDK dealership site.

---

## 1. The one thing you have to change

Open `index.html`, find this near the top, and paste in your **demo/sandbox project key**:

```js
var AMPLITUDE_API_KEY = "PASTE_YOUR_PROJECT_API_KEY_HERE";
```

Two lines below it:

```js
var SERVER_ZONE = "US";                  // set to "EU" if the project lives in the EU region
var SESSION_REPLAY_SAMPLE_RATE = 1;      // 1 = capture every session (what you want live)
var SIMULATED_GUIDES = true;             // built-in demo guides on/off — see section 3a
var DEMO_USER_ID = "demo-dana-whitfield";// one identity across all three products
```

`DEMO_USER_ID` matters more than it looks. Without it every event lands as "Anonymous User," your own session is hard to find in Session Replay, and Guides & Surveys targeting rides on a rotating device ID instead of a stable person. Change the string if two people are demoing at the same time, so you don't collide.

That's it. The page works fully **before** you add a key — every guide and survey runs locally — so you can click through it right now.

---

## 2. How all three products get on the page

This uses Amplitude's **Browser Unified Script**, which is the closest thing to "one SDK that covers everything":

| Product | How it loads |
|---|---|
| Analytics | Included in the unified script |
| Session Replay | Included in the unified script (explicitly added at sampleRate 1) |
| Guides & Surveys | Separate `.engagement.js` file — Amplitude splits it out for payload size |

Two script tags, **one API key, one identity, one install**. That's the line to use if Josh asks how much work this is.

The loader respects the ordering rules that broke the first CDK build:

- `.engagement.js` loads **after** the unified script
- `amplitude.add(window.engagement.plugin())` runs **before** `amplitude.init()`
- `engagement.init()` / `.boot()` are never called — `add()` handles that
- Injected scripts are set to `async = false` so execution order is preserved

**The SDK status panel** at the top of the presenter rail shows three live LEDs. If one is red on the day, that panel tells you why. Most common cause by far: an ad blocker eating `amplitude.com`. Open the site in an incognito window before you panic.

---

## 3. Presenter controls (the rail on the right)

Toggle with the **Presenter controls** tab on the right edge. Four sections:

**SDK status** — three LEDs proving all three products are live off one key.

**Audience** — four personas. Clicking one calls `setUserProperties` for real, so targeting recalculates live. This is the "no extra dev work" story: the properties are already flowing for analytics, so Guides & Surveys targets on them with no new global variables.

- First-time visitor
- Saved a quote, never came back
- Inside the renewal window
- Reading help, not progressing

**Trigger locally** — fire any guide or survey on demand, regardless of persona. Use this if a moment doesn't auto-fire, or to re-show one you dismissed. "Clear all dismissals" resets everything.

**Simulated guides — on/off** — see 3a below.

**Event stream** — live feed of every event as it fires. Blue = guide, gold = survey. This is what makes the loop visible in the room: guide fires → behavior event lands → survey response lands, all in one column.

Everything is in-memory. **Reloading the page resets the whole demo** — which is what you want between run-throughs.

---

## 3a. Simulated guides — why they exist and when to switch them off

The built-in guides are hardcoded JavaScript, labeled **"Simulated locally"** or **"Amplitude · simulated"** in their top-left corner. They exist so the demo works before anything is published in Amplitude, and they stay useful as a fallback if the network misbehaves in the room.

**They are not a substitute for real guides.** A simulated guide can't show targeting evaluating against live user properties, produces no guide analytics in Amplitude, and can't be edited in front of the customer. If someone asks how long it took to build, the honest answer for a simulated one — "I wrote it in code" — is exactly the wrong answer for a product whose pitch is that you don't need engineering.

**Two ways to switch them off:**

*Permanently* — set `var SIMULATED_GUIDES = false;` at the top of `index.html` and commit.

*Live, mid-demo* — the **Simulated guides** toggle at the top of the "Trigger locally" section in the rail. No redeploy.

**What "off" changes:** nothing auto-fires as you browse, so only guides genuinely published in Amplitude appear. Everything else stays intact — the manual trigger buttons still work as a fallback, personas still set user properties, and all behavioral events still fire (real guides may be targeting on them).

**Recommended for the session:** build the real guides, switch simulated off, and leave the rail triggers as your silent safety net. If a real guide doesn't render, tap the rail and keep talking.

### Screen 1 · Overview (`#/home`)
**Persona:** Saved a quote, never came back
**What fires:** the saved-quote nudge, anchored to the "Resume a saved quote" card

> "Dana has an active jewelry policy and a quote she abandoned a week ago. Without anything contextual, she lands here and has to remember what she was doing. Instead, the guide points at exactly one thing."

Open the "Why am I seeing this?" strip at the bottom of the guide. That's the targeting rule.

**Ask Josh:** *Which customer journey would benefit most from an in-context next step?*

---

### Screen 2 · Saved quote (`#/quote`)
**What fires:** the appraisal guide, anchored to the quote card

> "Here's the friction. She stopped at the appraisal upload — and nothing on the page told her that until now. The guide gives her one action, not a paragraph of instructions."

Click **Resume quote**. The completion screen loads, and about a second later the micro-survey appears.

Answer **"Not yet"** on purpose so the follow-up text field opens.

> "Notice what we asked and when. One question, right after the moment, with options that map to an action. And we know what she *did* — the Quote Resumed event is already in the stream on the right — so we can put what she said next to what she actually did."

**Ask Josh:** *Where do customers currently pause, search, or contact the service team unnecessarily?*

---

### Screen 3 · Renewal center (`#/renewal`)
**Switch persona to:** Inside the renewal window
**What fires:** the aging-appraisal guide

> "Guidance isn't only onboarding. This is a high-value account moment — two appraisals are stale, and a claim pays against the scheduled value. The guide surfaces that at the point it matters."

Click **Review renewal information** → the 1–5 ease survey follows.

**Ask Josh:** *What should we learn from the customer immediately after this step?*

---

### Screen 4 · Help center (`#/help`)
**Switch persona to:** Reading help, not progressing
**What fires:** the stuck-in-help guide on the escalation card

Open two or three articles first — the "Did this answer your question?" survey fires after an article. Answer **"Partly"**, and the thank-you state offers a direct route to the service team.

> "When self-service works, we can measure it. When it doesn't, the survey answer becomes the escalation — and the rep already knows which guide she saw and what she said. She doesn't explain it twice."

**Ask Josh:** *Which audience should* not *see this guide?*

---

### Screen 5 · Pilot plan (`#/pilot`)
Editable fields. Fill them in live with Josh — that's the close.

Below it, the measurement panel. Every tile reads `— —` on purpose.

> "A guide being published isn't the outcome. The question is whether more people took the next step and whether they said it got easier. These stay blank until the pilot fills them."

**Ask Josh:** *Who owns the decision to revise or retire the experience?*

---

## 5. Building real guides against this page

Once your key is in and the site is live, you can point the Amplitude visual editor at it and build real guides. Every element you'd want to anchor to already carries a stable hook:

| Anchor | Where |
|---|---|
| `[data-amp-anchor="task-resume-quote"]` | Saved-quote card, home |
| `[data-amp-anchor="task-renewal"]` | Renewal card, home |
| `[data-amp-anchor="saved-quote-card"]` | Quote detail card |
| `[data-amp-anchor="resume-quote-cta"]` | Resume quote button |
| `[data-amp-anchor="quote-help-cta"]` | "I need help finding my quote" |
| `[data-amp-anchor="renewal-date-card"]` | Renewal summary |
| `[data-amp-anchor="review-renewal-cta"]` | Review renewal button |
| `[data-amp-anchor="help-search"]` | Help search block |
| `[data-amp-anchor="still-need-help"]` | Escalation card |

Events available for targeting and measurement: `Page Viewed`, `Task Card Clicked`, `Quote Resumed`, `Help Requested`, `Renewal Review Completed`, `Help Search Performed`, `Help Article Opened`, `Escalation Started`, `Guide Shown`, `Guide CTA Clicked`, `Guide Dismissed`, `Survey Shown`, `Survey Response Submitted`.

User properties: `portal_visits`, `has_saved_quote`, `quote_age_days`, `days_to_renewal`, `lifecycle_stage`, `help_page_views`, `renewal_reviewed`, `policy_status`.

**Note:** the built-in guides are labeled "Simulated locally" in their top corner. Real Amplitude guides render themselves and won't carry that label — so you can tell them apart on stage. If you build real ones, use the rail's local triggers only as a backup.

---

## 6. Deploy to GitHub Pages

```bash
mkdir wrberkley-guides-demo && cd wrberkley-guides-demo
mv ~/Downloads/index.html .
mv ~/Downloads/README.md .

python3 -m http.server 8000     # preview at localhost:8000, Ctrl+C to stop

git init
git add .
git commit -m "WR Berkley guides and surveys demo site"
```

Then either:

```bash
gh repo create wrberkley-guides-demo --public --source=. --remote=origin --push
```

or create an empty public repo at github.com/new (no README) and run the three lines it gives you.

Turn on hosting: **repo → Settings → Pages →** Source *Deploy from a branch*, Branch *main*, folder */ (root)* → **Save**. Live in about a minute at `https://<your-username>.github.io/wrberkley-guides-demo/`.

To add the key later without the terminal: open `index.html` in the repo → pencil icon → change the one line → Commit. Pages redeploys itself.

A public repo means the key is visible. That's fine for a browser key — they're write-only and designed to sit in client-side code — but **use a demo org key, never a customer production key.**

---

## 7. Guardrails baked in

- Persistent "Demo environment — fictional data" banner on every screen
- Footer on every screen: "Fictional demo for discussion — not a production WR Berkley site"
- No WR Berkley logos, production URLs, or copied customer copy
- Every metric on the measurement panel is a visible placeholder — no invented lift, deflection, or satisfaction figures
- All names, policy numbers, item values, and premiums are invented

**Before any external use**, run this past WR Berkley's branding, privacy, and governance requirements. "Lavalier" is a real WR Berkley operating unit name — the demo uses it as a plausible customer context, not as approved brand usage. If that's a concern, it's a find-and-replace on one word in the header and title.
