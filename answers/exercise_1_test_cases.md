# Exercise 1 — Concurrency-Sensitive Payment API Test Cases

## Happy Path

### TC-01: Send gift successfully with valid inputs

- **Category:** Happy Path
- **Priority:** P0

- **Preconditions:**
  - User is authenticated
  - User joined room successfully
  - Wallet balance is sufficient
  - Valid gift exists

- **Steps:**
  1. Send POST request to `/api/gifts/send`
  2. Use valid `room_id`
  3. Use valid `gift_id`
  4. Set `quantity = 10`
  5. Set `winners_count = 5`
  6. Use unique `idempotency_key`

- **Expected Result:**
  - Response status = 200
  - Coins deducted correctly
  - Winners selected successfully
  - Remaining balance updated correctly
  - `gift.sent` websocket event triggered

- **Why it matters:**
  Validates the core money transfer flow.

---

### TC-02: Send gift with minimum valid quantity

- **Category:** Happy Path
- **Priority:** P1

- **Preconditions:**
  - User authenticated
  - Wallet contains enough balance

- **Steps:**
  1. Send request with `quantity = 1`
  2. Set `winners_count = 1`

- **Expected Result:**
  - Request succeeds
  - Minimum allowed transaction processed successfully

- **Why it matters:**
  Verifies lower boundary handling.

---

## Validation

### TC-03: Reject request when quantity = 0

- **Category:** Validation
- **Priority:** P0

- **Preconditions:**
  - User authenticated

- **Steps:**
  1. Send request with `quantity = 0`

- **Expected Result:**
  - Response status = 422
  - Validation error returned for quantity field

- **Why it matters:**
  Prevents invalid wallet deductions.

---

### TC-04: Reject request when winners_count exceeds audience size

- **Category:** Validation
- **Priority:** P0

- **Preconditions:**
  - Room audience size = 5

- **Steps:**
  1. Send request with `winners_count = 10`

- **Expected Result:**
  - Response rejected
  - Validation message returned

- **Why it matters:**
  Prevents impossible reward distribution.