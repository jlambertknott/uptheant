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
- `HIGHLIGHTS`: the three fact tiles at the top of the Route tab.
- `SPOT`: the wildlife checklist, as `[name, hint]` pairs.
- `QUIZ`: one object per question:
  - `r`: round name. Rounds appear in the order they first occur.
  - `q`: the question.
  - `o`: four options, shown in this order. Vary where the right answer sits.
  - `k`: the right answer, scrambled so it isn't obvious in the page source: `String.fromCharCode(97 + ((correctIndex * 7 + q.length) % 26))`, where correctIndex is 0 to 3. Recompute `k` whenever you change `q` or reorder `o`.
  - `f`: a fun fact shown after the reveal (can be "").
- The windy-day note, glossary and page titles are plain HTML above the script.

Question ids come from the round name plus question text, so editing a question that has already been asked mid-quiz detaches the answers to it. Avoid editing questions during a live quiz unless James asks.

## Live data (Firebase Firestore)

The `trip` collection holds three documents: `quiz` (current question, phase, revealed questions, offline code rounds), `answers` (`{questionId: {name: optionIndex}}`) and `helm` (log, fair-turn tracker, who's off duty). Rules allow read/write on `trip/*` only. Scores are worked out in the page from `answers` and the quiz key.

## Checking a change

Before pushing, extract the module script and run `node --check` on it (rename the imports out first, or check with `node --input-type=module --check`), and make sure every `QUIZ` entry has four options and a `k` that decodes to 0 to 3.
