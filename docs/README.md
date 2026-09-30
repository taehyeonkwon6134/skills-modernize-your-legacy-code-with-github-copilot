# COBOL Account Management

This directory documents the COBOL account management example in `src/cobol/`.
The current implementation manages one in-memory balance; it does not store
student records, identify account holders, or support multiple accounts.

## Source Files

### `main.cob` - `MainProgram`

The program entry point displays the account menu, reads the user's selection,
and dispatches account operations to `Operations`.

Key behavior:

- Option 1 requests and displays the current balance.
- Option 2 requests a credit amount.
- Option 3 requests a debit amount.
- Option 4 exits the menu loop.
- Other selections display an invalid-choice message.

### `operations.cob` - `Operations`

Processes the requested account operation and interacts with `DataProgram` to
read or update the balance.

Key behavior:

- `TOTAL` reads and displays the current balance.
- `CREDIT` accepts an amount, adds it to the balance, and writes the result.
- `DEBIT` accepts an amount and subtracts it only when the balance is
  sufficient; otherwise, it displays an insufficient-funds message.

### `data.cob` - `DataProgram`

Provides the balance storage interface used by `Operations`. A `READ` request
copies the stored balance to the caller, and a `WRITE` request replaces the
stored balance. The balance is held in COBOL working storage, not in a file or
database.

## Account Rules and Limits

- The single balance is initialized to `1000.00` when the program starts.
- A debit is rejected when its requested amount exceeds the available balance;
  rejected debits do not update the stored balance.
- Credits are added directly. The program does not explicitly reject zero or
  negative credit amounts.
- The amount and balance fields use `PIC 9(6)V99`: six integer digits and two
  implied decimal digits. The source does not include explicit validation for
  out-of-range input or arithmetic overflow.
- There is no student ID, student profile, per-student balance, transaction
  history, or persistent storage in the current implementation. These would
  need to be added before the program could enforce student-specific account
  rules.

## Application Data Flow

```mermaid
sequenceDiagram
  actor User
  participant Main as MainProgram
  participant Ops as Operations
  participant Data as DataProgram

  loop Until the user selects Exit
    Main->>User: Display account menu
    User->>Main: Enter menu choice
    alt View balance
      Main->>Ops: CALL Operations(TOTAL)
      Ops->>Data: CALL DataProgram(READ, balance)
      Data-->>Ops: Copy stored balance to balance argument
      Ops-->>User: Display current balance
    else Credit account
      Main->>Ops: CALL Operations(CREDIT)
      Ops->>User: Prompt for credit amount
      User-->>Ops: Enter amount
      Ops->>Data: CALL DataProgram(READ, balance)
      Data-->>Ops: Copy stored balance to balance argument
      Ops->>Ops: Add amount to balance
      Ops->>Data: CALL DataProgram(WRITE, updated balance)
      Data-->>Ops: Store updated balance
      Ops-->>User: Display new balance
    else Debit account
      Main->>Ops: CALL Operations(DEBIT)
      Ops->>User: Prompt for debit amount
      User-->>Ops: Enter amount
      Ops->>Data: CALL DataProgram(READ, balance)
      Data-->>Ops: Copy stored balance to balance argument
      alt Sufficient funds
        Ops->>Ops: Subtract amount from balance
        Ops->>Data: CALL DataProgram(WRITE, updated balance)
        Data-->>Ops: Store updated balance
        Ops-->>User: Display new balance
      else Insufficient funds
        Ops-->>User: Display insufficient-funds message
      end
    else Exit
      Main->>Main: Set continue flag to NO
    else Invalid choice
      Main-->>User: Display invalid-choice message
    end
  end
  Main-->>User: Display goodbye message
```
