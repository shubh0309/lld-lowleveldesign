# Cricbuzz — Low Level Design

Cricket match simulator and scorecard system. A match between two teams is played inning by inning, over by over and ball by ball with random outcomes; batting and bowling scorecards are updated by observers after every ball.

**Package:** `com.conceptcoding.interviewquestions.cricbuzz`

## Entry Point

`Demo.main()` — creates two teams (India, SriLanka) of 11 all-rounders, with players 8–11 as bowlers, and starts a `T20Match` at "SMS STADIUM". Prints batting and bowling scorecards for each inning and the winner.

Run `Demo` directly from your IDE. Output differs on every run since toss, runs and wickets are random.

## Folder Structure

```
cricbuzz/
├── Demo.java                          # Entry point (main method), builds teams
├── Match.java                         # Toss, two innings, winner
├── MatchType.java                     # Interface: overs per inning, max overs per bowler
├── T20Match.java                      # 20 overs, 5 per bowler
├── OneDayMatch.java                   # 50 overs, 10 per bowler
├── Inning/
│   ├── InningDetails.java             # Plays all overs of an inning
│   ├── OverDetails.java               # Plays 6 legal balls
│   ├── BallDetails.java               # Simulates one delivery, notifies observers
│   ├── BallType.java                  # NORMAL, WIDEBALL, NOBALL
│   └── RunType.java                   # ZERO, ONE, TWO, THREE, FOUR, SIX
├── ScoreUpdater/
│   ├── ScoreUpdaterObserver.java      # Observer interface
│   ├── BattingScoreUpdater.java       # Updates batsman scorecard
│   └── BowlingScoreUpdater.java       # Updates bowler scorecard
└── Team/
    ├── Team.java                      # Playing 11, bench, batting/bowling controllers
    ├── Wicket.java                    # Wicket type, bowler, over and ball
    ├── WicketType.java                # RUNOUT, BOLD, CATCH
    └── Player/
        ├── Person.java                # Name, age, address
        ├── PlayerDetails.java         # Person, type, batting/bowling scorecards
        ├── PlayerType.java            # BATSMAN, BOWLER, WICKETKEEPER, CAPTAIN, ALLROUNDER
        ├── PlayerBattingController.java # Batting order, striker / non-striker
        ├── PlayerBowlingController.java # Bowler rotation and over limits
        └── Score/
            ├── BattingScoreCard.java  # Runs, balls, 4s, 6s, wicket
            └── BowlingScoreCard.java  # Overs, runs given, wickets, extras
```

## Classes

### Match

| Class | Responsibility |
|-------|----------------|
| `Match` | Two teams, date, venue, `MatchType` and two `InningDetails`. `startMatch()` does a random toss (toss winner bats first), plays both innings (second inning chases the first's total), prints scorecards and the winner. |
| `MatchType` | Interface: `noOfOvers()`, `maxOverCountBowlers()`. |
| `T20Match` / `OneDayMatch` | 20 overs / 5 per bowler and 50 overs / 10 per bowler. |

### Inning (`Inning/`)

| Class | Responsibility |
|-------|----------------|
| `InningDetails` | Sets opening batsmen, then for each over picks the next bowler, plays an `OverDetails`, stops when the target is reached or batsmen run out, and swaps strike at over end. |
| `OverDetails` | Plays balls until 6 legal deliveries; brings in the next batsman on a wicket and flags the batting team as winner when the target is reached. Counts extras. |
| `BallDetails` | Simulates one delivery: random wicket (20%, `BOLD`) or random runs, rotates strike on odd runs, then notifies `BattingScoreUpdater` and `BowlingScoreUpdater`. |
| `BallType` / `RunType` | Enums: `NORMAL`, `WIDEBALL`, `NOBALL` / `ZERO`, `ONE`, `TWO`, `THREE`, `FOUR`, `SIX`. |

### Score Updaters (`ScoreUpdater/`)

| Class | Responsibility |
|-------|----------------|
| `ScoreUpdaterObserver` | Interface: `update(ballDetails)`. |
| `BattingScoreUpdater` | Adds runs, balls faced, 4s/6s and wicket details to the striker's `BattingScoreCard`. |
| `BowlingScoreUpdater` | Adds overs, runs conceded, wickets, no-balls and wides to the bowler's `BowlingScoreCard`. |

### Team (`Team/`)

| Class | Responsibility |
|-------|----------------|
| `Team` | Name, playing 11, bench, `PlayerBattingController`, `PlayerBowlingController` and winner flag; prints scorecards and totals runs. |
| `Wicket` | `WicketType`, bowler who took it, and the over/ball it fell on. |
| `WicketType` | Enum: `RUNOUT`, `BOLD`, `CATCH`. |
| `Person` | Name, age, address. |
| `PlayerDetails` | `Person`, `PlayerType`, `BattingScoreCard` and `BowlingScoreCard`; prints both scorecards. |
| `PlayerType` | Enum: `BATSMAN`, `BOWLER`, `WICKETKEEPER`, `CAPTAIN`, `ALLROUNDER`. |
| `PlayerBattingController` | Queue of batsmen yet to bat plus striker / non-striker; throws when all out. |
| `PlayerBowlingController` | Rotates bowlers in a deque, tracking overs bowled against the per-bowler limit. |
| `BattingScoreCard` | Runs, balls played, 4s, 6s, strike rate, wicket details. |
| `BowlingScoreCard` | Overs, runs given, wickets, no-balls, wides, economy rate. |

## Design Notes

- **Observer pattern:** each `BallDetails` notifies score updaters to refresh batting and bowling scorecards.
- **Strategy-like `MatchType`:** match format (T20 / ODI) is pluggable.
- **Hierarchy:** `Match` → `InningDetails` → `OverDetails` → `BallDetails`.
