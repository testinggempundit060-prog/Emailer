# GemPundit Emailer Builder

A small internal tool that turns your `/gempundit-emailer-builder-master-production-prompt`
skill into a web UI:

1. Upload the designer's zip of sliced GIF/PNG assets, plus the full reference PNG.
2. Edit **hover text (alt/title)**, **link URL**, **image URL (CDN)**, background color,
   and width per tile.
3. Set the campaign name and the **WhatsApp footer CTA message**.
4. Click **Generate emailer HTML** — the app calls the Anthropic API with the exact rules
   from your skill (canonical header/footer, mobile-safety system, alt/title conventions,
   UTM pattern, `target="_self"` everywhere) and returns send-ready HTML.
5. Preview it, request fixes in plain English, copy or download the final `.html`.

The skill's rules live in `lib/skillPrompt.js`. If you update your skill file later,
mirror the change there so the app and your Claude skill stay in sync.

## 1. Run it locally

```bash
npm install
cp .env.example .env.local   # then paste your key into .env.local
npm run dev
```

Open http://localhost:3000

You need an Anthropic API key from https://console.anthropic.com/settings/keys.
The key is only ever used server-side (in `app/api/generate/route.js`) — it is never
sent to the browser.

## 2. Deploy to Vercel

**Option A — Vercel CLI (fastest)**

```bash
npm i -g vercel
vercel login
vercel            # first deploy, follow the prompts
vercel env add ANTHROPIC_API_KEY   # paste your key when prompted, choose Production + Preview
vercel --prod
```

**Option B — GitHub + Vercel dashboard**

```bash
git init
git add .
git commit -m "GemPundit emailer builder"
git branch -M main
git remote add origin <your-empty-github-repo-url>
git push -u origin main
```

Then in https://vercel.com/new:
1. Import the GitHub repo.
2. Framework preset: Next.js (auto-detected) — no build settings to change.
3. Under **Environment Variables**, add `ANTHROPIC_API_KEY` with your key.
4. Deploy.

Every future `git push` to `main` redeploys automatically.

## Notes

- Model used: `claude-sonnet-5`. Change it in `app/api/generate/route.js` if you want
  a different one.
- The zip is parsed entirely in the browser (via `jszip`) — nothing is uploaded to any
  server except the final generation request to your own `/api/generate` route.
- Background-color swatches are auto-estimated per tile from each image's edge pixels
  (canvas sampling) as a starting point — always eyeball them against the reference
  image before generating, since pixel sampling can be a shade off from the design
  system's authoritative hex (e.g. product tile `#843736` vs a sampled `#853737`).
- The footer, header, mobile-safety CSS, and `target="_self"` rule are all enforced by
  the system prompt regardless of what's in the reference image — matching your skill's
  "do not rebuild the footer/header from the reference image" rule.
