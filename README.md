# CPR Coach (hackathon starter)

One file: `index.html`. All the HTML, CSS and JavaScript are inside it, so it looks the same wherever you open it. No build step, no npm. It's designed for desktop/laptop screens.

`camera.html` is the standalone, full-screen version of the camera coach (phone-friendly). The same coach is built into the **Coach** page of `index.html`.

## How to open it

- Quickest: double-click `index.html`, or press F5 in VS Code ("Open index.html in Chrome"). In Chrome everything works, including the camera. Other browsers may block the camera on a file, so use the local server below.
- Better (camera works): in this folder run `python3 -m http.server 8000`, then open http://localhost:8000 (or use VS Code "Live Server").
- To share a live link: drag the folder onto Netlify Drop, or push it to GitHub and turn on GitHub Pages.

Needs internet for the fonts (Google Fonts) and the camera coach's pose model (MediaPipe from jsDelivr / Google Storage, about 10 MB, loaded the first time you open the Coach page). Without internet it still works, just with system fonts and no camera coach.

## Where things are inside index.html

Search for these comment headers:

| Search for | What it controls |
|---|---|
| `All styles` | All CSS. Colors are at the top in `:root`. |
| `config.js` | Settings: `chatUrl`, metronome BPM, rescuer swap time. |
| `metronome.js` | Start CPR: 110 bpm beat, 30:2 counter, 2-minute rescuer swap timer, beep and voice. |
| `learn.js` | Learn tabs, Emergency steps, and the quiz questions (`QUIZ` array). |
| `chat.js` | Chat box. Placeholder reply until `chatUrl` is set. |
| `coach.js` | Camera coach: MediaPipe Pose tracks compression rate, chest recoil and elbow lock, with voice cues. Tuning values are in its `CONFIG` block. |
| `app.js` | Switches pages using the URL hash (`#coach` is the main page, then `#metronome`, `#emergency`, ...). |

Page content (text, steps, Learn facts) is plain HTML in the `<main>` section, one `<section class="page">` per page.

## Plugging in the RAG assistant

Set `chatUrl` in the config block to your Cloudflare Worker. The page sends

```json
POST /chat
{ "message": "How deep?", "history": [{ "role": "user", "content": "..." }] }
```

and expects `{ "reply": "...", "sources": ["optional"] }`. Same idea as the HEARTS worker: BM25 over a CPR JSON knowledge base, top passages into the Gemini prompt, return the answer. Add CORS headers (`Access-Control-Allow-Origin: *`) on the worker.

## Notes

- The Learn numbers are a placeholder summary. Check them against current AHA / Red Cross guidance before judging.
- The camera coach turns the camera off when you leave the Coach page.
- Every page says it is not a replacement for certified CPR training or for calling 911. Keep that.
