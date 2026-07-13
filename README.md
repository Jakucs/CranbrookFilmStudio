# Cranbrook Film Studio — Prop Warehouse & Production Rental System

## Overview

Your task is to design a prop management system for a film studio warehouse operating as part of a production rental network.

---

## Exam Rules & Circumstances

- You have **1.5 hours** to complete this assignment.
- Follow proper git commit practices: commits must be **atomic and descriptive**. A single final commit is not acceptable.
- This is an **individual task**. Do not use AI tools or any external sources (except Google search for documentation and language reference).
- You must submit code you **fully understand** and can explain in detail.

---

## The Warehouse

Every prop in the warehouse is tagged with a **catalogue number**, issued sequentially starting from **2001**. No two props ever share a number.

Each prop tracks its current status:

- `AVAILABLE`
- `ON_LOAN`
- `RESERVED`

These exact labels appear on every warehouse tag and the booking terminal.

---

## Prop Categories

The warehouse holds four categories of prop:

| Category | Loan Length | Late Fee |
|---|---|---|
| **Costume items** | 5 days | €5.00 / day |
| **Set pieces** *(furniture, vehicles, large structures)* | 10 days | €8.00 / day |
| **Technical rigs** *(lighting frames, camera dollies, rigging hardware)* | 3 days | €15.00 / day |
| **Archival props** *(original period artefacts, heritage pieces, preservation items)* | ❌ never loaned | — |

---

## Prop Properties

Every prop has:

- A **catalogue number**
- A **title** (name/designation)
- A **size classification**: `COMPACT`, `STANDARD`, or `OVERSIZED`
- A **fragile flag** (boolean)
- A **current status**: `AVAILABLE`, `ON_LOAN`, or `RESERVED`

---

## Production Rules

- A production may hold at most **8 props** at any one time.
- A production whose unpaid charges exceed **€50.00** is **suspended** from further borrowing until the balance is cleared.
- Only **costume items** and **set pieces** may be booked in advance.
- **Technical rigs** and **archival props** may not be reserved.

---

## Task 1: The Prop Collection

Model the prop catalogue so that shared traits live in one place and differences live where they belong.

**Requirements:**

- Every prop has a catalogue number, title, size classification, fragile flag, and current status.
- Each loanable category has its own loan length and daily late fee, and calculates its overdue charge accordingly.
- Given a number of days overdue, a prop must report the charge it has accrued.
  - Example: a costume item 3 days late → **€15.00**; a set piece 3 days late → **€24.00**
- A prop returned on time or early owes **nothing**.
- An **archival prop** is part of the collection but is never loaned out. Asking it to calculate an overdue charge must **signal an error** rather than return zero.

> 💡 You may need to create additional types beyond what is described here. Think carefully about where shared behaviour belongs and how to avoid duplication.

---

## Task 2: Advance Bookings

Some props can be booked ahead by a production; some cannot. Design this so that the ability to be booked is a **capability** a prop either has or does not have, independently of its category.

**Requirements:**

- **Costume items** and **set pieces** support advance bookings.
- **Technical rigs** and **archival props** do not — attempting to book either must **signal an error**.
- A bookable prop can be:
  - reserved
  - queried whether it is currently reserved
  - have its reservation released
- When a prop is reserved, its status becomes `RESERVED`.
- Releasing a booking returns the prop's status to `AVAILABLE`.
- A prop that is already `ON_LOAN` or already `RESERVED` cannot be booked again — attempting to do so must **signal an error**.

---

## Task 3: The Prop Desk

This is where productions interact with the warehouse. The prop desk handles check-out and return.

### Borrowing

When a production borrows a prop, all handbook rules apply. Any of the following violations must **signal an error and leave the system unchanged**:

- The prop is not currently `AVAILABLE` (e.g. already `ON_LOAN`)
- The prop is an **archival prop**, which can never be borrowed
- The production already holds **8 props** and cannot take a ninth
- The production's unpaid charges exceed **€50.00** — suspended until cleared

A successful check-out:
- moves the prop to `ON_LOAN`
- records that the production now holds it

### Returning

When a prop is returned, the production has held it for some number of days:

- **Overdue days** = days held − loan length (never negative; on time = 0 overdue)
- If overdue: the charge is calculated and **added to the production's outstanding balance**
- The prop returns to `AVAILABLE` and is removed from the production's held items
- Returning a prop the production is **not actually holding** must signal an error

> 💡 A production is part of this system too. Decide for yourself what it needs to keep track of. Paying off charges is outside scope — you only need to track the balance and enforce the suspension.

---

## Task 4: Proving It Works

A rental system that quietly miscalculates a late charge or lets a suspended production walk off with a set piece is worse than no system at all. Prove that yours behaves.

### Required Tests

**Charge calculations:**
- A costume item returned several days late → correct balance
- A set piece returned several days late → correct balance
- A prop returned on time → owes nothing

**Check-out rule violations (each must be rejected, system unchanged):**
- An unavailable prop (already `ON_LOAN`)
- An archival prop
- The ninth prop over the 8-item limit
- A suspended production (balance > €50.00)

**Happy path:**
- A successful check-out followed by a successful return must move the prop between `AVAILABLE` and `ON_LOAN` as expected

---

## Technical Requirements

- **Java 21** — use modern features where appropriate
- **Maven** — configure dependencies yourself in `pom.xml`
- Proper **encapsulation** of all fields
- **Immutable** objects where they make sense
- **Null safety** throughout the codebase
- Clean, maintainable code

---

## Deliverables (GitHub repository)

1. All Java source files with functionality implemented
2. Maven `pom.xml` with properly configured dependencies
3. Test classes (JUnit 5)
4. All code must **compile and run without errors**
5. Do **not** include `target/` or `.idea/` folders (configure `.gitignore`)
