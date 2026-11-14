# CPR Coach (hackathon starter)

One file: `index.html`. All the HTML, CSS and JavaScript are inside it, so it looks the same wherever you open it. No build step, no npm. It's designed for desktop/laptop screens.

`camera.html` is the standalone, full-screen version of the camera coach (phone-friendly). The same coach is built into the **Coach** page of `index.html`.

## How to open it

- Quickest: double-click `index.html`. Everything works except the camera, which browsers only allow on `localhost` or `https`.
- Better (camera works): in this folder run `python3 -m http.server 8000`, then open http://localhost:8000 (or use VS Code "Live Server").
- To share a live link: drag the folder onto Netlify Drop, or push it to GitHub and turn on GitHub Pages.

Needs internet for the fonts (Google Fonts) and the camera coach's pose model (MediaPipe from jsDelivr / Google Storage, about 10 MB, loaded the first time you open the Coach page). Without internet it still works, just with system fonts and no camera coach.

## Where things are inside index.html

Search for these comment headers:

| Search for | What it controls |
|---|---|
| `All styles` | All CSS. Colors are at the top in `:root`. |
| `config.js` | Settings: `chatUrl` (the CPR agent), `naturalVoice`, metronome BPM, rescuer swap time. |
| `agent.js` | The CPR agent: asking, natural voice, hands-free listening, and the assistant bar. |
| `metronome.js` | Start CPR: 110 bpm beat, 30:2 counter, 2-minute rescuer swap timer, beep and voice. |
| `learn.js` | Learn tabs, Emergency steps, and the quiz questions (`QUIZ` array). |
| `chat.js` | Ask page chat: mic, patient age, read-aloud, sources. |
| `coach.js` | Camera coach: MediaPipe Pose tracks compression rate, chest recoil and elbow lock, with voice cues. Tuning values are in its `CONFIG` block. |
| `app.js` | Switches pages using the URL hash (`#home`, `#emergency`, ...). |

Page content (text, steps, Learn facts) is plain HTML in the `<main>` section, one `<section class="page">` per page.

## The RAG assistant (live)

The site talks to the CPR RAG agent on Cloudflare: `https://cpr-rag-agent.andrewozimek01.workers.dev/ask` (`chatUrl` in the config block). It uses Workers AI over a knowledge base written from the 2025 AHA CPR guidelines: Llama 3.1 8B for fast answers during CPR, Llama 3.3 70B for fuller explanations, instant pre-written answers for the most common questions, and Deepgram Aura for a natural speaking voice.

Where it shows up:

| Where | What it does |
|---|---|
| Assistant bar (bottom of every page) | Type a question any time, or tap **Hands-free** once and just talk. Works while the metronome or camera coach is running: answers are short while CPR is going, the coach and metronome voices stay quiet while it answers, and a new question cuts off the old answer. |
| Ask page | Full chat with explained answers, a tap-to-talk mic, patient age picker, read-aloud and the guideline section each answer came from. |

Request / response (see `agent.js` in index.html):

```json
POST /ask
{ "question": "It's a child, what do I do?", "mode": "emergency", "patient": "child",
  "session": { "rate": 96, "elapsedSeconds": 140 }, "history": [{ "role": "user", "content": "..." }] }
-> { "answer": "...", "cached": true, "sources": [{ "title": "...", "section": "Child CPR", "score": 0.96 }], "latencyMs": 120 }

POST /tts   { "text": "..." }   -> MP3 audio
```

Set `naturalVoice: false` in the config to use the browser's built-in voice instead. If `chatUrl` is empty, the Ask page falls back to offline placeholder replies. Voice questions need Chrome or Edge; other browsers can still type. The agent's own test page is at `https://cpr-rag-agent.andrewozimek01.workers.dev/`.

## Notes

- The Learn facts follow the 2025 AHA guidelines (infant compressions use the heel of one hand or two thumbs; the two-finger method was dropped).
- The camera coach turns the camera off when you leave the Coach page.
- Every page says it is not a replacement for certified CPR training or for calling 911. Keep that.
