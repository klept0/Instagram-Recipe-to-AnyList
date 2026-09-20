# Instagram Recipe → AnyList

**Current version: 1.1.0**

A userscript that scrapes recipe captions off Instagram posts and reels, splits them into title/ingredients/steps, and gets that data into [AnyList](https://www.anylist.com) — somewhere AnyList's own import extension can't reach, because Instagram carries none of the `schema.org` Recipe markup that extension looks for.

## The problem

AnyList's browser extension imports recipes by reading `schema.org` Recipe markup (JSON-LD or microdata) from the current page. That works great on food blogs. Instagram doesn't have any of that markup — the recipe only ever exists as free-text in the caption (or, sometimes, a comment) — so AnyList's extension simply has nothing to read.

## What this does

- Adds a 🍳 **Extract Recipe** button on Instagram post (`/p/...`) and reel (`/reel/...`) pages.
- Pulls the caption (DOM scrape, with a fallback to the `og:description` meta tag; if the caption itself has no recognizable recipe, it'll also try the top comment as a last resort).
- Splits it into **Title / Ingredients / Preparation Steps** using regex heuristics — no LLM calls, no network requests, nothing leaves your browser. Handles both single-block captions ("Ingredients:" ... "Instructions:") and captions that bundle several mini sub-recipes back to back (ingredients → a few "•" steps → next component's ingredients → ...).
- Also picks up **Servings, Prep/Cook time, and Nutrition** (calories/protein/carbs/fat) when the caption states them — never guessed, left blank otherwise.
- **Unit converter**: toggles ingredients/steps between metric and imperial, remembers your last choice across reels.
- Everything is editable before you use it — this is a heuristic parser, not magic, and some captions will need a touch-up.
- **Copy buttons** for Title / Ingredients / Steps, matching AnyList's own documented "Paste Ingredients" / "Paste Preparation Steps" fields — this is the reliable path on every browser.
- **Copy full recipe** as one combined text block, for pasting anywhere that isn't AnyList's two-field format.
- **Copy as schema.org JSON-LD**, if you want the structured data for your own records.
- A **"Prep this page for AnyList import"** bonus button (Chrome/Brave only — see Limitations) that embeds the recipe as schema.org markup directly into the live Instagram page, so AnyList's own extension can read it the normal way.
- Panel follows your OS light/dark mode automatically.

## Install

You need a userscript manager:

- **Chrome / Brave / Edge / Firefox**: [Tampermonkey](https://www.tampermonkey.net/)
- **Safari (macOS)**: [Userscripts](https://github.com/quoid/userscripts) by Quoid — free, open source, available on the Mac App Store. (Tampermonkey's own Safari build may also work but hasn't been tested with this script.)

You also need the [AnyList app](https://www.anylist.com) itself. This script only prepares recipe data — it doesn't talk to AnyList's servers or import anything on its own. Getting that data *into* AnyList happens one of two ways:

- **Manual paste** (works everywhere, no extra install): copy Title/Ingredients/Steps from the panel and paste into AnyList's own "Paste Ingredients" / "Paste Preparation Steps" fields.
- **One-click import** (Chrome/Brave only, see Limitations): requires [AnyList's own browser extension](https://www.anylist.com/recipes/browser-extensions) to also be installed — this script just gets the page ready for it to read.

Then either:

- **Direct install**: with Tampermonkey (or Userscripts.app) installed, just open [the raw script URL](https://raw.githubusercontent.com/klept0/Instagram-Recipe-to-AnyList/main/instagram-recipe-to-anylist.user.js) — your userscript manager should recognize it and prompt to install. Future updates will be offered automatically (Tampermonkey checks `@version` against this URL).
- **Manual install**: open `instagram-recipe-to-anylist.user.js` in this folder, create a new script in your userscript manager, and paste the whole file in (or use "Import from file"). Save/enable it.

## Usage

1. Open a recipe reel or post on `instagram.com`.
2. Click **🍳 Extract Recipe**.
3. Review/edit the Title, Servings/Prep/Cook, Nutrition, Ingredients, and Steps fields.
4. In AnyList: create a new recipe, tap **Paste Ingredients** and paste, then tap into Preparation Steps and paste those.
5. On Chrome/Brave, you can instead try **Prep this page for AnyList import** → click the real AnyList toolbar icon on the same tab.

## Known limitations

- **Heuristic parsing.** Captions are free-form text written by humans for humans, not structured data — the regex-based parser handles common formats well but won't be perfect on everything. Always review the extracted fields before using them. If a caption's structure isn't parsing right, the **raw caption** box (with a **Re-parse** button) lets you hand-edit and re-run the parser.
- **Safari's one-click AnyList import doesn't work**, and this isn't fixable from the userscript side. AnyList's Chrome extension captures the live page HTML and hands it to their import backend directly. Their Safari extension instead just forwards the page's *URL* and lets AnyList's server fetch it independently — and since Instagram requires a login to render anything, that server-side fetch just gets an empty shell. On Safari, use the Copy buttons and paste manually; that path works identically everywhere.
- **Comment fallback is best-effort.** The DOM selectors used to find a top comment haven't been verified against a live page the way the caption-scraping path has. If you hit a caption with no recipe text and it doesn't find the comment either, that's why.
- Instagram frequently changes its DOM. If extraction stops working, the caption-scraping selectors in `extractFromDom()` are the first place to look.

## Privacy

Everything runs locally in your browser. No API keys, no external requests, no analytics. The only "network" activity is you manually pasting into AnyList, or (Chrome/Brave only) the AnyList extension itself fetching data when you click its icon — same as it does on any other site.

## Changelog

**1.1.0**
- Servings, Prep/Cook time, and Nutrition extraction (never guessed — blank if the caption doesn't state them).
- Metric ⇄ imperial unit converter, with your last choice remembered across reels.
- Multi-block parsing: captions that bundle several mini sub-recipes (ingredients → a few "•" steps → next component...) with no top-level "Instructions" header now split correctly.
- Emoji stripped from imported Title/Ingredients/Steps text.
- Light/dark mode follows the OS automatically.
- Panel auto-grows to fit content instead of scrolling internally; width capped so it can't overflow narrow windows.
- **Clear** button — wipes all fields and removes any AnyList markup injected into the page by a previous import attempt.
- **Copy as schema.org JSON-LD** and **Copy full recipe** (one combined text block) buttons.
- **Prep this page for AnyList import** — embeds schema.org markup into the live page for AnyList's extension to read (Chrome/Brave only; see Limitations for why Safari can't do this).
- **Re-parse** button on the raw-caption box, for re-running the parser after hand-editing.
- Comment fallback: if the caption itself has no recognizable recipe, tries the top comment as a last resort (best-effort, unverified DOM path).
- Works on Safari via Userscripts.app: `@inject-into content` avoids Instagram's CSP blocking inline script injection, and a local `GM_addStyle` polyfill covers managers that don't implement it.
- Debug logging gated behind a flag instead of always-on console noise.

**0.1.0 - 1.0.0**
- Initial release: caption extraction, title/ingredients/steps parsing, copy-to-clipboard panel.

## Disclaimer

This is an independent, unofficial project — it's not affiliated with, endorsed by, or supported by AnyList or Purple Cover, Inc. "AnyList" is their name and trademark, used here only to describe what this script is compatible with, and I claim no rights to it.

This script works by observing how AnyList's apps and browser extensions currently behave, not by using any published or supported API — that behavior is theirs to change at any time, without notice, and without any obligation to keep working with this script. If AnyList changes their import process, the AnyList-related features here (the one-click import button, and possibly the manual paste format) could break. The manual Copy-buttons workflow is the most resilient path since it just produces plain text for you to paste yourself.

## License

MIT — do whatever you want with it.
