# Windows 98 theme for Discord

A Windows 98 look for the Discord desktop client, for **Vencord** and
**BetterDiscord**. Built for the 2025+ Discord client: it changes Discord's own
CSS variables and only targets classes by partial match, so small Discord
updates shouldn't break it.

- Teal desktop, silver windows with classic 3D bevels
- Navy gradient title bars with white text
- Gray raised buttons, white sunken text fields
- Square corners everywhere, pixel "MS Sans Serif" font
- Chunky Win98 scrollbars with arrow buttons

## Install

### Vencord

**Online (updates automatically):**
1. Settings → **Themes** → **Online Themes**.
2. Paste this link and save:
   ```
   https://raw.githubusercontent.com/jerzky/WinTheme/main/win98.theme.css
   ```

**Local file:**
1. Download [`win98.theme.css`](win98.theme.css).
2. Settings → **Themes** → **Open Themes Folder** (`%appdata%\Vencord\themes`,
   or `%appdata%\vesktop\themes` for Vesktop) and put the file there.
3. Enable **Windows 98** in the theme list.

### BetterDiscord

1. Download [`win98.theme.css`](win98.theme.css). The name has to end in `.theme.css`.
2. Settings → **Themes** → **Open Themes Folder** (`%appdata%\BetterDiscord\themes`)
   and put the file there.
3. Turn on **Windows 98**.

## Customizing

All colors, bevels and the font are variables at the top of the file
(`--w98-desktop`, `--w98-face`, `--w98-title-start`, …). Change them in one place.

- **Square avatars and server icons:** uncomment section 10 at the end of the file.
- **Scrollbar gap:** `--w98-scroll-inset` (default `5px`).

## Something broke after a Discord update?

[THEMING-NOTES.md](THEMING-NOTES.md) has the DevTools snippets for finding Discord's
current variable and class names, the names confirmed so far, and the gotchas we
hit. Issues and pull requests are welcome.

## Credits

Bevels and the pixel font come from [98.css](https://github.com/jdan/98.css)
by Jordan Scales (MIT).
