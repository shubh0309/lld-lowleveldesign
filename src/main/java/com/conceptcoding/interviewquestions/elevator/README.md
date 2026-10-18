# Elevator System — Low Level Design

Multi-elevator system for a building. External floor buttons go through a dispatcher and a pluggable selection strategy to pick an elevator; internal buttons go straight to that elevator's controller. Each controller runs on its own thread and serves requests using up/down priority queues.

**Package:** `com.conceptcoding.interviewquestions.elevator`

## Entry Point

`Demo.main()` — creates two elevator cars with controllers, an `ElevatorScheduler` using `NearestElevatorStrategy`, an `ExternalDispatcher` and a 5-floor `Building`. Starts both controller threads, then submits a mix of external (floor UP/DOWN) and internal (floor number) requests.

Run `Demo` directly from your IDE. Controller threads loop forever, so stop the program manually once requests are served.

## Folder Structure

```
elevator/
├── Demo.java                        # Entry point (main method)
├── Building.java                    # List of floors
├── Floor.java                       # Floor number + UP/DOWN external buttons
├── ExternalButton.java              # Button on a floor; sends request to ExternalDispatcher
├── ExternalDispatcher.java          # Asks scheduler for an elevator, forwards request
├── InternalButton.java              # Button inside a car; sends request to InternalDispatcher
├── InternalDispatcher.java          # Singleton; forwards request to the car's controller
├── ElevatorScheduler.java           # Picks a controller using a selection strategy
├── ElevatorSelectionStrategy.java   # Strategy interface
├── NearestElevatorStrategy.java     # Same-direction nearest, else idle, else first
├── LeastBusyStrategy.java           # Fewest pending requests
├── ElevatorController.java          # Runnable; up/down priority queues, drives the car
├── ElevatorCar.java                 # Current floor, direction, door; moves floor by floor
├── Door.java                        # Opens/closes the car door
└── enums/
    ├── DoorState.java               # DOOR_OPEN, DOOR_CLOSED
    └── ElevatorDirection.java       # UP, DOWN, IDLE
```

## Classes

### Building & Buttons

| Class | Responsibility |
|-------|----------------|
| `Building` | Creates `Floor`s 1..N sharing one `ExternalDispatcher`; `getFloor(n)` returns a floor. |
| `Floor` | Floor number with an up and down `ExternalButton`; `pressUpButton()` / `pressDownButton()`. |
| `ExternalButton` | Sends `(floor, direction)` to the `ExternalDispatcher`. |
| `InternalButton` | Bound to one `ElevatorController`; `pressButton(floor)` sends the request via `InternalDispatcher`. |

### Dispatching & Scheduling

| Class | Responsibility |
|-------|----------------|
| `ExternalDispatcher` | Uses `ElevatorScheduler.assignElevator()` to choose a controller, then calls `submitRequest(floor)` on it. |
| `InternalDispatcher` | Eager **Singleton**; forwards the destination floor to the given controller. |
| `ElevatorScheduler` | Holds all controllers and an `ElevatorSelectionStrategy` (swappable via `setStrategy()`). |
| `ElevatorSelectionStrategy` | Interface: `selectElevator(controllers, requestFloor, direction)`. |
| `NearestElevatorStrategy` | Picks the closest elevator moving in the requested direction that hasn't passed the floor; falls back to an idle elevator, then the first one. |
| `LeastBusyStrategy` | Picks the elevator with the fewest queued requests. |

### Elevator

| Class | Responsibility |
|-------|----------------|
| `ElevatorController` | `Runnable` per car. Queues requests in a min-heap (`upMinPQ`) or max-heap (`downMaxPQ`) relative to the car's next stop; waits on a monitor when idle and serves all up requests, then all down requests. |
| `ElevatorCar` | Car `id`, `currentFloor`, `nextFloorStoppage`, `movingDirection` and `Door`. `moveElevator()` closes the door, moves floor by floor printing its display, then opens the door. |
| `Door` | Tracks `DoorState`; `openDoor()` / `closeDoor()`. |

### Enums (`enums/`)

| Enum | Values |
|------|--------|
| `DoorState` | `DOOR_OPEN`, `DOOR_CLOSED` |
| `ElevatorDirection` | `UP`, `DOWN`, `IDLE` |

## Design Notes

- **Strategy pattern:** elevator selection is pluggable (`NearestElevatorStrategy`, `LeastBusyStrategy`).
- **Singleton:** `InternalDispatcher`.
- **Concurrency:** each `ElevatorController` runs on its own thread using `PriorityBlockingQueue`s and `wait`/`notify`.
- **Separation of concerns:** buttons → dispatcher → scheduler → controller → car.
