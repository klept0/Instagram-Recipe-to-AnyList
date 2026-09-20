# Instagram Recipe → AnyList

**Version 1.1.0** · [MIT License](#license)

A userscript that pulls recipe captions off Instagram posts and reels, splits them into title/ingredients/steps, and gets that into [AnyList](https://www.anylist.com) — somewhere AnyList's own import extension can't reach, because Instagram carries none of the `schema.org` markup that extension looks for.

## Contents

- [Quick start](#quick-start)
- [Why this exists](#why-this-exists)
- [Features](#features)
- [Install](#install)
- [Usage](#usage)
- [Platform support](#platform-support)
- [Known limitations](#known-limitations)
- [Privacy](#privacy)
- [Disclaimer](#disclaimer)
- [Changelog](#changelog)
- [License](#license)

## Quick start

1. Install a userscript manager — [Tampermonkey](https://www.tampermonkey.net/) (Chrome/Brave/Edge/Firefox) or [Userscripts](https://github.com/quoid/userscripts) (Safari).
2. Open the [raw script URL](https://raw.githubusercontent.com/klept0/Instagram-Recipe-to-AnyList/main/instagram-recipe-to-anylist.user.js) — your manager should offer to install it.
3. Open a recipe reel or post on `instagram.com` and click **🍳 Extract Recipe**.
4. Review the fields, then either copy Title/Ingredients/Steps into AnyList's paste fields, or (Chrome/Brave) click **Prep this page for AnyList import** and use AnyList's own toolbar icon.

Details on each step are below.

## Why this exists

AnyList's browser extension imports recipes by reading `schema.org` Recipe markup (JSON-LD or microdata) from the current page. That works great on food blogs. Instagram carries none of it — the recipe only ever exists as free text in the caption, or sometimes a comment — so AnyList's extension has nothing to read.

This script fills that gap: it scrapes the caption itself and gets you the same result AnyList would give you on any other site.

## Features

**Extraction & parsing**
- 🍳 **Extract Recipe** button appears automatically on Instagram post (`/p/...`) and reel (`/reel/...`) pages.
- Pulls the caption via DOM scrape, falling back to the `og:description` meta tag, and finally to the top comment if the caption itself has no recognizable recipe.
- Splits it into **Title / Ingredients / Preparation Steps** with regex heuristics — no LLM calls, no network requests, nothing leaves your browser. Handles both single-block captions (`Ingredients:` … `Instructions:`) and captions that bundle several mini sub-recipes back to back (ingredients → a few "•" steps → next component's ingredients → …).
- Picks up **Servings, Prep/Cook time, and Nutrition** (calories/protein/carbs/fat) when the caption states them — never guessed, left blank otherwise.
- Strips decorative emoji from the imported text so it reads cleanly once it's in AnyList.
- Everything is editable before you use it. This is a heuristic parser, not magic — some captions will need a touch-up. A **raw caption** box plus a **Re-parse** button let you hand-edit and re-run the parser without starting over.

**Getting it into AnyList**
- **Copy buttons** for Title / Ingredients / Steps, matching AnyList's own documented "Paste Ingredients" / "Paste Preparation Steps" fields — the reliable path on every browser.
- **Copy full recipe** as one combined text block, for pasting anywhere that isn't AnyList's two-field format.
- **Copy as schema.org JSON-LD**, if you want the structured data for your own records.
- **Prep this page for AnyList import** (Chrome/Brave only — see [Platform support](#platform-support)): embeds the recipe as schema.org markup directly into the live Instagram page, so AnyList's own extension can read it the normal way.

**Quality of life**
- **Unit converter**: toggles ingredients/steps between metric and imperial, and remembers your last choice across reels.
- **Clear** button wipes all fields and removes any AnyList markup injected by a previous import attempt.
- Panel follows your OS light/dark mode automatically, and auto-grows to fit content instead of scrolling internally.

## Install

You need a userscript manager:

| Browser | Manager |
|---|---|
| Chrome / Brave / Edge / Firefox | [Tampermonkey](https://www.tampermonkey.net/) |
| Safari (macOS) | [Userscripts](https://github.com/quoid/userscripts) by Quoid — free, open source, Mac App Store. (Tampermonkey's own Safari build may also work but hasn't been tested with this script.) |

Then install the script itself, either way:

- **Direct install** *(recommended)*: open [the raw script URL](https://raw.githubusercontent.com/klept0/Instagram-Recipe-to-AnyList/main/instagram-recipe-to-anylist.user.js) — your userscript manager should recognize it and prompt to install. Future updates are offered automatically (your manager checks `@version` against this URL).
- **Manual install**: open `instagram-recipe-to-anylist.user.js` in this repo, create a new script in your userscript manager, and paste the whole file in (or use "Import from file"). Save/enable it.

You'll also want the [AnyList app](https://www.anylist.com) itself — this script only *prepares* recipe data, it doesn't talk to AnyList's servers or import anything on its own:

| Path | What you need |
|---|---|
| Manual paste (works everywhere) | Just the AnyList app |
| One-click import (Chrome/Brave only) | AnyList app **+** [AnyList's browser extension](https://www.anylist.com/recipes/browser-extensions) |

## Usage

1. Open a recipe reel or post on `instagram.com`.
2. Click **🍳 Extract Recipe**.
3. Review/edit Title, Servings/Prep/Cook, Nutrition, Ingredients, and Steps.
4. In AnyList: create a new recipe, tap **Paste Ingredients** and paste, then tap into Preparation Steps and paste those.
5. On Chrome/Brave, you can instead click **Prep this page for AnyList import**, then click the real AnyList toolbar icon on the same tab.

## Platform support

| | Chrome / Brave | Safari |
|---|---|---|
| Extraction, parsing, unit conversion, all Copy buttons | ✅ | ✅ |
| One-click "Prep this page for AnyList import" | ✅ | ❌ (see below) |

Safari's one-click import doesn't work, and it isn't fixable from the userscript side. AnyList's Chrome extension captures the live page HTML and hands it to their import backend directly. Their Safari extension instead just forwards the page's *URL* and lets AnyList's server fetch it independently — and since Instagram requires a login to render anything, that server-side fetch just gets an empty shell. On Safari, use the Copy buttons and paste manually; that path works identically everywhere.

## Known limitations

- **Heuristic parsing.** Captions are free-form text written by humans for humans, not structured data. The parser handles common formats well but won't be perfect on everything — always review the extracted fields before using them.
- **Comment fallback is best-effort.** The DOM selectors used to find a top comment haven't been verified against a live page the way the caption-scraping path has. If a caption has no recipe text and the comment fallback doesn't find it either, that's why.
- **Instagram changes its DOM often.** If extraction stops working, the caption-scraping selectors in `extractFromDom()` are the first place to look.

## Privacy

Everything runs locally in your browser. No API keys, no external requests, no analytics. The only "network" activity is you manually pasting into AnyList, or (Chrome/Brave only) the AnyList extension itself fetching data when you click its icon — same as it does on any other site.

## Disclaimer

This is an independent, unofficial project — not affiliated with, endorsed by, or supported by AnyList or Purple Cover, Inc. "AnyList" is their name and trademark, used here only to describe what this script is compatible with; no rights to it are claimed.

This script works by observing how AnyList's apps and browser extensions currently behave, not by using any published or supported API. That behavior is theirs to change at any time, without notice, and without any obligation to keep working with this script. If AnyList changes their import process, the AnyList-related features here (the one-click import button, and possibly the manual paste format) could break. The manual Copy-buttons workflow is the most resilient path, since it just produces plain text for you to paste yourself.

## Changelog

<details>
<summary><strong>1.1.0</strong></summary>

- Servings, Prep/Cook time, and Nutrition extraction (never guessed — blank if the caption doesn't state them).
- Metric ⇄ imperial unit converter, with your last choice remembered across reels.
- Multi-block parsing: captions that bundle several mini sub-recipes (ingredients → a few "•" steps → next component…) with no top-level "Instructions" header now split correctly.
- Emoji stripped from imported Title/Ingredients/Steps text.
- Light/dark mode follows the OS automatically.
- Panel auto-grows to fit content instead of scrolling internally; width capped so it can't overflow narrow windows.
- **Clear** button — wipes all fields and removes any AnyList markup injected into the page by a previous import attempt.
- **Copy as schema.org JSON-LD** and **Copy full recipe** (one combined text block) buttons.
- **Prep this page for AnyList import** — embeds schema.org markup into the live page for AnyList's extension to read (Chrome/Brave only; see Platform support for why Safari can't do this).
- **Re-parse** button on the raw-caption box, for re-running the parser after hand-editing.
- Comment fallback: if the caption itself has no recognizable recipe, tries the top comment as a last resort (best-effort, unverified DOM path).
- Works on Safari via Userscripts.app: `@inject-into content` avoids Instagram's CSP blocking inline script injection, and a local `GM_addStyle` polyfill covers managers that don't implement it.
- Debug logging gated behind a flag instead of always-on console noise.

</details>

<details>
<summary><strong>0.1.0 – 1.0.0</strong></summary>

- Initial release: caption extraction, title/ingredients/steps parsing, copy-to-clipboard panel.

</details>

## License

MIT — do whatever you want with it. Full text in [`LICENSE`](./LICENSE).
