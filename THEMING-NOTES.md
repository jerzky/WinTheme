# Discord theming notes

What we learned building `win98.theme.css` against the Discord desktop client
(Vencord, Windows, post-2025 "visual refresh"). Everything here was checked on a
live client on **2026-10-25**. Discord renames things, so treat names as
"last known good" and re-check with the snippets below when something breaks.

---

## 1. Workflow

1. Open DevTools with **Ctrl+Shift+I**. Vencord and BetterDiscord enable it.
   Stock Discord needs `"DANGEROUS_ENABLE_DEVTOOLS_ONLY_ENABLE_IF_YOU_KNOW_WHAT_YOURE_DOING": true`
   in `%appdata%\discord\settings.json`.
2. Use the **Console** snippets below rather than the Styles pane filter.
   Filtering the `<html>` element's styles freezes DevTools, because it carries
   hundreds of variables.
3. `copy(...)` puts the result on the clipboard and prints `undefined`. That's
   normal. Each `copy` overwrites the previous one, so paste after each run.
   The first paste may require typing `allow pasting`.
4. Run class or layout snippets **on the screen you're styling**. The Friends
   page has no chat, member list or message box, so their classes won't show up.
5. Test CSS changes outside Discord with a mock page (section 7).

---

## 2. Console snippets

**All Discord variables of a given kind** (edit the regex):
```js
copy([...new Set([...document.styleSheets].flatMap(s=>{try{return [...s.cssRules]}catch{return []}}).flatMap(r=>r.style?[...r.style].filter(p=>p.startsWith('--')):[]))].filter(v=>/background|bg-|text|header|interactive|icon|channel|input|border|radius|scrollbar|font|chat|modal/.test(v)).sort().join('\n'))
```

**Class names on screen that start with a word** (edit the regex):
```js
copy([...new Set([...document.querySelectorAll('[class]')].flatMap(e=>[...e.classList]))].filter(c=>/^(chat|members|channelTextArea|scrollableContainer|menu|tooltip|modeSelected)/.test(c)).sort().join('\n'))
```

**Which value a variable has at each level, from an element up to `<html>`.**
Use this when a variable "doesn't apply": the level where the value changes is
where it gets redefined.
```js
let e=document.querySelector('[class*="chatContent_"]'),out=[];while(e){const s=getComputedStyle(e);out.push((typeof e.className=='string'?e.className:e.tagName).slice(0,80)+' | lower='+s.getPropertyValue('--background-base-lower').trim()+' | bg='+s.backgroundColor);e=e.parentElement}copy(out.join('\n'))
```

**Every CSS rule that sets a property**, with stylesheet, layer and `!important`.
Note: theme files loaded via `<link>` may be unreadable here, so a theme rule
missing from this list doesn't prove it wasn't loaded.
```js
const P='--background-base-lower',out=[];const walk=(rules,layer,src)=>{for(const r of rules){if(r.style&&r.style.getPropertyValue(P))out.push([src,layer||'(no layer)',r.selectorText,r.style.getPropertyPriority(P)||'normal',r.style.getPropertyValue(P).trim().slice(0,50)].join(' | '));if(r.cssRules)walk(r.cssRules,r instanceof CSSLayerBlockRule?layer+'/'+(r.name||'anon'):layer,src)}};for(const s of [...document.styleSheets,...document.adoptedStyleSheets]){let rs;try{rs=s.cssRules}catch{continue}walk(rs,'',String(s.href||s.ownerNode?.id||'inline').split('/').pop())}copy(out.join('\n'))
```

**Every scrolling area on screen**:
```js
copy([...document.querySelectorAll('*')].filter(e=>{const s=getComputedStyle(e);return /(auto|scroll)/.test(s.overflowY)&&e.clientHeight>50}).map(e=>{const s=getComputedStyle(e);return [String(e.className).slice(0,70),s.overflowY,'sw='+s.scrollbarWidth,'sc='+s.scrollbarColor,'gutter='+(e.offsetWidth-e.clientWidth),'overflows='+(e.scrollHeight>e.clientHeight)].join(' | ')}).join('\n'))
```

**Discord's own rules that mention a word** (here: scrollbar):
```js
const out=[];for(const s of document.styleSheets){let rs;try{rs=s.cssRules}catch{continue}const walk=rs=>{for(const r of rs){if(r.cssRules)walk(r.cssRules);if(!r.style)continue;const t=r.cssText;if(/scrollbar/.test(t)&&!/vc-|w98-/.test(t))out.push(t.replace(/\s+/g,' ').slice(0,300))}};walk(rs)}copy([...new Set(out)].join('\n'))
```

**Position and size of an element and its parents, plus overlays**:
```js
const r=e=>{const b=e.getBoundingClientRect(),s=getComputedStyle(e);return String(e.className).slice(0,60)+' | pos='+s.position+' top='+Math.round(b.top)+' bottom='+Math.round(b.bottom)+' h='+Math.round(b.height)+' mb='+s.marginBottom+' pb='+s.paddingBottom};const out=[];for(const sel of ['[class*="chatContent_"] [class*="scroller"]','[class*="sidebar_"] [class*="scroller"][class*="thin_"]']){let e=document.querySelector(sel);out.push('--- '+sel);for(let i=0;e&&i<5;i++,e=e.parentElement)out.push(r(e))}for(const sel of ['[class*="chatContent_"] form','[class*="panels_"]'])document.querySelectorAll(sel).forEach(e=>out.push('OVERLAY '+r(e)));copy(out.join('\n'))
```

---

## 3. How Discord defines colors

- `<html>` carries the classes `platform-win theme-dark theme-darker images-dark density-default font-size-16 ...`.
  Some containers repeat them; `sidebar__5e434` also has `theme-dark theme-darker`.
- Discord's variables are plain (non-`!important`) declarations on `.theme-dark`,
  `.theme-light`, `.theme-darker` and `.theme-midnight`. They're not in a cascade layer.
  Values look like `color-mix(in oklab, var(--neutral-69) 100%, var(--custom-theme-…) 0%)`.
- So overriding them needs `!important`. The theme also adds ID-level specificity
  as a safety margin, applied to `<html>` and any element whose class contains `theme-`:
  ```css
  :root:not(#w98-a):not(#w98-b),
  [class*="theme-"]:not(#w98-a):not(#w98-b) { --background-base-lower: … !important; }
  ```
  `:not(#id)` matches every element, but counts as an ID selector for specificity.
- Discord also re-sets some colors **locally**, for example inside the channel header.
  Inheriting a variable isn't enough there; set `color` on the elements directly.

---

## 4. Variables that exist (and the old ones that don't)

**No longer exist.** Setting them does nothing:
`--background-primary/secondary/tertiary`, `--background-floating`,
`--background-modifier-*`, `--header-primary/secondary`, `--text-normal`,
`--interactive-normal/hover/active`, `--font-display`.

**Their replacements and the useful ones:**

| Purpose | Variables |
|---|---|
| Outer frame | `--app-frame-background`, `--app-frame-border` |
| Surfaces | `--background-base-lowest/lower/low`, `--background-surface-high/higher/highest`, `--bg-surface-raised`, `--chat-background(-default)`, `--modal-background`, `--card-background-default`, `--embed-background` |
| Hover / selected | `--background-mod-muted/subtle/normal/strong`, `--interactive-background-default/hover/active/selected` |
| Text | `--text-default`, `--text-strong`, `--text-muted`, `--text-subtle`, `--text-link`, `--text-code`, `--channels-default`, `--chat-text-muted` |
| Icons | `--icon-default/strong/muted/subtle`, `--interactive-icon-default/hover/active`, `--interactive-text-default/hover/active`, `--channel-icon` |
| Inputs | `--input-background-default`, `--input-border-default/hover/active`, `--input-text-default`, `--input-placeholder-text-default`, `--channeltextarea-background`, `--custom-input-*` |
| Buttons | `--control-primary-*`, `--control-secondary-*`, `--control-critical-primary-*`, `--control-critical-secondary-*` (each with `background/border/text/icon` × `default/hover/active`) |
| Messages | `--message-background-hover`, `--message-mentioned-background-*`, `--message-highlight-background-*`, `--mention-background` |
| Checkboxes, radios, reactions | `--checkbox-*`, `--radio-*`, `--reaction-*` |
| Scrollbars | `--scrollbar-thin-thumb/track`, `--scrollbar-auto-thumb/track` |
| Radii | `--radius-none/xs/sm/md/lg/xl/xxl/round`, `--button-border-radius(-lg)`, `--custom-border-radius` |
| Fonts | `--font-primary`, `--font-headline`, `--font-code` |

**Watch out:** `--background-base-lowest` is used for the outer frame **and** for the
channel and DM lists. If you color it like a "desktop", those lists turn that color
too. Use `--app-frame-background` plus direct selectors for the outermost layer.

---

## 5. Class names (confirmed)

Classes look like `name_hash` or `name__hash` (for example `sidebar__5e434`,
`chat_f75fb0`). Never use the hash. Use one of these:

- `[class*="name_"]`: any class *containing* `name_`. Loose; `bar_` also matches `sidebar_`.
- `[class^="name_"], [class*=" name_"]`: a class that *starts with* `name_`. Use this by default.
- `:has()` to pick by structure. It's well supported in Discord's Chromium.

| Area | Classes / reliable selector |
|---|---|
| App root | `app__`, `appMount__`, `layers__`, `layer__ baseLayer__`, `base__`, `content__` |
| Server column | `guilds__`; the scroller's wrapper is `wrapper_ … guilds__` |
| Channel / DM sidebar | `sidebar__` > `sidebarList__` (+ `sidebarListRounded__`), resize handle `sidebarResizeHandle__` |
| User panel (bottom left) | `panels__` (position: absolute, **overlays** the sidebar bottom) |
| Main area | `page__` > `chat_` > `content_` > `chatContent_` > `messagesWrapper__` > `scroller__` (absolute) |
| Message box | `form_` (overlaps the chat scroller by 16px) > `channelTextArea_` > `scrollableContainer__` + `themedBackground__` > `textArea__` / `textAreaSlate__` |
| Chat fade behind input | `chatGradient__`, `chatGradientBase__` |
| Channel / Friends header | `container__ themed__` whose direct child is `upperContainer__`: `[class*="themed_"]:has(> [class*="upperContainer_"])`; title in `titleWrapper__` / `title__` |
| Header tabs (Online / All …) | `item_ themed_` (+ `selected_`) |
| Server name header | `header_` containing `headerContent_`; decoration layers `headerGlass_`, `headerEllipseBackdrop_`, `headerEllipseForeground_` |
| Top app bar | there are **two** `bar_` classes; the right one contains `winButtons_`: `div[class^="bar_"]:has([class*="winButtons_"])` |
| Window buttons | `winButton_`, `winButtonMinMax_`, `winButtonClose_` |
| Member list | `membersWrap_` > `members_`; groups `membersGroup_`, `membersGroupHeader_` |
| Channel rows | `containerDefault_`, `link__`, selected: `modeSelected__`, `selectedChannel_` |
| Buttons (old component) | `button__201d5` + `lookFilled_` / `lookOutlined_`; `lookBlank_` is icon-only |
| Buttons (new design system) | `button_` with a child `buttonChildrenWrapper_`: `button:has(> [class*="buttonChildrenWrapper_"])` |
| DM sidebar links | `linkButton__`, `link__`. These are `<a>`, so a global `a {}` rule hits them |

---

## 6. Gotchas we hit

1. **`*/` inside a CSS comment ends the comment early.** For example
   `--interactive-text-*/--interactive-icon-*`. The rest of the comment then becomes a
   broken selector, and the browser **silently drops the next rule**, in our case the
   whole variable block. Check with a search for `\*/.+`.
2. **`html:not(:focus-within)` is true almost all the time** (whenever no input is
   focused). It can't be used as "window inactive".
3. **Global `a { color }` recolors sidebar rows** (DMs, Nitro, Shop, Quests).
   Limit it to `[class*="markup_"] a` and embeds.
4. **Avatars and server icons are clipped by SVG masks** (`foreignObject[mask]`), so
   `border-radius` doesn't affect them. Remove `mask` to make them square.
5. **Scrollbar pseudo-elements aren't covered by `*`.** `border-radius: 0` must be set
   on `::-webkit-scrollbar-thumb` and `::-webkit-scrollbar-track` explicitly.
6. The **server name header** draws frosted glass and glow layers over its background.
   Hide `headerGlass_` / `headerEllipse*` or your background won't show.
7. After a big CSS splice, **check brace balance**. A stray `}` breaks the rule after it.

### Scrollbars specifically

- `<html>` has the class `has-webkit-scrollbar`, so Discord styles scrollbars with
  `::-webkit-scrollbar`. The standard `scrollbar-width` / `scrollbar-color` rules only
  apply under `.no-webkit-scrollbar`. Don't reset those globally: you'd re-show bars
  Discord hides with `scrollbar-width: none`.
- Scroller type classes: `thin_` (8px), `auto_` (16px), `none_` (0px, hidden),
  `fade_` (thumb and track `visibility: hidden` until hover). Others are hidden one-off
  (`noScrollbar_`, `display:none`, `width:0`).
- Force size **only** on `thin_` / `auto_` with `!important`. Give everything else a
  non-important default, so Discord's "hide this" rules (more specific) still win.
- Discord rounds thumb and track (`border-radius: 8px`, `border: 4px transparent`,
  `background-clip: padding-box`). Override all three.
- `.scroller__36d07:not(:hover)::-webkit-scrollbar-thumb { background: transparent }`
  makes the chat thumb vanish unless hovered. A `background-color … !important` beats it.
- The message box (`scrollableContainer__`) always has a 12px **transparent**
  scrollbar. Style it, and arrows appear next to the input. Keep its track transparent
  with no buttons.
- **Arrow buttons ignore margins, and always sit at the very ends of the scroller**,
  even with a track `margin-bottom` (tested in Chromium). So when a list scrolls under
  an overlay, shorten the scroller itself:
  - chat: `[class*="messagesWrapper_"] > [class*="scroller"] { bottom: 16px }`
  - server column wrapper: `margin-bottom: 66px`
  - channel/DM list (`sidebarList_`): `margin-bottom: 16px` (a wrapper already gives ~50px)
  - top gap on the DM list goes on `[class*="sidebarList_"] [class*="scrollerBase_"]`,
    not on `sidebarList_`, because that block also holds the search bar / server header.
- Set the `::-webkit-scrollbar` background to transparent. Otherwise the empty
  stretch beside an overlay shows as a stray strip.

---

## 7. Testing CSS outside Discord

The fastest loop is a mock page that copies Discord's `<html>` classes and the rules
you're fighting, then loads the theme:

```html
<!doctype html>
<html class="theme-dark theme-darker has-webkit-scrollbar">
<head>
<style>
  .theme-dark   { --background-base-lower: red }
  .theme-darker { --background-base-lower: blue }
  /* paste Discord rules from the "rules that mention a word" snippet here */
</style>
<link rel="stylesheet" href="win98.theme.css">
</head>
<body><div class="chat_f75fb0"><div class="chatContent_f75fb0">x</div></div></body>
</html>
```

Serve the folder (`python -m http.server 8765`), open it in any Chromium browser,
and check computed values in the console, for example
`getComputedStyle(document.documentElement).getPropertyValue('--background-base-lower')`.
If a rule is missing from `document.styleSheets[i].cssRules`, the parser dropped it
(see gotcha 1).
