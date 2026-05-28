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
### TC-11: Reject request when room is empty

- **Category:** Business Logic & Money
- **Priority:** P1

- **Preconditions:**
  - User authenticated
  - Room has no eligible viewers

- **Steps:**
  1. Send request with valid gift and valid balance
  2. Set `winners_count = 1`

- **Expected Result:**
  - Request rejected
  - No coins deducted
  - No winners returned

- **Why it matters:**
  Prevents charging users when rewards cannot be distributed.

---

### TC-12: Ensure sender is not selected as winner

- **Category:** Business Logic & Money
- **Priority:** P1

- **Preconditions:**
  - Sender is inside the room
  - Room has other eligible viewers

- **Steps:**
  1. Send valid gift request
  2. Review returned winners list

- **Expected Result:**
  - Sender user_id is not included in winners
  - Coins are awarded only to eligible viewers

- **Why it matters:**
  Prevents self-reward abuse.

---

## Side Effects

### TC-13: WebSocket event is broadcast only after successful transaction

- **Category:** Side Effects
- **Priority:** P1

- **Preconditions:**
  - User has sufficient balance
  - Room has eligible viewers

- **Steps:**
  1. Send valid gift request
  2. Listen to room websocket channel

- **Expected Result:**
  - `gift.sent` event is broadcast once
  - Event contains correct transaction_id, sender, gift, and winners
  - Event is not sent before DB transaction commits

- **Why it matters:**
  Prevents users seeing fake gift events when transaction fails.

---

### TC-14: No websocket event when transaction fails

- **Category:** Side Effects
- **Priority:** P1

- **Preconditions:**
  - User has insufficient balance

- **Steps:**
  1. Send gift request exceeding wallet balance
  2. Listen to websocket events

- **Expected Result:**
  - Request fails
  - No `gift.sent` event is broadcast
  - No wallet or winner balance changes occur

- **Why it matters:**
  Ensures rollback also prevents incorrect real-time notifications.

---

## Negative / Security

### TC-15: Reject negative quantity value

- **Category:** Negative / Security
- **Priority:** P0

- **Preconditions:**
  - User authenticated

- **Steps:**
  1. Send request with `quantity = -1`

- **Expected Result:**
  - Response status = 422
  - Validation error returned
  - No wallet update occurs

- **Why it matters:**
  Prevents balance manipulation using negative values.

---

### TC-16: Reject SQL injection attempt in room_id

- **Category:** Negative / Security
- **Priority:** P1

- **Preconditions:**
  - User authenticated

- **Steps:**
  1. Send request with `room_id = "12345 OR 1=1"`

- **Expected Result:**
  - Request rejected
  - No database error exposed
  - No unauthorized room access occurs
- **Why it matters:**
  Protects the endpoint from injection and data leakage.
### TC-17: Reject malformed JSON request body

- **Category:** Negative / Security
- **Priority:** P1

- **Preconditions:**
  - User authenticated

- **Steps:**
  1. Send request with broken or malformed JSON body

- **Expected Result:**
  - Response status = 400
  - Clear error message returned
  - No wallet deduction occurs

- **Why it matters:**
  Ensures the API handles invalid payloads safely.

---

### TC-18: Reject unsupported or inactive gift_id

- **Category:** Validation
- **Priority:** P1

- **Preconditions:**
  - User authenticated
  - Gift is inactive or does not exist

- **Steps:**
  1. Send request using invalid `gift_id`

- **Expected Result:**
  - Request rejected
  - No coins deducted
  - No websocket event sent

- **Why it matters:**
  Prevents using deleted or disabled gifts.

---

### TC-19: Verify exact total cost calculation

- **Category:** Business Logic & Money
- **Priority:** P0

- **Preconditions:**
  - Gift price = 100 coins
  - Quantity = 10
  - User balance = 1500 coins

- **Steps:**
  1. Send valid gift request

- **Expected Result:**
  - Total cost calculated as 1000 coins
  - Remaining balance becomes 500 coins
  - No rounding or calculation error occurs

- **Why it matters:**
  Prevents financial calculation defects.

---

### TC-20: Verify winner rewards are distributed correctly

- **Category:** Business Logic & Money
- **Priority:** P0

- **Preconditions:**
  - Gift total reward amount is known
  - winners_count = 5

- **Steps:**
  1. Send valid gift request
  2. Check each winner wallet balance

- **Expected Result:**
  - All selected winners receive correct reward amount
  - Total awarded amount matches expected gift reward rules
  - No extra user receives coins

- **Why it matters:**
  Ensures reward distribution integrity.

---

### TC-21: Prevent duplicate winner selection in same request

- **Category:** Business Logic & Money
- **Priority:** P1

- **Preconditions:**
  - Room has enough eligible viewers

- **Steps:**
  1. Send request with `winners_count = 5`
  2. Review returned winners list

- **Expected Result:**
  - Each winner appears only once
  - Winners count equals requested count

- **Why it matters:**
  Prevents one user receiving multiple rewards from the same gift.

---

### TC-22: Verify full rollback when winner credit fails

- **Category:** Transaction / Rollback
- **Priority:** P0

- **Preconditions:**
  - Sender has sufficient balance
  - Simulate failure while crediting winner wallet

- **Steps:**
  1. Send valid gift request
  2. Force or mock failure during winner reward update

- **Expected Result:**
  - Entire transaction is rolled back
  - Sender balance is restored
  - No partial winner rewards remain
  - No websocket event is sent

- **Why it matters:**
  Prevents inconsistent money state across wallets.

---

### TC-23: Handle mobile retry after timeout

- **Category:** Idempotency / Reliability
- **Priority:** P0

- **Preconditions:**
  - First request succeeds on server but client times out
  - Same idempotency_key is reused

- **Steps:**
  1. Send valid request
  2. Simulate client timeout
  3. Retry same request with same idempotency_key

- **Expected Result:**
  - Same original transaction result is returned
  - Coins are deducted once only
  - Winners list stays the same

- **Why it matters:**
  Protects users from double charging during network issues.

---

### TC-24: Verify audit log is created for successful gift transaction

- **Category:** Audit / Observability
- **Priority:** P2

- **Preconditions:**
  - User sends valid gift successfully

- **Steps:**
  1. Complete successful gift request
  2. Check transaction/audit logs

- **Expected Result:**
  - Audit log contains sender_id, room_id, gift_id, quantity, total_cost, winners, timestamp, and transaction_id

- **Why it matters:**
  Helps investigation, reconciliation, and fraud analysis.