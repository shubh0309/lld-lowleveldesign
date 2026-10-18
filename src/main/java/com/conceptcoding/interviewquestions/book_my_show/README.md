# BookMyShow — Low Level Design

Movie ticket booking system. Users pick a city and date, browse movies, theatres and show times, select seats and book them. Seat booking is made safe for concurrent users with per-seat locks.

**Package:** `com.conceptcoding.interviewquestions.book_my_show`

## Entry Point

`BookMyShowApp.main()` — `initialize()` creates two movies (BAAHUBALI, AVENGERS), an INOX theatre in Bangalore and a PVR theatre in Delhi (one 20-seat screen each) and four shows. `userFlow()` then walks a user through: select city → list movies for today → pick a movie → list theatres → pick a theatre → list shows → pick a show → book seats 1, 2, 3, printing the booking ID.

Run `BookMyShowApp` directly from your IDE.

## Folder Structure

```
book_my_show/
├── BookMyShowApp.java            # Entry point (main method), seeds data and runs user flow
├── controllers/
│   ├── TheatreController.java    # Add theatres, browse movies/theatres/shows
│   └── BookingController.java    # Create and fetch bookings
├── service/
│   ├── TheatreService.java       # Theatres by city; movie/theatre/show lookups
│   └── BookingService.java       # Lock seats, simulate payment, confirm booking
├── entities/
│   ├── Movie.java
│   ├── Theatre.java              # Name, city, screens
│   ├── Screen.java               # Seats and shows grouped by date
│   ├── Seat.java                 # Seat id and category
│   ├── Show.java                 # Movie, date, time, seat status + per-seat locks
│   ├── Booking.java              # Booking id, user, show, seats, payment
│   ├── Payment.java              # Payment id and status
│   └── User.java
└── enums/
    ├── City.java                 # BANGALORE, DELHI
    ├── SeatCategory.java         # SILVER, GOLD, PLATINUM
    ├── SeatStatus.java           # AVAILABLE, LOCKED, BOOKED
    └── PaymentStatus.java        # SUCCESS, FAILED
```

## Classes

### Controllers (`controllers/`)

| Class | Responsibility |
|-------|----------------|
| `TheatreController` | Delegates to `TheatreService`: `addTheatre()`, `getMovies(city, date)`, `getTheatres(city, movie, date)`, `getShows(movie, date, theatre)`. |
| `BookingController` | Delegates to `BookingService`: `createBooking(user, show, seats)`, `getBooking(id)`, `getBookingsForUser(user)`. |

### Services (`service/`)

| Class | Responsibility |
|-------|----------------|
| `TheatreService` | Stores `Map<City, List<Theatre>>`; finds movies running in a city on a date, theatres showing a movie, and shows for a movie in a theatre. |
| `BookingService` | `book()` locks the seats on the `Show` (fails with "Seat unavailable"), simulates a successful `Payment`, confirms seats and stores the `Booking`; releases seats if payment fails. |

### Entities (`entities/`)

| Class | Responsibility |
|-------|----------------|
| `Movie` | Movie name. |
| `Theatre` | Name, `City` and list of `Screen`s. |
| `Screen` | Screen ID, `Seat`s and `Map<LocalDate, List<Show>>`; `addShow()`, `getShows(date)`. |
| `Seat` | Seat ID and `SeatCategory`. |
| `Show` | Movie, date, start time, `seatStatusMap` and a `ReentrantLock` per seat. `lockSeats()` acquires locks in sorted order (avoids deadlock), checks all are `AVAILABLE`, marks them `LOCKED`. `confirmSeats()` → `BOOKED`, `releaseSeats()` → `AVAILABLE`. |
| `Booking` | Random UUID, `User`, `Show`, seat IDs and `Payment`. |
| `Payment` | Random UUID and `PaymentStatus`. |
| `User` | User ID and name. |

### Enums (`enums/`)

| Enum | Values |
|------|--------|
| `City` | `BANGALORE`, `DELHI` |
| `SeatCategory` | `SILVER`, `GOLD`, `PLATINUM` |
| `SeatStatus` | `AVAILABLE`, `LOCKED`, `BOOKED` |
| `PaymentStatus` | `SUCCESS`, `FAILED` |

## Design Notes

- **Layered design:** App → Controllers → Services → Entities.
- **Concurrency:** seat status is per `Show`, guarded by per-seat locks acquired in sorted order to prevent deadlocks.
- **Seat lifecycle:** `AVAILABLE → LOCKED → BOOKED`, or back to `AVAILABLE` on payment failure.
