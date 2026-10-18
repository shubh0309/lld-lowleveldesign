# Tic-Tac-Toe — Low Level Design

Interactive two-player Tic-Tac-Toe on a 3x3 board, played from the console. Players take turns entering a `row,column`; the game ends with a win (full row, column, diagonal or anti-diagonal) or a draw when the board is full.

**Package:** `com.conceptcoding.interviewquestions.tictactoe`

## Entry Point

`PlayGame.main()` — creates a `TicTacToeGame`, initializes it, runs the game loop and prints the result (`WIN` with the winner's name, or `DRAW`).

Run `PlayGame` directly from your IDE. When prompted, enter a zero-based position such as `0,2`.

## Folder Structure

```
tictactoe/
├── PlayGame.java            # Entry point (main method)
├── TicTacToeGame.java       # Game controller: players queue, game loop, winner check
└── model/
    ├── Board.java           # N x N grid of PlayingPiece, add piece, free cells, print
    ├── Player.java          # Player name + assigned PlayingPiece
    ├── PlayingPiece.java    # Base piece holding a PieceType
    ├── PlayingPieceX.java   # X piece
    ├── PlayingPieceO.java   # O piece
    ├── PieceType.java       # X, O
    └── GameStatus.java      # WIN, DRAW
```

## Classes

### Core

| Class | Responsibility |
|-------|----------------|
| `PlayGame` | Entry point; starts the game and prints the final `GameStatus`. |
| `TicTacToeGame` | Holds a `Deque<Player>` for turn order, the `Board` and the winner. `initializeGame()` creates Player1 (X), Player2 (O) and a 3x3 board. `startGame()` loops: takes the next player, reads `row,column` from stdin, places the piece (re-prompting the same player on an occupied cell), and calls `checkForWinner()`. Returns `WIN` or `DRAW`. |

### Model (`model/`)

| Class | Responsibility |
|-------|----------------|
| `Board` | `PlayingPiece[][]` grid of configurable `size`. `addPiece()` places a piece if the cell is empty, `getFreeCells()` lists empty cells (as ANTLR `Pair<Integer, Integer>`), `printBoard()` renders the grid. |
| `Player` | Player `name` and their `PlayingPiece`. |
| `PlayingPiece` | Base class holding a `PieceType`. |
| `PlayingPieceX` / `PlayingPieceO` | Concrete pieces for `X` and `O`. |
| `PieceType` | Enum: `X`, `O`. |
| `GameStatus` | Enum: `WIN`, `DRAW`. |

## Design Notes

- **Turn order with a Deque:** the current player is removed from the front and added to the back after a valid move (or back to the front after an invalid one).
- **Extensible pieces:** new symbols can be added by extending `PlayingPiece` and adding a `PieceType`.
- **Board size:** `Board` supports any `N x N`; `TicTacToeGame` uses 3.
- **Dependency:** `Board` and `TicTacToeGame` use `org.antlr.v4.runtime.misc.Pair` from the `antlr4-runtime` dependency in `pom.xml`.
