# iConnectivity User Guide: Setup

Squarespace can't host loose markdown files, so the guides are stored in the public GitHub repository [iConnectivity/user-guides](https://github.com/iConnectivity/user-guides) and shown on a Squarespace page by a Code Block. When you edit a `.md` file on GitHub, the website picks up the change (see "Changes not showing?" below). You don't need to touch Squarespace again.

## What's in this folder

| File | What it is |
| --- | --- |
| `squarespace-code-block.html` | Paste this into a Code Block on your Squarespace page |
| `guides/guides.json` | The list of guides and the order of the Guide menu |
| `guides/<product>/` | One folder per guide (for example `playaudio2d/`), with the `.md` file and its `images` folder |
| `HOW-TO-WRITE-A-GUIDE.md` | Notes for whoever edits or adds guides (not shown on the website) |

## 1. Where the guides live

The guides are in the `guides` folder of the public repository https://github.com/iConnectivity/user-guides, on the `main` branch. The website reads them through jsDelivr at:

```
https://cdn.jsdelivr.net/gh/iConnectivity/user-guides@main/guides/
```

The repository must stay **public**; jsDelivr can't read private repositories.

## 2. Create the page in Squarespace

1. In Squarespace, go to **Pages**, click **+**, choose **Blank Page**, and name it **User Guide**. The URL slug becomes `/user-guide`.
2. Edit the page, add a block, and choose **Code**.
3. Open `squarespace-code-block.html` in a text editor (TextEdit in plain-text mode, or Notepad), copy all of it, and paste it into the Code Block.
4. Check that the `baseUrl` line near the top reads `https://cdn.jsdelivr.net/gh/iConnectivity/user-guides@main/guides/`.
5. Make sure **Display Source** is off, then save and publish.

> Squarespace only runs JavaScript in Code Blocks on its business-level plans, not the Personal plan. If the page shows the code instead of the guide, check your plan.

### How it matches iconnectivity.com

The guide uses the styles from your site's own stylesheet:

| | Your site | Used in the guide for |
| --- | --- | --- |
| Font | Futura PT (from your Adobe Fonts kit) | All text |
| Paragraph text | `#464444`, 18px | Guide text |
| Headings | `#111`, regular weight | Guide headings |
| Links | Cyan `#00aeef` with a fine underline (used on your pages with background images) | Links and the highlighted contents entry |
| Buttons | Charcoal `#272727`, bold uppercase | Previous / Next buttons |
| Dividers and quotes | `#ddd` lines | Section dividers, notes, tables |

Your template normally makes a single block half the page width. The guide widens its own Code Block to the full content width (up to 1240px), so you don't need to add Custom CSS for this page.

### Fine-tuning (optional)

These settings are at the top of the pasted code:

- **`accent`**: the link and highlight color. It's set to your cyan, `"#00aeef"`. Use `"#d39850"` for the gold your other pages use for links.
- **`topOffset`**: leave as `"auto"`. The guide measures your header, including the fixed top bar on phones. If headings end up hidden behind a header after you click a contents link, set a number of pixels instead.
- **`tocLevels`**: `[2, 3]` lists `##` and `###` headings in the contents. Use `[2]` for a shorter list.

Other colors and sizes are listed at the top of the `<style>` section (for example `--icx-text` and `--icx-heading`) if you ever want to change them.

## 3. Editing guides from now on

- **Edit a guide:** on GitHub, open the `.md` file, click the pencil icon, make changes, and click **Commit changes**.
- **Add a guide:** see the included "How to Write a Guide" page.
- **Changes not showing?** jsDelivr caches files for up to 12 hours. To clear the cache right away, visit `https://purge.jsdelivr.net/gh/iConnectivity/user-guides@main/guides/FILE.md`, replacing `FILE.md` with the path of the file you changed inside `guides` (for example `playaudio2d/playaudio2d-user-guide-2026.md`) (and once for `guides.json` if you edited it).

## Links people can share

- `iconnectivity.com/user-guides#playaudio2d` opens a specific guide.
- `iconnectivity.com/user-guides#playaudio2d/updating-playaudio2d-firmware` opens a specific section.
