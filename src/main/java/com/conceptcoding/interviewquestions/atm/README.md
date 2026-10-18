# ATM — Low Level Design

ATM machine that lets a user insert a card, authenticate with a PIN, and either withdraw cash or check their balance. ATM behaviour is modelled with the **State** pattern and note dispensing with **Chain of Responsibility**.

**Package:** `com.conceptcoding.interviewquestions.atm`

## Entry Point

`ATMRoomComponents/ATMRoom.main()` — loads the ATM with ₹3500 (1 × ₹2000, 2 × ₹500, 5 × ₹100), creates a user whose bank account holds ₹3000, then walks through insert card → enter PIN → select `CASH_WITHDRAWAL` → withdraw ₹2700, printing the ATM status before and after.

Run `ATMRoom` directly from your IDE.

## Folder Structure

```
atm/
├── ATMRoomComponents/
│   ├── ATMRoom.java                      # Entry point (main method), sets up ATM and user
│   ├── ATM.java                          # Singleton ATM: current state, balance, note counts
│   ├── Card.java                         # Card details, PIN check, linked bank account
│   ├── User.java                         # User holding a Card
│   └── UserBankAccount.java              # Account balance and withdrawal
├── ATMStates/
│   ├── ATMState.java                     # Abstract state with default "something went wrong" ops
│   ├── IdleState.java                    # Waiting for card
│   ├── HasCardState.java                 # Card inserted, waiting for PIN
│   ├── SelectOperationState.java         # PIN verified, choose transaction
│   ├── CashWithdrawalState.java          # Withdraw cash
│   └── CheckBalanceState.java            # Show account balance
├── AmountWithdrawal/
│   ├── CashWithdrawProcessor.java        # Abstract handler in the chain
│   ├── TwoThousandWithdrawProcessor.java # Dispenses ₹2000 notes
│   ├── FiveHundredWithdrawProcessor.java # Dispenses ₹500 notes
│   └── OneHundredWithdrawProcessor.java  # Dispenses ₹100 notes
└── enums/
    └── TransactionType.java              # CASH_WITHDRAWAL, BALANCE_CHECK
```

## Classes

### ATM Room Components (`ATMRoomComponents/`)

| Class | Responsibility |
|-------|----------------|
| `ATMRoom` | Entry point. Initializes the `ATM` and a `User` with a `Card` and `UserBankAccount`, then drives the ATM through its states. |
| `ATM` | Eager-initialized **Singleton** (`getATMObject()` resets it to `IdleState`). Holds the current `ATMState`, total balance and count of ₹2000/₹500/₹100 notes, with methods to deduct each. |
| `Card` | Card details and linked `UserBankAccount`. `isCorrectPINEntered()` validates the PIN (hard-coded `112211`); `getBankBalance()` / `deductBankBalance()` delegate to the account. |
| `User` | Holds the user's `Card`. |
| `UserBankAccount` | Account `balance` and `withdrawalBalance()`. |

### ATM States (`ATMStates/`)

| Class | Responsibility |
|-------|----------------|
| `ATMState` | Abstract base defining every ATM operation (`insertCard`, `authenticatePin`, `selectOperation`, `cashWithdrawal`, `displayBalance`, `returnCard`, `exit`). Defaults print an error so each state only overrides what it allows. |
| `IdleState` | `insertCard()` → `HasCardState`. |
| `HasCardState` | `authenticatePin()` → `SelectOperationState` on correct PIN, otherwise ejects card and returns to `IdleState`. |
| `SelectOperationState` | Lists `TransactionType`s; `selectOperation()` → `CashWithdrawalState` or `CheckBalanceState`. |
| `CashWithdrawalState` | Checks ATM and bank balance, deducts both, dispenses notes via the processor chain, then exits to `IdleState`. |
| `CheckBalanceState` | `displayBalance()` prints the account balance, then exits to `IdleState`. |

### Amount Withdrawal (`AmountWithdrawal/`)

| Class | Responsibility |
|-------|----------------|
| `CashWithdrawProcessor` | Abstract handler holding the next processor; `withdraw()` forwards the remaining amount down the chain. |
| `TwoThousandWithdrawProcessor` | Dispenses as many ₹2000 notes as possible and passes on the remainder. |
| `FiveHundredWithdrawProcessor` | Dispenses ₹500 notes and passes on the remainder. |
| `OneHundredWithdrawProcessor` | Dispenses ₹100 notes; reports an error if an amount still remains. |

### Enums (`enums/`)

| Enum | Values |
|------|--------|
| `TransactionType` | `CASH_WITHDRAWAL`, `BALANCE_CHECK` (with `showAllTransactionTypes()`) |

## Design Notes

- **State pattern:** `ATM` delegates every action to its current `ATMState`; each state moves the ATM to the next one.
- **Chain of Responsibility:** `TwoThousand → FiveHundred → OneHundred` processors split the withdrawal into notes.
- **Singleton:** a single `ATM` instance via `ATM.getATMObject()`.
