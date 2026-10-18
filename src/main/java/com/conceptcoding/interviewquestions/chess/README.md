# Chess — Low Level Design

Two-player chess on an 8x8 board. Each piece type validates its own moves, the `Game` keeps a move history, and capturing the opposing `King` ends the game.

**Package:** `com.conceptcoding.interviewquestions.chess`

## Entry Point

`ChessDemo.main()` — creates two players (Alice as WHITE, Bob as BLACK) and plays a scripted Fool's Mate sequence, printing game status after each move and the full move history at the end.

Run `ChessDemo` directly from your IDE.

## Folder Structure

```
chess/
├── ChessDemo.java          # Entry point (main method)
├── Game.java               # Game controller: turns, move execution, status, history
├── Board.java              # 8x8 grid of cells, initial piece placement
├── Cell.java               # A square on the board (Position + optional Piece)
├── Position.java           # Row/column coordinate, printed in algebraic notation (e.g. e4)
├── Move.java               # A single move: player, start/end cell, piece moved/killed
├── Player.java             # Player name and playing side (colour)
├── enums/
│   ├── GameStatus.java     # IN_PROGRESS, BLACK_WIN, WHITE_WIN, DRAW, RESIGNATION
│   ├── PieceColour.java    # WHITE, BLACK
│   └── PieceType.java      # KING, QUEEN, BISHOP, KNIGHT, ROOK, PAWN
└── pieces/
    ├── Piece.java          # Abstract base: colour, type, captured flag, isValidMove()
    ├── King.java
    ├── Queen.java
    ├── Rook.java
    ├── Bishop.java
    ├── Knight.java
    └── Pawn.java
```

## Classes

### Core

| Class | Responsibility |
|-------|----------------|
| `Game` | Holds both players, the `Board`, the current turn, `GameStatus` and the list of `Move`s. `playMove(player, start, end)` builds a `Move` and executes it if the piece belongs to the player and `isValidMove` passes. Marks captured pieces, records history, switches turns, and sets `WHITE_WIN`/`BLACK_WIN` when a `King` is captured. |
| `Board` | Creates the 8x8 `Cell[][]` grid and places all 32 pieces in their starting positions (white on rows 0–1, black on rows 6–7). `getCell(position)` returns the cell at a position. |
| `Cell` | A board square: its `Position` and the `Piece` on it (or `null`). |
| `Position` | Immutable row/column pair with `isValid()` bounds check; `toString()` renders algebraic notation (`a1`–`h8`). |
| `Move` | Records the `Player`, start and end `Cell`, the piece moved and the piece captured (if any). |
| `Player` | Player name and `PieceColour` side. |

### Pieces (`pieces/`)

| Class | Movement rule implemented in `isValidMove` |
|-------|--------------------------------------------|
| `Piece` | Abstract base class. Stores colour, `PieceType` and captured flag. Provides `isPathClear()` for sliding pieces; `toString()` prints an uppercase symbol for white and lowercase for black (`K`, `Q`, `R`, `B`, `N`, `P`). |
| `King` | One square in any direction. |
| `Queen` | Any distance horizontally, vertically or diagonally, path must be clear. |
| `Rook` | Any distance horizontally or vertically, path must be clear. |
| `Bishop` | Any distance diagonally, path must be clear. |
| `Knight` | L-shape (2+1); can jump over pieces. |
| `Pawn` | One square forward, two squares forward on its first move, or one square diagonally to capture. |

All pieces reject moves onto a square occupied by a piece of the same colour.

### Enums (`enums/`)

| Enum | Values |
|------|--------|
| `GameStatus` | `IN_PROGRESS`, `BLACK_WIN`, `WHITE_WIN`, `DRAW`, `RESIGNATION` |
| `PieceColour` | `WHITE`, `BLACK` |
| `PieceType` | `KING`, `QUEEN`, `BISHOP`, `KNIGHT`, `ROOK`, `PAWN` |

## Design Notes

- **Polymorphism / Template:** each concrete `Piece` overrides `isValidMove`, so `Game` validates moves without knowing the piece type.
- **Composition:** `Game` → `Board` → `Cell` → `Piece`.
- **Simplification:** the game ends when a `King` is captured; check, castling, en passant and promotion are not modelled.
