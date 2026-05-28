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
## Authentication & Authorization

### TC-05: Reject request with invalid bearer token

- **Category:** Authentication & Authorization
- **Priority:** P0

- **Preconditions:**
  - Invalid bearer token used

- **Steps:**
  1. Send POST request using invalid token

- **Expected Result:**
  - Response status = 401
  - Unauthorized error returned
  - No wallet deduction occurs

- **Why it matters:**
  Prevents unauthorized money transfer.

---

### TC-06: Reject request when user is not joined to the room

- **Category:** Authentication & Authorization
- **Priority:** P0

- **Preconditions:**
  - User authenticated
  - User is not joined to target room

- **Steps:**
  1. Send request using valid room_id user did not join

- **Expected Result:**
  - Request rejected
  - No winners selected
  - No coins deducted

- **Why it matters:**
  Prevents abusing room APIs.

---

## Concurrency / Race Conditions

### TC-07: Parallel requests with insufficient balance for both

- **Category:** Concurrency / Race Conditions
- **Priority:** P0

- **Preconditions:**
  - Wallet balance only covers one request
  - Two identical requests prepared

- **Steps:**
  1. Send two requests simultaneously
  2. Each request deducts full remaining balance

- **Expected Result:**
  - Only one request succeeds
  - Second request fails gracefully
  - Wallet balance never becomes negative

- **Why it matters:**
  Prevents double-spending race conditions.

---

### TC-08: Concurrent requests using same idempotency_key

- **Category:** Concurrency / Race Conditions
- **Priority:** P0

- **Preconditions:**
  - Same authenticated user
  - Same idempotency_key

- **Steps:**
  1. Send two requests simultaneously with same key

- **Expected Result:**
  - Only one transaction created
  - Coins deducted once only
  - Duplicate request returns cached/replayed response

- **Why it matters:**
  Prevents duplicate payments caused by retries.

---

## Idempotency

### TC-09: Retry same request after successful response

- **Category:** Idempotency
- **Priority:** P0

- **Preconditions:**
  - First request already completed successfully

- **Steps:**
  1. Resend identical request with same idempotency_key

- **Expected Result:**
  - Existing transaction returned
  - No additional wallet deduction
  - Winners list unchanged

- **Why it matters:**
  Prevents duplicated charges from mobile retry behavior.

---

## Business Logic & Money

### TC-10: Reject request when wallet balance is insufficient

- **Category:** Business Logic & Money
- **Priority:** P0

- **Preconditions:**
  - Wallet balance less than required amount

- **Steps:**
  1. Send request exceeding available balance

- **Expected Result:**
  - Request rejected
  - No partial deduction occurs
  - Transaction rolled back fully

- **Why it matters:**
  Protects wallet integrity.