# Brainvita

A browser version of Brainvita (peg solitaire) on the classic 33-hole cross board, with hints, scoring and a family leaderboard.

**Play:** https://yasaswinpalukuri.github.io/brainvita/

## How to play

- Enter your name (or tap it if you've played before) and start.
- Tap a marble to pick it up. Holes it can jump into glow.
- Tap a glowing hole: the marble jumps over its neighbour, and the jumped marble is removed.
- Jumps are straight only (up, down, left, right), never diagonal.
- The game ends when no jumps are left. One marble in the centre is a perfect game.
- Undo takes back your last move and Hint shows a move that still leads to a centre finish. Both cost points.

## Tech

One self-contained `index.html` with plain HTML, CSS and JavaScript. No build step, no dependencies. The solver runs in an inline Web Worker; the leaderboard uses the Firestore REST API when configured, otherwise `localStorage`.

## Run locally

Open `index.html` in any browser.

## Shared leaderboard (Firebase, free tier)

Without setup, scores stay on the device they were played on. To share one
leaderboard across the family's phones:

1. Go to https://console.firebase.google.com, create a project (Analytics not needed).
2. Build → Firestore Database → Create database (production mode, any nearby region).
3. Firestore → Rules, paste this and publish:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{db}/documents {
       match /brainvita_scores/{id} {
         allow read: if true;
         allow create: if request.resource.data.keys().hasOnly(
             ['name','score','at','left','centre','moves','jumps','undos','hints','seconds'])
           && request.resource.data.name is string
           && request.resource.data.name.size() > 0
           && request.resource.data.name.size() <= 20
           && request.resource.data.score is int
           && request.resource.data.score >= 0
           && request.resource.data.score <= 5000;
         allow update, delete: if false;
       }
     }
   }
   ```

4. Project settings (gear icon) → General → Your apps → add a Web app. Copy
   `projectId` and `apiKey` into the `LEADERBOARD` block at the top of the
   script in `index.html`.

The Firebase web API key is an identifier, not a secret; the rules above are
what protect the data (anyone can add a score, nobody can edit or delete one).
To remove a bad score, delete it in the Firebase console.

## Scoring

| Part | Points |
|---|---|
| Each marble removed | +100 |
| Finish with one marble in the centre | +1000 (one marble elsewhere: +300) |
| Each chain jump (same marble jumping again) | +40 |
| Speed bonus | up to +300, shrinks over the first 10 minutes |
| Each undo | −30 |
| Each hint | −150 |

## How hints work

- **Hint book (trie):** winning move sequences from the opening, stored as a
  trie keyed by moves. If your game so far is a prefix of a known winning line,
  the next move comes straight from the trie with no search.
- **Solver:** otherwise a depth-first search runs in a Web Worker, with a
  memo of dead board states (merged across the 8 symmetries) and pagoda
  functions that prove some boards lost without searching them. Every line it
  finds, and every real centre finish, is added to the hint book.
- If the position can't be won, it looks back through your moves and tells you
  how many undos get you to a winnable position.
