# Snake and Ladder — Low Level Design

Automated two-player Snake and Ladder on a 10x10 board with randomly placed snakes and ladders. Players roll a dice in turn until one reaches the last cell.

**Package:** `com.conceptcoding.interviewquestions.snakeNladder`

## Entry Point

`SnakeNLadderDemo.main()` — creates a `Game` and calls `startGame()`, which simulates the whole game and prints every turn, every snake/ladder jump and the winner.

Run `SnakeNLadderDemo` directly from your IDE. Output differs on every run since dice rolls and snake/ladder positions are random.

## Folder Structure

```
snakeNladder/
├── SnakeNLadderDemo.java   # Entry point (main method)
├── Game.java               # Game controller: setup, turn loop, jump handling, winner
├── Board.java              # N x N grid of cells, random snake/ladder placement
├── Cell.java               # A board cell, optionally holding a Jump
├── Jump.java               # Snake or ladder: start -> end position
├── Dice.java               # Rolls one or more six-sided dice
└── Player.java             # Player id and current position
```

## Classes

| Class | Responsibility |
|-------|----------------|
| `SnakeNLadderDemo` | Entry point; creates and starts a `Game`. |
| `Game` | Sets up a `Board(10, 5, 4)` (10x10, 5 snakes, 4 ladders), a single `Dice` and two players in a `Deque<Player>`. `startGame()` loops: picks the next player, rolls the dice, applies any snake/ladder via `jumpCheck()`, and declares the winner once a player reaches the last cell (`size * size - 1`). |
| `Board` | Creates the `Cell[][]` grid and randomly places snakes (`start > end`) and ladders (`start < end`). `getCell(position)` maps a linear position to its row/column. |
| `Cell` | A board square; holds a `Jump` if a snake or ladder starts here. |
| `Jump` | `start` and `end` positions. A snake when `start > end`, a ladder when `start < end`. |
| `Dice` | `rollDice()` returns the sum of `diceCount` random rolls between 1 and 6. |
| `Player` | Player `id` and `currentPosition`. |

## Design Notes

- **Single `Jump` abstraction:** snakes and ladders share one class; direction (`start` vs `end`) decides which it is.
- **Turn order with a Deque:** the current player is taken from the front and appended to the back each turn.
- **Configurable:** board size, number of snakes/ladders and number of dice are constructor parameters.
