# PROMPT FOR CLAUDE CODE — Lëtzebuergesch Snell! Language Trainer

---

## PROJECT OVERVIEW

Build **"Lëtzebuergesch Snell!"** — a personal Luxembourgish language trainer for Andrey (age 50, F1 fan, chess player, English speaker, living in Luxembourg, goal: pass the Sproochentest exam and get citizenship).

**ОПЯТЬ 25! ® — Languages for Life** series.

Single HTML file, no server, no frameworks, no dependencies.
Working reference: `lux_index.html` (current file in the repo).

---

## CRITICAL RULES

1. **Language**: ALL interface text in **English only**. Luxembourgish in lesson content. ZERO Russian anywhere.
2. **Architecture**: Single `index.html` file, vanilla JS, no build tools, no npm.
3. **Navigation**: Tab buttons must work immediately on page load — no `pointer-events:none`, no `opacity:0` blocking. Use direct `onclick="navHome()"` function calls, NOT `onclick="window.goHome&&window.goHome()"`.
4. **Startup screen**: Opens to **lesson list** (home screen), NOT splash/intro screen.
5. **Voice modal** (`z-index:9999`) must NOT appear within first 3 seconds — it blocks all clicks. Delay: 3000ms, auto-dismiss after 8 seconds.
6. **No Russian** in any field: not in `r:`, `ph:`, `ru:`, `ok:`, `no:`, `exp:`, `q:`, `o:[]`, UI strings, JS comments, or HTML.

---

## FILE STRUCTURE

Single `index.html`, ~140 KB:

```
index.html
├── <head>
│   ├── CSS (all styles inline in <style> tag)
│   └── Script 1: loading bar timeout (300ms → ready)
├── <body>
│   ├── Topbar (#topbar)
│   ├── Loading indicator (#load-bar)
│   ├── #main (scrollable content area)
│   │   ├── #scr-splash (hidden by default, display:none!important)
│   │   ├── #scr-home (class="scr on" — VISIBLE ON LOAD)
│   │   ├── #scr-lesson
│   │   ├── #scr-words
│   │   ├── #scr-exercise
│   │   ├── #scr-exam
│   │   └── #scr-progress
│   ├── #bottomnav (6 tab buttons)
│   └── #voice-modal
│
├── <script> 2: nav functions (navHome/navDialog/navWords/navExercises/navExam/navProgress)
└── <script> 3: main JS
    ├── var L = [...] (25 lesson objects)
    ├── var EXAM_QUESTIONS = [...] (10 exam questions)
    ├── State variables
    └── All functions (35 total)
```

---

## CSS DESIGN SYSTEM

```css
:root {
  --g: #C4282C;      /* primary red (Léon avatar, buttons) */
  --gd: #8B1A1E;     /* dark red */
  --gl: #FAEBEC;     /* light red */
  --pu: #7F77DD;     /* purple (John avatar) */
  --pl: #EEEDFE;     /* light purple */
  --or: #E8722A;     /* orange (Andrey avatar) */
  --ol: #FAECE7;     /* light orange */
  --am: #EF9F27;     /* amber (F1 lessons) */
  --bl: #1D6EA8;     /* blue */
  --bll: #E6F1FB;    /* light blue */
  --bdr: #E0E0E0;    /* border */
  --txt: #1A1A1A;    /* main text */
  --txt2: #666;      /* secondary text */
  --bg: #F4F6F8;     /* background */
  --r: 14px;         /* border radius */
}
```

Font: system `-apple-system, sans-serif`. Mobile-first, max-width flexible.

---

## NAVIGATION — CRITICAL IMPLEMENTATION

### HTML (nav buttons)
```html
<div id="bottomnav">
  <button class="nvi" id="nb-home" onclick="navHome()">
    <span class="nvi-i">🏠</span><span class="nvi-l">Lessons</span>
  </button>
  <button class="nvi" id="nb-lesson" onclick="navDialog()">
    <span class="nvi-i">💬</span><span class="nvi-l">Dialog</span>
  </button>
  <button class="nvi" id="nb-words" onclick="navWords()">
    <span class="nvi-i">📖</span><span class="nvi-l">Words</span>
  </button>
  <button class="nvi" id="nb-exercise" onclick="navExercises()">
    <span class="nvi-i">✏️</span><span class="nvi-l">Exercises</span>
  </button>
  <button class="nvi" id="nb-exam" onclick="navExam()">
    <span class="nvi-i">📝</span><span class="nvi-l">Exam</span>
  </button>
  <button class="nvi" id="nb-progress" onclick="navProgress()">
    <span class="nvi-i">📊</span><span class="nvi-l">Progress</span>
  </button>
</div>
```

### CSS (NO pointer-events blocking!)
```css
#bottomnav {
  position: fixed; bottom: 0; left: 0; right: 0;
  z-index: 999; background: #fff;
  border-top: 1px solid var(--bdr);
  display: flex;
  /* NEVER add opacity:.5 or pointer-events:none here */
}
.nvi { flex:1; border:none; background:none; padding:10px 2px 8px; cursor:pointer; }
```

### Script 2 (nav functions — DIRECT, no window. lookup):
```javascript
function navHome(){
  try {
    if(typeof buildHome==='function') buildHome();
    document.querySelectorAll('.scr').forEach(s=>s.classList.remove('on'));
    document.getElementById('scr-home').classList.add('on');
    document.querySelectorAll('.nvi').forEach(b=>b.classList.remove('on'));
    document.getElementById('nb-home').classList.add('on');
  } catch(e){ console.error('navHome:',e); }
}
function navDialog(){
  try {
    if(typeof curLesson!=='undefined' && curLesson && typeof showScr==='function'){
      showScr('lesson');
      document.querySelectorAll('.nvi').forEach(b=>b.classList.remove('on'));
      document.getElementById('nb-lesson').classList.add('on');
    } else { navHome(); }
  } catch(e){ console.error('navDialog:',e); }
}
// ... navWords, navExercises, navExam, navProgress same pattern
```

---

## LESSON DATA STRUCTURE

Each lesson object in `var L = [...]`:

```javascript
{
  id: 1,
  title: "Alfabet a kleng",      // Luxembourgish title (shown in lesson list)
  icon: "🔤",
  level: "A1",
  ru_title: "Alphabet & sounds", // English subtitle (shown as lesson card subtitle)
  type: "normal",                // "normal" | "f1" | "chess"
  words: [
    {
      cz: "Moien",               // Luxembourgish word/phrase
      ru: "Good morning / Hello", // English meaning
      ph: "[Móyyen]",            // Latin phonetics (NOT Russian Cyrillic!)
      ic: "☀️"                   // emoji icon
    },
    // ... 7-8 words per lesson
  ],
  sd: [   // Standard Dialog (Andrey tab)
    {
      w: "L",                    // speaker: "L"=Léon, "A"=Andrey, "J"=John
      n: "Léon",                 // display name
      t: "Moien, Andrey! Wéi geet et?",  // Luxembourgish text (spoken by TTS)
      ph: "[Móyyen, Andrey! Vey get et?]", // Latin phonetics
      r: "Good morning, Andrey! How are you?" // English translation
    },
    // ... 6-8 dialog lines per lesson
  ],
  jd: [   // John's Dialog (John A2 tab — more advanced version of same topic)
    // same structure as sd, but more complex Luxembourgish
  ],
  ex: [   // Exercises
    {
      q: "How do you say 'Hello' in Luxembourgish?",  // English question
      o: ["Hallo","Moien","Bonjour","Guten Morgen"],  // answer options (mix lb/en)
      c: 1,                      // correct answer index (0-based)
      ok: "Correct! Moien — the main Luxembourgish greeting",  // English feedback
      no: "No. Hello = Moien [MOY-en]",               // English wrong feedback
      exp: "Moien is used all day — morning, noon and evening."  // English explanation
    },
    // 2 exercises per lesson = 50 total
  ]
}
```

---

## CHARACTERS

| Avatar | Name | Color | Role |
|--------|------|-------|------|
| 🔴 **L** | Léon | `var(--g)` red | Native speaker, teacher |
| 🟠 **A** | Andrey | `var(--or)` orange | Learner A1 (the user) |
| 🟣 **J** | John | `var(--pu)` purple | Level A2, target in 6 months |

**Avatar colors in CSS:**
```css
.ava.L { background: linear-gradient(135deg, #C4282C, #8B1A1E); }
.ava.A { background: linear-gradient(135deg, #E8722A, #B84E18); }
.ava.J { background: linear-gradient(135deg, #7F77DD, #534AB7); }
```

**Dialog card rules:**
- ▶ button: Léon=red, Andrey=orange, John=purple
- ▶ on Andrey/John lines plays **Léon's model phrase** (not their own line)
- 🎤 "I speak!" button on ALL three characters
- Recording comparison panel: ▶ Léon | ▶ Me
- Speed buttons (0.7× / 1×) only on Léon's lines

---

## DIALOG SCREEN LAYOUT

```
┌─────────────────────────────────┐
│ [L] Léon — teacher              │ ← red avatar
│     Moien, Andrey! Wéi geet et? │ ← Luxembourgish (bold)
│     [Móyyen, Andrey!...]        │ ← Latin phonetics (italic, muted)
│     Good morning, Andrey!       │ ← English translation
│     [▶] [0.7×] [1×] [🎤 I speak!] │
│     [recording panel if active] │
├─────────────────────────────────┤
│ [A] Andrey — you!               │ ← orange avatar
│     Moien, Léon! Et geet mir... │
│     [Móyyen, Lyeon!...]         │
│     Good morning, Léon!...      │
│     [▶] [🎤 I speak!]           │
│     [recording panel if active] │
└─────────────────────────────────┘
```

Two tabs at top: **👤 Andrey** | **🧑 John A2**

---

## EXAM QUESTIONS STRUCTURE

```javascript
var EXAM_QUESTIONS = [
  {
    id: 1,
    type: "listen",              // "listen" | "vocab" | "grammar"
    q: "Léon seet: «Moien! Wéi geet et?» Wat äntwerts du?", // Luxembourgish question
    hint: "Léon greets you and asks how you are",  // English hint
    audio: "Moien! Wéi geet et?",                 // text for TTS
    o: ["Et geet mir gutt, merci!", "Jo, ech sinn hei.", "Äddi, Léon!", "Ech sinn krank."],
    c: 0,                        // correct index
    ok: "Parfait! Et geet mir gutt, merci! — standard reply",
    no: "No. To 'Wéi geet et?' reply: Et geet mir gutt, merci!",
    exp: "Wéi geet et? = How are you? Reply: Et geet mir gutt, merci!"
  },
  // ... 10 total
];
```

Exam: 10 questions, passing score 70% (7/10), unlimited retries, stats saved in `localStorage('lux77exam')`.

---

## VOICE / TTS

- Language: `de-DE` (German, closest to Luxembourgish in Web Speech API)
- Default speed: `curSpeed = 1.0`
- Slow: `0.7`
- `spk(text, rate)` — main TTS function
- Voice modal: shows after 3000ms, auto-hides after 8000ms
- Skip saves `localStorage.setItem('lux77voice', 'skip')`

```javascript
function spk(txt, rate) {
  if (!window.speechSynthesis) return;
  window.speechSynthesis.cancel();
  var u = new SpeechSynthesisUtterance(txt.replace(/&[a-z]+;/g,'').replace(/&#[0-9]+;/g,''));
  u.lang = 'de-DE';
  u.rate = rate || 1.0;
  if (deVoice) u.voice = deVoice;
  window.speechSynthesis.speak(u);
}
```

---

## MICROPHONE / RECORDING

```javascript
function startRec(btnId, modelTxt, rid) {
  // Ask permission, record via MediaRecorder
  // Store blob in recBlobs[rid]
  // Show: ▶ Léon | ▶ Me | Compare! panel
}
function playRec(rid) {
  // Play back recording from recBlobs[rid]
}
```

Recording panel HTML:
```html
<div class="splayer" id="pl_XXXX">
  <div class="splayer-lbl">🎧 My recording:</div>
  <div class="spl-row">
    <button class="spl-play" onclick="playRec('XXXX')">▶</button>
    <div class="spl-wave"><!-- 12 animated bars --></div>
  </div>
  <div class="cmp-row">
    <button class="cmp-l" onclick="spk('...', curSpeed)">▶ Léon</button>
    <button class="cmp-me" onclick="playRec('XXXX')">▶ Me</button>
    <span class="cmp-hint">Compare!</span>
  </div>
</div>
```

---

## SCREENS

### `#scr-home` (default visible on load)
- Mini welcome banner (red gradient): "Lëtzebuergesch Snell!" + "About" button
- Model Exam quick-access card + stats (Passed/Failed)
- `#lesson-grid` — 2-column grid of lesson cards
- Lesson card: `Lesson N · A1`, Luxembourgish title, emoji icon, F1/Chess tag if applicable
- Click → `openLesson(id)`

### `#scr-splash` (hidden: `display:none!important`)
- Characters intro (Léon/Andrey/John)
- Personalization block (F1 + Chess)
- "Start learning →" button → `navHome()`
- Accessible via "About" button on home screen

### `#scr-lesson` — Dialog
- Top: lesson subtitle + tip text
- Two tabs: Andrey / John A2
- Dialog cards (built by `buildDlg()`)
- Bottom: "✅ Lesson complete! Next →" button

### `#scr-words` — Vocabulary
- Word cards in grid, tap to hear pronunciation
- Each card: Luxembourgish, Latin phonetics, English meaning, emoji

### `#scr-exercise` — Exercises
- 2 exercises per lesson
- Multiple choice, 4 options
- Color feedback: green=correct, red=wrong
- Explanation shown after each answer

### `#scr-exam` — Sproochentest Model Exam
- 10 questions, 3 types: listen/vocab/grammar
- Progress: "Question X of 10"
- Results: score, Passed/Failed, counter, "Try again!"
- Stats saved to localStorage

### `#scr-progress` — Progress
- Counters: lessons completed, words learned (×8), days streak
- Exam stats: Passed ✅ / Failed ❌ / Success %
- Full lesson list with Done/Now/Soon badges

---

## localStorage KEYS

| Key | Content |
|-----|---------|
| `lux77` | JSON array of completed lesson IDs, e.g. `[1,2,3]` |
| `lux77exam` | JSON `{passed:0, failed:0}` |
| `lux77voice` | `"skip"` or `"done"` (voice modal dismissed) |

---

## LESSON LIST (all 25 A1 lessons)

| ID | Title (lb) | English subtitle | Type | Notes |
|----|-----------|-----------------|------|-------|
| 1 | Alfabet a kleng | Alphabet & sounds | normal | |
| 2 | Virstellung — Kennenléiere | Introductions | normal | |
| 3 | Zuelen 1-20 | Numbers 1–20 | normal | |
| 4 | Am Café | At the café | normal | |
| 5 | Am Supermarché | At the shop | normal | |
| 6 | Transport zu Lëtzebuerg | Transport in Luxembourg | normal | Free transport! |
| 7 | Zäit a Woch | Time & days | normal | |
| 8 | F1: Am Bistro bei der Course | F1: At the bar — watching the race | **f1** | 🏎️ |
| 9 | Am Restaurant | At the restaurant | normal | Judd mat Gaardebounen |
| 10 | D'Famill | Family | normal | |
| 11 | Schachclub Lëtzebuerg | Chess: At the Luxembourg club | **chess** | ♟️ |
| 12 | Gesondheet | Health | normal | |
| 13 | Wunnen zu Lëtzebuerg | Housing in Luxembourg | normal | Cross-border workers |
| 14 | Beruffer a Schaffen | Jobs & work | normal | |
| 15 | Sprooch zu Lëtzebuerg | Languages in Luxembourg | normal | Sproochentest |
| 16 | Wieder a Saisons | Weather & seasons | normal | |
| 17 | Emotiounen a Gefiller | Emotions & feelings | normal | |
| 18 | Bei der Post a Bank | At the post office & bank | normal | |
| 19 | Telefongespréich | Phone conversation | normal | |
| 20 | A1 Ofschlossdialog | A1 Final dialogue | normal | |
| 21 | Aarbecht kréien | Getting a job — interview | normal | |
| 22 | Beim Dokter detailléiert | At the doctor — in detail | normal | |
| 23 | Um Gemengamt | At the government office | normal | |
| 24 | Schachclub Tournoi | Chess: Tournament | **chess** | ♟️ 7 rounds |
| 25 | Ofschlossdialog A1 + F1 + Schach | A1 Final: F1 + Chess | **f1** | 🏎️♟️ |

---

## LESSON CARD VISUAL INDICATORS

```css
/* F1 lesson — amber left border */
.lcard.f1 { border-left: 4px solid var(--am); }

/* Chess lesson — purple left border */
.lcard.chess { border-left: 4px solid var(--pu); }

/* Completed lesson */
.lcard.done { background: #E8F8F0; border-color: #1D9E75; }

/* Current lesson */
.lcard.cur { border: 2px solid var(--g); }
```

---

## KEY JS FUNCTIONS

```javascript
// State
var curLesson = null;   // current lesson object
var curSpeed = 1.0;     // TTS speed
var done = [];          // completed lesson IDs from localStorage
var deVoice = null;     // German TTS voice object
var recBlobs = {};      // recorded audio blobs {rid: Blob}
var examStats = {passed:0, failed:0}; // from localStorage

function buildHome()      // builds lesson grid, updates exam mini
function openLesson(id)   // loads lesson, shows dialog screen
function buildLesson(l)   // renders lesson header, tip, tabs
function buildDlg(dialogs, type) // renders dialog cards HTML
function switchDlg(type)  // switches Andrey/John tab
function buildWords()     // renders vocabulary cards
function buildExercises() // renders exercise questions
function buildExam()      // renders full exam
function buildProgress()  // renders progress screen
function markDone(id)     // marks lesson complete, saves to localStorage
function showScr(name)    // shows screen by name, hides others
function setNav(id)       // sets active nav button
function spk(txt, rate)   // TTS speak
function startRec(btnId, modelTxt, rid)  // start voice recording
function playRec(rid)     // play back recording
function setSpd(rate, btn, groupId)  // set speed, update button UI
function checkVoiceAndShow()  // show voice setup modal (delay 3000ms)
function closeVoiceModal(status) // close voice modal
function setReady()       // called after init — ensure UI is interactive
```

---

## INITIALIZATION (end of main script)

```javascript
// Called at the very bottom of Script 3
buildHome();
showScr('home');
setNav('nb-home');
checkVoiceAndShow();  // delays 3000ms
setReady();           // ensure buttons work
```

---

## KNOWN BUGS TO AVOID

1. **Nav buttons not working**: NEVER use `pointer-events:none` on `#bottomnav` or `.spl-btn`. Use direct `onclick="navHome()"` not `onclick="window.goHome&&window.goHome()"`.

2. **Voice modal blocks UI**: It has `z-index:9999` and covers everything. Delay must be ≥3000ms. Always add auto-dismiss timeout.

3. **Lesson list empty**: `buildHome()` must be called BEFORE `showScr('home')`. `goHome()` in main JS must also call `buildHome()` first.

4. **Splash screen showing instead of home**: `#scr-splash` must have `display:none!important` in inline style. `#scr-home` must have `class="scr on"`.

5. **Russian text remaining**: Check ALL fields — `r:`, `ru:`, `ph:`, `ok:`, `no:`, `exp:`, `q:`, `o:[]`. Phonetics must be Latin (e.g. `[MOY-en]`) not Cyrillic (`[Мойен]`).

6. **lcard-name showing Russian**: `buildHome()` must use `l.title` (Luxembourgish) NOT `l.ru_title`.

7. **Progress list showing Russian**: `buildProgress()` must use `'Lesson ' + l.id + ' — ' + l.title` NOT `l.ru_title`.

---

## PLANNED FUTURE FEATURES (do NOT build now, just note)

- **Lessons 26–77**: A2 block (26-50) + B1 block (51-77)
- **HuggingFace Space**: Pre-recorded MP3 files for Luxembourgish (voice: Maxine via CoquiTTS)
- **PWA**: Service worker, offline mode, installable on Android/iOS
- **Google Play**: PWABuilder export, account `tenzoretto@gmail.com`
- **Commercial version**: Stripe payments, user accounts, database, personal cabinet
- **Onboarding**: Ask 2 interests → personalize dialogs

---

## REPOSITORY INFO

- **Repo**: `perimeter-hub/Lux_Training`
- **URL**: `https://perimeter-hub.github.io/Lux_Training/`
- **File**: `index.html` (single file deployment)
- **Brand**: ОПЯТЬ 25! ® — Languages for Life
- **Trademark logo**: 🇨🇿🇩🇪🇮🇱🇬🇧🇱🇻🇬🇪🇪🇪🇵🇹 with golden "25"

---

## TASK FOR CLAUDE CODE

1. Read `index.html` from the repo
2. Fix navigation (use pattern above)
3. Ensure home screen opens on load
4. Verify zero Russian in the file
5. Test all 6 nav buttons work
6. Commit as `index.html`

OR: Rebuild from scratch using this spec — the data (25 lessons, 10 exam questions) is in the existing `index.html`.
