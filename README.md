# RP Clone — Workout Engine

A self-hosted, single-file hypertrophy training app. Implements RP-style autoregulation locally: per-muscle volume landmarks (MV/MEV/MAV/MRV), an RIR ladder with per-exercise floors, double-progression load/rep rules, set autoregulation from pre/post-session feedback, deload triggers, projected progression analytics, phase-aware engine biases (cut/maintenance/surplus), exercise swap with custom-add, multi-user profiles, and an onboarding wizard for new users.

No server, no internet calls, no accounts. Everything lives in your browser's `localStorage`. JSON export/import for backup or device-to-device transfer.

## Try it

Open `index.html` in any modern browser. First launch drops you into the onboarding wizard. Pick exercises, set base sets per exercise, define your weekly session shape, and tap "Start block."

## Hosting on GitHub Pages

1. Create a new repository on github.com (public or private — private requires a paid GitHub plan for Pages).
2. Push this folder (see the steps under "Pushing to GitHub" below).
3. In the repo on GitHub: **Settings → Pages → Source: Deploy from a branch → Branch: `main`, folder: `/ (root)` → Save**.
4. After ~30 seconds, GitHub gives you a URL like `https://<your-username>.github.io/<repo-name>/`. Bookmark it.

## Installing as an app on iOS

1. Open the GitHub Pages URL in Safari on your iPhone.
2. Tap the Share button → **Add to Home Screen**.
3. A home-screen icon appears. Tapping it opens the app fullscreen — no Safari chrome, no URL bar.

The app already includes iOS PWA meta tags and a 180×180 icon (inline SVG, no external assets needed).

## State storage and multi-user

State lives in browser `localStorage`, scoped per user. Each profile gets its own entry under `rp-clone-user-<id>`. The user picker in the top nav (`👤 <name> ▾`) lets you switch between profiles or add new ones. Different users on the same physical device get fully independent state; the same URL on a friend's device is theirs alone, never yours.

To move your data between devices: Settings → **Export JSON**, save the file, **Import** on the other device.

## Documentation

- `docs/ALGORITHM.md` — the engine design: volume landmarks, RIR ladder, autoregulation matrix, deload triggers, between-block learning, set counting rules, floating sets / catch-up sessions, phase biases.
- `docs/DIAGNOSTIC.md` — a worked example diagnostic on a real routine, used to validate the engine on real input before any UI was written. Useful as a reference for what the diagnostic + first-block proposal looks like.

## Pushing to GitHub

After creating an empty repository on github.com (don't initialize it with a README or `.gitignore` — let your push populate it), run from this folder:

```bash
git init -b main
git add .
git commit -m "Initial commit: RP Clone single-file workout engine"
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

Subsequent updates: edit files locally, then:

```bash
git add .
git commit -m "Describe the change"
git push
```

GitHub Pages re-serves automatically after each push (usually within a minute).

## File layout

```
.
├── index.html          # the app (open in browser; also hosted as GitHub Pages root)
├── app.html            # identical to index.html; kept for local-file shortcuts that already point here
├── README.md
├── .gitignore
└── docs/
    ├── ALGORITHM.md    # engine design spec
    └── DIAGNOSTIC.md   # worked example diagnostic
```

## Customization

The exercise library, volume landmarks, weekly targets, and block template are all editable from inside the app (Settings + onboarding). For deeper changes — adding new feedback inputs, changing the autoregulation matrix, etc. — edit the `<script>` block inside `index.html` directly. Helpful entry points:

| Concept | Function / constant |
|---|---|
| Volume landmarks | `DEFAULT_LANDMARKS` |
| Exercise library | `EXERCISES` |
| Block template (example) | `BLOCK_1_TEMPLATE` |
| RIR ladder | `RIR_LADDER`, `RIR_FLOOR` |
| Set autoregulation rules | `autoregDelta()` |
| Load + rep progression | `nextRx()` |
| Deload triggers | `checkDeload()` |
| Catch-up evaluation | `pendingFloating()` |
| Projection engine | `projectExercise()`, `sparklineSVG()` |

## Acknowledgments

The training principles encoded here are publicly documented by Renaissance Periodization (Dr. Mike Israetel and team). This project is an independent reimplementation of the underlying ideas for personal use — it does not redistribute any RP-proprietary content, app code, or data. For the original RP Hypertrophy app, training plans, and educational content, see [rpstrength.com](https://rpstrength.com).

## License

Personal-use project; no formal license. If you fork it for your own training, that's the point.
