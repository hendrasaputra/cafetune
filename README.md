# Cafetune

A café that never closes, playing music that never repeats. Live at **https://cafetune.hensap.id**

Nothing on the page is a recording. A five-piece chiptune band and a room full of talking
people are both synthesised in the browser, one bar at a time, using the Web Audio API.
No libraries, no audio files, no network calls after the page loads.

Part of [Pixelized Thoughts](https://hendrasaputra.com/).

---

## Publish it

Do these six things in order. It takes about ten minutes, plus waiting for DNS.

### 1. Create the repository

On GitHub, create a new **public** repository called `cafetune`. Do not add a README,
licence or .gitignore — this folder already has what it needs.

> GitHub Pages only serves private repositories on paid plans. Make it public unless you pay
> for GitHub Pro, Team or Enterprise.

### 2. Push this folder

From inside this folder, run:

```bash
git init
git add .
git commit -m "Cafetune: generative chiptune cafe"
git branch -M main
git remote add origin https://github.com/YOUR-GITHUB-USERNAME/cafetune.git
git push -u origin main
```

Replace `YOUR-GITHUB-USERNAME` with your actual GitHub username.

### 3. Turn on GitHub Pages

In the repository, go to **Settings → Pages**. Under "Build and deployment", set
**Source** to `Deploy from a branch`, then set the branch to `main` and the folder to
`/ (root)`. Press Save.

Wait two minutes, then check that `https://YOUR-GITHUB-USERNAME.github.io/cafetune/` loads.
Get this working before you touch DNS — it isolates any problem to one place.

> The file `.github/workflows/deploy.yml` is an alternative to this step, not an addition.
> If you would rather deploy with GitHub Actions, set **Source** to `GitHub Actions` instead.
> If you deploy from a branch as above, delete that workflow file so it doesn't run pointlessly.

### 4. Point the subdomain at GitHub

Go to whoever runs DNS for `hensap.id` and add one record:

| Field | Value |
| --- | --- |
| Type | `CNAME` |
| Name / Host | `cafetune` |
| Value / Target | `YOUR-GITHUB-USERNAME.github.io` |
| TTL | leave at the default |

Two things people get wrong here:

- The target ends in `.github.io` with **no** repository name and **no** `https://`.
- If `hensap.id` sits behind Cloudflare, set the record to **DNS only** (grey cloud, not
  orange). GitHub cannot issue the HTTPS certificate through Cloudflare's proxy. You can
  switch the proxy on later once the certificate exists.

### 5. Tell GitHub about the domain

Back in **Settings → Pages**, put `cafetune.hensap.id` in the **Custom domain** box and press
Save. The `CNAME` file in this repository already contains that name, so GitHub should pick
it up on its own — but set it in the interface anyway, because that is what triggers the
certificate request.

GitHub now runs a DNS check. It can fail for the first hour while the record spreads. That is
normal. Come back later and press **Check again**.

### 6. Force HTTPS

Once the DNS check passes and the certificate is issued, tick **Enforce HTTPS** on the same
screen. If the tickbox is greyed out, the certificate isn't ready yet — wait and come back.

---

## Check it worked

- `https://cafetune.hensap.id` loads and the title glows amber.
- Pressing **Open the cafe** produces sound within a second.
- The `--mode=` button in the top right switches between light and dark, and the choice
  survives a page reload.
- Paste the URL into WhatsApp or LinkedIn and the share card shows `og.png`.

If the page loads but stays silent, open the browser console. Browsers block audio until you
interact with the page, which is why the sound starts on the button press rather than on load.

---

## What's in here

| File | What it does |
| --- | --- |
| `index.html` | The whole thing. Page, styling and audio engine in one file. |
| `CNAME` | Tells GitHub Pages the site answers to `cafetune.hensap.id`. Don't rename it. |
| `.nojekyll` | Stops GitHub running the files through Jekyll, which it does by default. |
| `og.png` | The 1200×630 image that shows when the link is shared. |
| `favicon.svg` | Pixel coffee cup, drawn in the site's amber and coral. |
| `robots.txt`, `sitemap.xml` | Lets search engines index the page. |
| `.github/workflows/deploy.yml` | Optional. Only used if you deploy via GitHub Actions. |
| `LICENSE` | MIT for the code, with the brand and copy carved out. |

## Changing it

Everything lives in `index.html`. The colours at the top of the `<style>` block are copied
from hendrasaputra.com, so if you restyle the parent site, update the two `:root` blocks here
to match and the subdomain will follow.

To change how the music behaves, look for the numbered comment blocks in the script:
harmony is section 3, the band is section 4, the room is section 5.

---

## Licence

The code is MIT licensed. Read `LICENSE` for the full text. In short: take it,
change it, ship it, sell it — just keep the copyright notice in the source.

The licence covers code and nothing else. These stay © 2026 hendrasaputra.com,
all rights reserved:

- the names "Pixelized Thoughts", "Cafetune", "hensap" and "Hendra Saputra"
- the `hensap@pixelized:~$` mark and the visual identity around it
- `og.png` and `favicon.svg`
- the written copy on the page and in this README

So you are welcome to fork the audio engine and the layout. Give the result
your own name and your own artwork.

The four typefaces load from Google Fonts at runtime and are never copied into
this repository, so nothing here redistributes them. They are IBM Plex Mono,
VT323, Silkscreen and Newsreader, all under the SIL Open Font License.

© 2026 [hendrasaputra.com](https://hendrasaputra.com/)
