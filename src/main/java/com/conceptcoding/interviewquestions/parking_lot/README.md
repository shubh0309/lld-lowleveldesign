# Parking Lot — Low Level Design

Multi-level parking lot for two- and four-wheelers. Vehicles get a ticket at the entrance gate, each level has a thread-safe spot manager per vehicle type, and the exit gate computes the fee and takes payment before releasing the spot.

**Package:** `com.conceptcoding.interviewquestions.parking_lot`

## Entry Point

`ParkingLotClient.main()` — builds a 2-level `ParkingBuilding` (each level with two-wheeler and four-wheeler spot managers using `RandomLookupStrategy`), parks a bike and a car, then exits them paying with `CashPayment` and `UPIPayment` respectively.

Run `ParkingLotClient` directly from your IDE.

## Folder Structure

```
parking_lot/
├── ParkingLotClient.java                    # Entry point (main method)
├── Ticket.java                              # Vehicle, level, spot and entry time
├── Entity/
│   ├── ParkingSpot.java                     # Spot id and free/occupied flag
│   └── Vehicle.java                         # Vehicle number and type
├── enums/
│   └── VehicleType.java                     # TWO_WHEELER, FOUR_WHEELER
├── parkinglot/
│   ├── ParkingLot.java                      # Facade: building + entrance/exit gates
│   ├── ParkingBuilding.java                 # Levels; allocates and releases spots
│   ├── ParkingLevel.java                    # Spot managers per vehicle type
│   ├── EntranceGate.java                    # Issues tickets
│   └── ExitGate.java                        # Computes cost, takes payment, frees spot
├── spotManagers/
│   ├── ParkingSpotManager.java              # Abstract, lock-protected park/unpark
│   ├── TwoWheelerSpotManager.java
│   └── FourWheelerSpotManager.java
├── LookupStrategy/
│   ├── ParkingSpotLookupStrategy.java       # Strategy interface for spot selection
│   └── RandomLookupStrategy.java            # Returns the first free spot
├── pricing/
│   ├── PricingStrategy.java                 # Strategy interface for fees
│   ├── FixedPricingStrategy.java            # Flat fee of 100
│   └── CostComputation.java                 # Applies a PricingStrategy to a ticket
└── payment/
    ├── Payment.java                         # Payment interface
    ├── CashPayment.java
    └── UPIPayment.java
```

## Classes

### Core

| Class | Responsibility |
|-------|----------------|
| `ParkingLotClient` | Entry point; wires up levels, managers, gates and strategies and runs a park/exit flow. |
| `Ticket` | Issued on entry: `Vehicle`, `ParkingLevel`, `ParkingSpot` and `entryTime`. |

### Entities (`Entity/`, `enums/`)

| Class | Responsibility |
|-------|----------------|
| `ParkingSpot` | Spot ID with `occupySpot()`, `releaseSpot()` and `isSpotFree()`. |
| `Vehicle` | Vehicle number and `VehicleType`. |
| `VehicleType` | Enum: `TWO_WHEELER`, `FOUR_WHEELER`. |

### Parking Lot (`parkinglot/`)

| Class | Responsibility |
|-------|----------------|
| `ParkingLot` | Facade exposing `vehicleArrives(vehicle)` → `Ticket` and `vehicleExits(ticket, payment)`. |
| `EntranceGate` | `enter()` asks the building to allocate a spot and returns the ticket. |
| `ExitGate` | `completeExit()` computes the fee via `CostComputation`, charges the `Payment` (denies exit on failure) and releases the spot. |
| `ParkingBuilding` | Iterates levels to find one with a free spot for the vehicle type and creates the `Ticket`; throws "Parking Full" if none. `release()` frees the ticket's spot. |
| `ParkingLevel` | Level number and `Map<VehicleType, ParkingSpotManager>`; `hasAvailability()`, `park()`, `unPark()`. |

### Spot Managers (`spotManagers/`)

| Class | Responsibility |
|-------|----------------|
| `ParkingSpotManager` | Abstract; holds a list of spots and a `ParkingSpotLookupStrategy`. `park()`, `unPark()` and `hasFreeSpot()` are guarded by a fair `ReentrantLock`. |
| `TwoWheelerSpotManager` / `FourWheelerSpotManager` | Manage spots for their vehicle type, each with its own lock. |

### Strategies (`LookupStrategy/`, `pricing/`, `payment/`)

| Class | Responsibility |
|-------|----------------|
| `ParkingSpotLookupStrategy` | Interface: `selectSpot(spots)`. |
| `RandomLookupStrategy` | Returns the first free spot. |
| `PricingStrategy` | Interface: `calculate(ticket)`. |
| `FixedPricingStrategy` | Always returns 100. |
| `CostComputation` | Delegates fee calculation to its `PricingStrategy`. |
| `Payment` | Interface: `pay(amount)`. |
| `CashPayment` / `UPIPayment` | Print the payment and return success. |

## Design Notes

- **Strategy pattern:** spot lookup, pricing and payment are all pluggable.
- **Facade:** `ParkingLot` hides building, level and gate interactions.
- **Concurrency:** each spot manager has its own lock so different vehicle types don't block each other.
