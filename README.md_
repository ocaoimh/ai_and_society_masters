# Artificial Intelligence and Society
A self-contained slide deck (built with [reveal.js](https://revealjs.com/), the HTML presentation framework) for the PSL *Artificial Intelligence and Society* master's, with live audience polls. The deck is one HTML file; the polling runs on a free Google Apps Script backend attached to a Google Sheet.

| File | What it is |
|---|---|
| `bsf-ai-society-deck.html` | The whole deck. Fonts, images, reveal.js and the QR library are embedded, so it runs offline from any browser. |
| `Code.gs` | The Apps Script backend (collects and tallies votes). |
| `Participant.html` | The page students see on their phones when they scan the QR code. |

## Running the deck (no polling)

Just open `bsf-ai-society-deck.html` in Chrome or Edge. Use the arrow keys to move, `S` for speaker notes, `F` for fullscreen, `Esc` for the slide overview. The poll slides show a "live voting isn't connected" note and you can run them as a show of hands. Nothing else is needed.

## Running the deck with live polling

Students scan a QR code (or open a short link), answer on their phones, and the results appear on the next slide in real time. Setup is about ten minutes, once.

### 1. Create the backend

1. Create a new Google Sheet (any name).
2. In the Sheet: **Extensions > Apps Script**.
3. In the editor, replace the contents of `Code.gs` with this repo's `Code.gs`.
4. Add an HTML file: next to **Files**, click **+ > HTML**, name it exactly `Participant` (capital P, no extension), and paste in this repo's `Participant.html`.
5. In `Code.gs`, change `PRESENTER_KEY` from `'change-me'` to a word only you know. Save (Ctrl/Cmd+S).

### 2. Publish it

1. **Deploy > New deployment > Web app.**
2. Set **Execute as: Me** and **Who has access: Anyone**. Deploy and authorise when prompted.
3. Copy the **Web app URL** (it ends in `/exec`).

> Every time you later edit `Code.gs`, you must re-publish for the change to go live: **Deploy > Manage deployments > (pencil) > Version: New version > Deploy**. The URL stays the same.

### 3. Point the deck at the backend

Open `bsf-ai-society-deck.html` in a text editor (not a browser). Search for `CONFIG` near the bottom and fill in the three values:

```js
var CONFIG = {
  apiUrl: 'https://script.google.com/macros/s/XXXXX/exec',  // your /exec URL
  presenterKey: 'your-word',                                 // must match PRESENTER_KEY in Code.gs
  shortLink: 'shorturl.at/xxxx',                             // optional, shown under the QR code
```
