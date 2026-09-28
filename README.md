# Grid Pursuit — Police vs Thief

A turn-based chase on an 10×10 grid, built to showcase **shortest-distance
pathfinding (BFS)**: the AI opponent — whichever side you don't play —
always plans its moves using shortest-path search across the board.

No build step, no dependencies. Open `index.html` and play.

## How to play

Open `index.html` in any modern browser and pick a mode from the main menu:

**Play vs AI** — pick a side, **Police** (hunter) or **Thief** (evader),
and a difficulty (Easy / Intermediate / Hard). The AI takes the other
side and plans its moves with the same shortest-path search described
below.

**Offline · 2 Player** — enter two names, choose who's Police for round 1
(Player 2 gets the opposite role), then hand the device back and forth.
Round 2 swaps roles. Between every turn, the screen blurs for 3 seconds
with a "pass the device" prompt so the incoming player doesn't see the
outgoing player's position. See **Scoring** below for how the winner is
decided.

**Online · 2 Player** — play a friend on their own device. One player picks
**Host room**, enters a name and gets a 5-character room code; the other picks
**Join room**, enters a name plus that code. Same two-round format and scoring
as offline mode, but there is no pass-the-device blur since each player has
their own screen. See **Online play** below for how it works.

Click a highlighted tile on your turn to move there. Each turn you also
have **8 seconds to act** — if the clock runs out, a random legal move is
made for you automatically, so no one can stall the whole match out.

The overall match clock is **5 minutes**: the Thief wins a round by
surviving it, the Police win by making the capture first.

### Difficulty (Play vs AI)

| | Easy | Intermediate | Hard |
|---|---|---|---|
| Reaction speed | slow | moderate | fast |
| Move quality | often random | mostly optimal | always shortest-path optimal |
| Power usage | rare / mistimed | fairly reliable | used at the ideal moment |

### Scoring (Offline · 2 Player)

Both players play Police exactly once — round 1 with the roles you
picked, round 2 with them swapped. Whichever player, **as Police**, makes
their capture in less time wins the match. If only one of the two rounds
ends in a capture, that catcher wins outright. If neither round ends in a
capture, the match is a draw.

## Rules

| | Police | Thief |
|---|---|---|
| Movement per turn | 2 tiles | 1 tile |
| Vision radius | 3 tiles | 5 tiles |
| Win condition | Step onto the Thief's tile | Survive the 5-minute timer |

Movement is 4-directional (up/down/left/right) and distances are computed
with a real breadth-first search over the grid, not just straight-line
math — so the logic is ready to extend with walls or obstacles later
without changing how movement or vision is calculated.

### Powers (rechargeable)

Every power comes back **5 moves after you use it** — five of your own moves,
counted after the turn you used it on. The dashboard shows each item as
`ready` or `recharging · N`. This applies in all three modes.

**Thief**
- **Stopper** — freezes the Police for their next turn. Costs your turn to use.
- **Teleport** — jump instantly to a tile far from the Police. Costs your turn to use.

**Police**
- **Indicator** — reveals the Thief's rough compass direction (N/S/E/W or a
  combination), but never the distance. Costs your turn to use.
- **Jump** — move up to 4 tiles in one turn instead of 2. Replaces your normal move.

Each side gets exactly one turn's worth of action: a normal move, **or**
one of their two powers. Two different kinds of pause protect that:

- **Indicator** reveals its reading, then freezes the clock for 5 seconds
  *afterward* so you actually have time to read it before the turn moves on.
- **Stopper**, **Jump**, and **Teleport** freeze the clock the moment you
  click them — *before* anything happens — so you get unhurried thinking
  time to pick a destination tile (Jump/Teleport) or just confirm the play
  (Stopper). Once you act, the turn resolves immediately.

### Fog of war

- The Police only see the Thief's token when the Thief is within 3 tiles.
- The Thief only sees the Police's token when the Police is within 5 tiles.
- Outside those radii, the opposing token is simply not drawn — you're
  playing on partial information, same as the AI is.

## Online play

Online mode is peer-to-peer over WebRTC using [PeerJS](https://peerjs.com/)
(loaded from a CDN — still no build step or server of your own). The room code
is the host's PeerJS id.

- The **host's** browser runs the whole game (same engine as offline) and is the
  source of truth; the **guest's** browser sends its clicks and renders what the
  host sends back.
- The host only sends the guest the opponent's position when it is inside the
  guest's vision radius, so fog of war holds up online.
- The 8-second turn clock and 5-minute match clock run on the host and are
  mirrored to the guest. Rounds and scoring work as in offline mode.
- If either player disconnects mid-match, the match ends and the other player
  returns to the menu.

Needs an internet connection (for PeerJS and its free signaling server). Some
strict networks/NATs block direct WebRTC connections; the default setup has no
TURN relay, so in rare cases two players may be unable to connect.

## Project structure

```
police-vs-thief-game/
├── index.html          # page shell, setup / game / result screens
├── css/
│   └── style.css        # dark tactical theme, layout, board styling
├── js/
│   ├── pathfinding.js   # BFS shortest-distance / shortest-path / vision helpers
│   ├── game.js          # game state, rendering, turn loop, AI, timer
│   └── online.js        # online mode: host/join, PeerJS networking, snapshots
├── LICENSE
└── README.md
```

## The shortest-distance logic

`js/pathfinding.js` is the core reusable piece:

- `shortestDistance(a, b)` — BFS step-count between two tiles.
- `shortestPath(a, b)` — full shortest route between two tiles.
- `reachableWithin(origin, steps)` — every tile reachable within a move
  budget, used to highlight legal moves each turn.
- `sightDistance(a, b)` — Chebyshev distance, used only for the vision
  radii (an 8-directional "can I see you" check, separate from movement).
- `compassDirection(a, b)` — coarse direction label for the Police's
  Indicator power.

The AI uses these same functions: the Police AI chases the Thief's last
known position along the shortest BFS path, and the Thief AI evaluates
its reachable tiles each turn and picks whichever one maximizes shortest
distance from the Police.

## Ideas for extending this

- Add wall/obstacle tiles — `pathfinding.js` is already obstacle-ready,
  you'd only need to make `neighborsOf()` skip blocked cells.
- Persist match results with `localStorage` for a simple win/loss record.

## License

MIT — see [LICENSE](LICENSE).
