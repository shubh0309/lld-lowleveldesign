# Car Rental System — Low Level Design

Vehicle rental system with multiple stores. Users search a store's inventory for vehicles available over a date range, reserve one, start and end the trip, then get billed and pay. Billing and payment are pluggable strategies, and reservations are made safe with per-vehicle locks.

**Package:** `com.conceptcoding.interviewquestions.carrental`

## Entry Point

`Demo.main()` — creates a `VehicleRentalSystem` with one store (ID 1001), two users and two four-wheelers (₹1100/day and ₹1400/day). Then searches available vehicles for 5–7 Dec 2025, reserves vehicle 1 (`DAILY`), starts the trip, submits the vehicle, generates a bill with `DailyBillingStrategy` and pays with `UPIPaymentStrategy`, printing the receipt.

Run `Demo` directly from your IDE.

## Folder Structure

```
carrental/
├── Demo.java                            # Entry point (main method)
├── VehicleRentalSystem.java             # Holds stores and users
├── Store.java                           # Facade per store: search, reserve, trip, bill, pay
├── Location.java                        # Store address
├── User.java                            # User id, name, driving licence
├── product/
│   ├── Vehicle.java                     # Vehicle id, number, type, daily cost, status
│   ├── VehicleInventoryManager.java     # Store inventory, availability, atomic reserve/release
│   ├── DateInterval.java                # From/to dates with overlap check
│   ├── VehicleType.java                 # FOUR_WHEELER, TWO_WHEELER
│   └── VehicleStatus.java               # AVAILABLE, BOOKED, MAINTENANCE
├── reservation/
│   ├── Reservation.java                 # Reservation details and status
│   ├── ReservationManager.java          # Create, cancel, start trip, submit vehicle
│   ├── ReservationRepository.java       # In-memory reservation store
│   ├── ReservationStatus.java           # SCHEDULED, IN_USE, COMPLETED, CANCELLED
│   └── ReservationType.java             # HOURLY, DAILY
├── Bill/
│   ├── Bill.java                        # Bill id, reservation id, amount, paid flag
│   ├── BillManager.java                 # Generates and stores bills via a strategy
│   ├── BillingStrategy.java             # Strategy interface
│   └── DailyBillingStrategy.java        # days × daily rental cost
└── payment/
    ├── Payment.java                     # Payment id, bill id, amount, mode, date
    ├── PaymentManager.java              # Processes and stores payments via a strategy
    ├── PaymentStrategy.java             # Strategy interface
    ├── UPIPaymentStrategy.java          # UPI payment, marks bill paid
    └── PaymentMode.java                 # CASH, ONLINE, UPI
```

## Classes

### Core

| Class | Responsibility |
|-------|----------------|
| `VehicleRentalSystem` | Lists of `Store`s and `User`s with add/get/remove. |
| `Store` | Store ID, `Location` and its own `VehicleInventoryManager`, `ReservationManager`, `BillManager` and `PaymentManager`. Exposes `getVehicles()`, `createReservation()`, `cancelReservation()`, `startTrip()`, `submitVehicle()`, `generateBill(reservationId, strategy)` and `makePayment(bill, strategy, amount)` (removes the reservation once paid). |
| `Location` | Building number, area, city, state, country, pincode. |
| `User` | User ID, name and driving licence number. |

### Product (`product/`)

| Class | Responsibility |
|-------|----------------|
| `Vehicle` | ID, number, `VehicleType`, daily rental cost and `VehicleStatus`. |
| `VehicleInventoryManager` | Stores vehicles, their reservation IDs and a `ReentrantLock` per vehicle. `isAvailable()` rejects vehicles in maintenance or with overlapping reservations; `reserve()` / `release()` run under the vehicle lock; `getAvailableVehicles(type, from, to)` searches. |
| `DateInterval` | Validated date range with `overlaps()`. |
| `VehicleType` / `VehicleStatus` | Enums: `FOUR_WHEELER`, `TWO_WHEELER` / `AVAILABLE`, `BOOKED`, `MAINTENANCE`. |

### Reservation (`reservation/`)

| Class | Responsibility |
|-------|----------------|
| `Reservation` | ID, vehicle ID, user ID, from/to dates, `ReservationType` and `ReservationStatus` (starts `SCHEDULED`). |
| `ReservationManager` | Generates IDs, reserves the vehicle in inventory and saves the reservation; `cancelReservation()` (→ `CANCELLED`, releases vehicle), `startTrip()` (→ `IN_USE`), `submitVehicle()` (→ `COMPLETED`, releases vehicle). |
| `ReservationRepository` | `ConcurrentHashMap` store: `save()`, `findById()`, `remove()`. |
| `ReservationStatus` / `ReservationType` | Enums: `SCHEDULED`, `IN_USE`, `COMPLETED`, `CANCELLED` / `HOURLY`, `DAILY`. |

### Billing (`Bill/`)

| Class | Responsibility |
|-------|----------------|
| `Bill` | Bill ID, reservation ID, total amount, paid flag. |
| `BillManager` | Generates bills with the current `BillingStrategy` and stores them. |
| `BillingStrategy` | Interface: `generateBill(reservation)`. |
| `DailyBillingStrategy` | Charges (days booked, inclusive) × vehicle daily rental cost. |

### Payment (`payment/`)

| Class | Responsibility |
|-------|----------------|
| `Payment` | Payment ID, bill ID, amount paid, `PaymentMode`, date. |
| `PaymentManager` | Processes payments with the current `PaymentStrategy` and stores them; lookup by bill or payment ID. |
| `PaymentStrategy` | Interface: `processPayment(bill, amount)`. |
| `UPIPaymentStrategy` | Creates a UPI `Payment` and marks the bill as paid. |
| `PaymentMode` | Enum: `CASH`, `ONLINE`, `UPI`. |

## Design Notes

- **Strategy pattern:** billing and payment strategies are swappable per call.
- **Facade:** `Store` hides inventory, reservation, billing and payment managers.
- **Concurrency:** per-vehicle `ReentrantLock` makes availability-check + reserve atomic; repositories use concurrent maps.
