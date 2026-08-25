# Kurate Recommendations — UX Feedback

A password-gated, single-page review tool for walking through the Kurate
Recommendations flow screen-by-screen. Cloned from the structure of
[kurate.psyfilab.com](https://github.com/Shreenandbhattad/kurate.psyfilab.com).

## What's here

- `index.html` — the whole app. A sidebar lists every screen grouped by
  section (Onboarding, Home, Discover, Focus Mode, Post Interaction,
  View Post, Profile, Notifications). Each screen renders in a phone frame;
  clicking a numbered pin opens a "What / Why" note plus a reply thread.
- `assets/screens/` — the 55 screenshots, numbered and grouped by section.
- `Logo.jpeg` — corner/gate branding (reused from the source repo — swap it
  out if you want different branding).
- `APPSCRIPT.md` — instructions for wiring up a Google Sheet so replies sync
  across browsers instead of staying in localStorage.

## Status

- Screenshots are wired up; **annotations are currently empty** for every
  screen (`annotations:[]`). Add your "what/why" feedback per screen by
  editing the `screens` array near the top of the `<script>` block in
  `index.html`.
- The gate password is set — change it by editing the `val===` check in the
  `unlock()` function.
- `REPLIES_URL` is left blank, so comments only persist in your own browser's
  localStorage until you deploy your own Apps Script (see `APPSCRIPT.md`) and
  paste the URL in.

## Adding a feedback pin

Each screen in the `screens` array looks like:

```js
{id:"onboarding_1",page:2,title:"Onboarding 1",section:"Onboarding",
 img_src:"assets/screens/01-onboarding-1.png",annotations:[]}
```

Add pins by filling in `annotations` with objects like:

```js
{x:63,y:24,type:"content",label:"Short label",what:"What to change",why:"Why it matters"}
```

`x`/`y` are percentage positions within the phone frame; `type` is either
`"content"` (Feature/Content, purple dot) or `"ux"` (UI/UX, coral dot).
