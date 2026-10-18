# Splitwise — Low Level Design

Expense-sharing app where users form groups, add expenses split between members (equal, unequal or percentage), and view per-user balance sheets showing who owes whom.

**Package:** `com.conceptcoding.interviewquestions.splitwise`

## Entry Point

`Demo.main()` — creates a `Splitwise` instance and calls `demo()`, which onboards three users, creates group `G1001`, adds a ₹900 breakfast split equally between all three (paid by `U1001`) and a ₹500 lunch split unequally (paid by `U2001`), then prints every user's balance sheet.

Run `Demo` directly from your IDE.

## Folder Structure

```
splitwise/
├── Demo.java                        # Entry point (main method)
├── Splitwise.java                   # App facade: controllers + demo flow
├── BalanceSheetController.java      # Updates and prints user balance sheets
├── UserExpenseBalanceSheet.java     # Per-user totals and balance with each other user
├── Balance.java                     # Amount owed / amount to get back
├── User/
│   ├── User.java                    # User id, name and balance sheet
│   └── UserController.java          # Add/find users
├── Group/
│   ├── Group.java                   # Members and expenses of a group
│   └── GroupController.java         # Create/find groups
└── Expense/
    ├── Expense.java                 # Expense details and splits
    ├── ExpenseController.java       # Validates split, creates expense, updates balances
    ├── ExpenseSplitType.java        # EQUAL, UNEQUAL, PERCENTAGE
    ├── SplitFactory.java            # Returns the ExpenseSplit validator for a type
    └── Split/
        ├── Split.java               # A user's share of an expense
        ├── ExpenseSplit.java        # Validator interface
        ├── EqualExpenseSplit.java
        ├── UnequalExpenseSplit.java
        └── PercentageExpenseSplit.java
```

## Classes

### Core

| Class | Responsibility |
|-------|----------------|
| `Demo` | Entry point; runs `Splitwise.demo()`. |
| `Splitwise` | Holds `UserController`, `GroupController` and `BalanceSheetController`; `demo()` sets up users, a group and expenses, then prints balances. |
| `BalanceSheetController` | `updateUserExpenseBalanceSheet()` updates the payer's total payment and get-back amounts and each participant's owed amount, including pairwise `Balance`s. `showBalanceSheetOfUser()` prints a user's sheet. |
| `UserExpenseBalanceSheet` | Per user: `totalYourExpense`, `totalPayment`, `totalYouOwe`, `totalYouGetBack` and `Map<userId, Balance>`. |
| `Balance` | `amountOwe` and `amountGetBack` with one other user. |

### Users (`User/`)

| Class | Responsibility |
|-------|----------------|
| `User` | `userId`, `userName` and a `UserExpenseBalanceSheet`. |
| `UserController` | Stores users; `addUser()`, `getUser(id)`, `getAllUsers()`. |

### Groups (`Group/`)

| Class | Responsibility |
|-------|----------------|
| `Group` | Group ID, name, members and expenses; `createExpense()` delegates to its `ExpenseController`. |
| `GroupController` | `createNewGroup()` (creator is added as first member) and `getGroup(id)`. |

### Expenses (`Expense/`)

| Class | Responsibility |
|-------|----------------|
| `Expense` | ID, description, amount, paid-by user, `ExpenseSplitType` and list of `Split`s. |
| `ExpenseController` | Gets a validator from `SplitFactory`, validates the splits, creates the `Expense` and updates balance sheets. |
| `ExpenseSplitType` | Enum: `EQUAL`, `UNEQUAL`, `PERCENTAGE`. |
| `SplitFactory` | **Factory** returning `EqualExpenseSplit`, `UnequalExpenseSplit` or `PercentageExpenseSplit`. |
| `Split` | A `User` and the amount they owe. |
| `ExpenseSplit` | Interface: `validateSplitRequest(splits, totalAmount)`. |
| `EqualExpenseSplit` | Checks every share equals `total / participants`. |
| `UnequalExpenseSplit` / `PercentageExpenseSplit` | Validation placeholders (not yet implemented). |

## Design Notes

- **Factory + Strategy:** `SplitFactory` picks the `ExpenseSplit` validator for each split type.
- **Controllers** separate user, group, expense and balance-sheet responsibilities.
