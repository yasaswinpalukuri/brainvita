# Brainvita

A browser version of Brainvita (peg solitaire) on the classic 33-hole cross board.

**Play:** https://yasaswinpalukuri.github.io/brainvita/

## How to play

- Tap a marble to pick it up. Holes it can jump into glow.
- Tap a glowing hole: the marble jumps over its neighbour, and the jumped marble is removed.
- Jumps are straight only (up, down, left, right), never diagonal.
- The game ends when no jumps are left. One marble in the centre is a perfect game.

Undo takes back your last move, and your best result is saved in your browser.

## Tech

One self-contained `index.html` with plain HTML, CSS and JavaScript. No build step, no dependencies.

## Run locally

Open `index.html` in any browser.
