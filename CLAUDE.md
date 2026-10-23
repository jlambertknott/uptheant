# Up the Ant: trip app

A one-page web app for James's Norfolk Broads sailing trip with Jimmy, Anna, Pippa, Simon, Sally and Charly.
Live site: https://jlambertknott.github.io/up-the-ant/

James is not a developer. He asks for changes in plain English, often from his phone on the boat. Keep replies short, make the change, publish it, and tell him to refresh.

## How publishing works

- Everything is in `index.html`. There is no build step.
- Commit and push your branch. The **Publish site** workflow (`.github/workflows/publish.yml`) merges your branch into `main` and publishes it to GitHub Pages, so the site updates about a minute later.
- Don't open a pull request or ask James to merge anything.
- After pushing, tell James: "Done, it'll be live in about a minute. Everyone just needs to refresh the page."

## Where things are in index.html

All content is near the top of the `<script type="module">` block:

- `CREW`: the six names. Used for the quiz, leaderboard and helm wheel.
- `HOST_PIN`: the quizmaster PIN (currently 2026).
- `firebaseConfig`: the live database connection. Don't change it.
- `STOPS`: the route, in order. Fields: `name`, `river` ("bure" = dark blue line, "ant" = light blue, "dyke" = green), `label` (optional heading shown above the first stop of a new stretch of water), `sub`, `hist`, `geo`, `legend` (optional), `trivia`, `tags` (array).
- `HIGHLIGHTS`: the pool of fact tiles. Three are picked at random on each page load, so add to the pool rather than swapping facts in and out. Each entry is `[big bit, the rest]`; keep the big bit short.
- `SPOT`: the wildlife checklist, as `[name, hint]` pairs.
- `QUIZ`: one object per question:
  - `r`: round name. Rounds appear in the order they first occur.
  - `q`: the question.
  - `o`: four options, shown in this order. Vary where the right answer sits.
  - `k`: the right answer, scrambled so it isn't obvious in the page source: `String.fromCharCode(97 + ((correctIndex * 7 + q.length) % 26))`, where correctIndex is 0 to 3. Recompute `k` whenever you change `q` or reorder `o`.
  - `f`: a fun fact shown after the reveal (can be "").
- The windy-day note, glossary and page titles are plain HTML above the script.

## Pictures

The drawings are hand-made SVG, not photos, so there are no image files to upload and nothing extra to download on a bad signal.

- They live in one `<svg class="sprite">` block at the very top of `<body>`, one `<symbol>` per drawing.
- The Route tab's banner is `art-hero`. Each stop names its drawing in its `art` field (`art-wroxham`, `art-horning`, `art-stbenets`, `art-antmouth`, `art-howhill`, `art-irstead`, `art-barton`, `art-neatishead`, `art-stalham`); a stop with no `art` field simply shows no picture.
- Every colour is a CSS variable starting `--a-` (set twice at the top of the stylesheet: once for light mode, once for dark), so the drawings follow the phone's theme. Never hard-code a colour inside a symbol.
- Each `<symbol>` has `viewBox="0 0 640 200"` (the hero is `0 0 640 260`) and the CSS pins the same aspect ratio, so nothing is ever cropped. If you add a drawing, match that viewBox.
- They were drawn with a generator script rather than by hand; it isn't in the repo, so edit the SVG directly or redraw.

Question ids come from the round name plus question text, so editing a question that has already been asked mid-quiz detaches the answers to it. Avoid editing questions during a live quiz unless James asks.

## Photos (the Across the Years tab)

- Photos live in the `photos/` folder in the repo. James adds them through GitHub's "Add files via upload" (25 MB per file through the browser).
- The **Publish site** workflow resizes each one into `_site/photos/` (long edge 1600) and `_site/photos/t/` (thumbnails, 700), and writes `_site/photos.json`, which is what the page reads. Nothing in `index.html` needs editing when a photo is added.
- Name a file `01-whatever.jpg` to control the order; the number is stripped for the display name.
- Captions are written in the app, not in the filename, and live in Firestore.

## Live data (Firebase Firestore)

The `trip` collection holds: `quiz` (current question, phase, revealed questions, offline code rounds), `answers` (`{questionId: {name: optionIndex}}`), `helm` (log, fair-turn tracker, who's off duty), `gallery` (`{photoKey: {t, by, at}}`, one caption per photo) and `journal` (`{entries: {id: {by, at, title, body, photo, comments[]}}}`). Each journal photo is a document of its own, `jp_<entryId>`, holding a compressed data URI, because Firestore caps one document at 1 MB; `shrink()` in the page scales and re-compresses until a photo fits inside that. Rules allow read/write on `trip/*` only, and the page only ever reads documents by name, never a collection query. Scores are worked out in the page from `answers` and the quiz key.

## Checking a change

Before pushing, extract the module script and run `node --check` on it (rename the imports out first, or check with `node --input-type=module --check`), and make sure every `QUIZ` entry has four options and a `k` that decodes to 0 to 3.
