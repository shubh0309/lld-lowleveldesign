# Payment Gateway — Low Level Design

Payment gateway where users register, add payment instruments (bank account or card) and send money to each other. Each domain (user, instrument, transaction) follows a Controller → Service → Model layering with DOs (data objects) used at the API boundary.

**Package:** `com.conceptcoding.interviewquestions.paymentgateway`

## Entry Point

`PaymentGateway.main()` — adds two users (Alice, Bob), adds a bank instrument for Alice and a card instrument for Bob, makes a ₹500 payment from Alice's bank to Bob's card, then prints each user's instruments and transaction history.

Run `PaymentGateway` directly from your IDE.

## Folder Structure

```
paymentgateway/
├── PaymentGateway.java               # Entry point (main method)
├── user/
│   ├── UserController.java           # Add / get user
│   ├── UserService.java              # In-memory user store, User <-> UserDO mapping
│   ├── User.java                     # User model
│   └── UserDO.java                   # User data object
├── instrument/
│   ├── InstrumentController.java     # Add instrument, list/find a user's instruments
│   ├── InstrumentServiceFactory.java # Returns BankService or CardService
│   ├── InstrumentService.java        # Abstract service + shared userVsInstruments store
│   ├── BankService.java              # Bank instrument logic
│   ├── CardService.java              # Card instrument logic
│   ├── Instrument.java               # Abstract instrument model
│   ├── BankInstrument.java           # Account number + IFSC
│   ├── CardInstrument.java           # Card number + CVV
│   ├── InstrumentDO.java             # Instrument data object
│   └── InstrumentType.java           # BANK, CARD
└── transaction/
    ├── TransactionController.java    # Make payment, get history
    ├── TransactionService.java       # Loads instruments, calls Processor, records history
    ├── Processor.java                # Payment processing hook (stub)
    ├── Transaction.java              # Transaction model
    ├── TransactionDO.java            # Transaction data object
    └── TransactionStatus.java        # SUCCESS, DENIED, PENDING
```

## Classes

### User (`user/`)

| Class | Responsibility |
|-------|----------------|
| `UserController` | `addUser(UserDO)` and `getUser(id)` via `UserService`. |
| `UserService` | Static in-memory list of `User`s; assigns a random ID on add and maps `User` ↔ `UserDO`. |
| `User` / `UserDO` | User ID, name and email (model / data object). |

### Instrument (`instrument/`)

| Class | Responsibility |
|-------|----------------|
| `InstrumentController` | `addInstrument()` picks the service by type via the factory; `getAllInstruments(userId)` combines bank and card instruments; `getInstrumentByID()`. |
| `InstrumentServiceFactory` | **Factory** returning `BankService` or `CardService` for an `InstrumentType`. |
| `InstrumentService` | Abstract: `addInstrument()`, `getInstrumentsByUserID()`; shared static `Map<userId, List<Instrument>>`. |
| `BankService` / `CardService` | Create `BankInstrument` / `CardInstrument` with a random ID, store it under the user, and map to `InstrumentDO`. |
| `Instrument` | Abstract model: instrument ID, user ID, type. |
| `BankInstrument` | Bank account number and IFSC code. |
| `CardInstrument` | Card number and CVV. |
| `InstrumentDO` | Flat data object carrying fields for either instrument type. |
| `InstrumentType` | Enum: `BANK`, `CARD`. |

### Transaction (`transaction/`)

| Class | Responsibility |
|-------|----------------|
| `TransactionController` | `makePayment(TransactionDO)` and `getTransactionHistory(userId)`. |
| `TransactionService` | Loads sender/receiver instruments, calls `Processor`, creates a `Transaction` (status currently set to `SUCCESS`) and adds it to both users' history. |
| `Processor` | Placeholder for validating instruments and debiting/crediting. |
| `Transaction` / `TransactionDO` | Transaction ID, amount, sender/receiver IDs, debit/credit instrument IDs and status. |
| `TransactionStatus` | Enum: `SUCCESS`, `DENIED`, `PENDING`. |

## Design Notes

- **Factory pattern:** `InstrumentServiceFactory` selects the instrument service by type.
- **Controller → Service → Model** layering per domain, with DOs decoupling API data from internal models.
- **Storage** is in-memory static maps/lists.
