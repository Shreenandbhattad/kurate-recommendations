# Firebase Firestore — Shared Replies Setup

Replaces the old Google Apps Script approach. No code to write, no
"redeploy as a new deployment" dance — and unlike Apps Script, **you only
ever do this once.** Every future clone of this template reuses the exact
same Firebase project, config, and security rule — the only thing that
changes per clone is one string (`REPLIES_SITE`) in `index.html`.

## One-time setup (~5 min)

1. Go to **console.firebase.google.com** → **Add project** → name it
   something generic you'll reuse forever, e.g. **"kurate-reviews"**.
   (Google Analytics prompt: skip it, not needed.)
2. In the left sidebar: **Build → Firestore Database → Create database**.
   Pick any region close to you. Start in **production mode** (we'll paste
   a scoped rule below, so this is safe).
3. **Rules** tab → replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       // Open read/write, but scoped to exactly one collection — the same
       // one every clone of this template uses. Comments are low-stakes
       // (same trust model as the old Apps Script "Anyone" deployment).
       match /kurate_reviews/{doc} {
         allow read, write: if true;
       }
     }
   }
   ```

   Click **Publish**.
4. **Project settings** (gear icon, top left) → scroll to **Your apps** →
   click the **</>** (Web) icon → register an app (any nickname) → it shows
   you a `firebaseConfig` object. Copy it.
5. In `index.html`, find `var FIREBASE_CONFIG = { ... }` and paste your
   values in (apiKey, authDomain, projectId, storageBucket,
   messagingSenderId, appId).
6. Done. Comments now sync live across everyone viewing the page — no
   polling, no manual refresh.

## Reusing this for your *next* clone repo

Don't repeat any of the steps above. Just:
1. Copy the same `FIREBASE_CONFIG` object into the new repo's `index.html`.
2. Change `REPLIES_SITE` to something unique, e.g. `'kurate-onboarding-v2'`.

That's it — same project, same collection, same rule, forever. Replies are
scoped by a compound `skey` field (`site::key`) so different clone-sites
never see each other's comments even though they share one collection.

## How it works

- `addReply` / `deleteReply` / `saveEdit` call Firestore directly from the
  browser (`db.collection(...).add/.doc(id).delete/.update`).
- `watchReplies(si, ni)` attaches a live `onSnapshot` listener scoped to the
  currently-open pin's thread when a popup opens, and `stopWatchingReplies()`
  detaches it when the popup closes — so only one listener is ever active,
  and every viewer sees new replies appear instantly without refreshing.
- If `FIREBASE_CONFIG.apiKey` is left blank, the page automatically falls
  back to localStorage-only replies (per-browser, not shared) — same
  fallback behavior as before, so the page never breaks pre-setup.

## Troubleshooting

**Replies aren't saving / console shows a permissions error?**
- Check the Firestore **Rules** tab was actually published (step 3) and
  matches the collection name in `REPLIES_COLLECTION` (`kurate_reviews` by
  default — keep these in sync if you rename either one).

**Nothing happens, no errors?**
- Open devtools console — if you see "Firebase init failed", double-check
  the pasted `FIREBASE_CONFIG` values (a stray comma or missing quote is
  the usual culprit).

**Want to see all replies across every clone-site in one place?**
- Firestore console → `kurate_reviews` collection → each doc has a `site`
  field showing which clone it came from.
