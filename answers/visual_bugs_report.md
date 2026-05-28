# Exercise 7 — Visual Bugs Report

## Bug #1: Total cost is calculated incorrectly
- **Where:** Send Gift screen — gift selection and quantity field
- **Steps to reproduce:**
  1. Open the Send Gift tab.
  2. Select Diamond gift with price 1,500 coins.
  3. Enter quantity = 10.
  4. Check the displayed Total cost.
- **Expected behavior:** Total cost should be 15,000 coins.
- **Actual behavior:** Total cost shows 1,510 coins.
- **Screenshot:** Not attached.
- **Severity:** High
- **Severity reasoning:** This is a money-related issue that can mislead the admin before sending the gift.
- **Suspected root cause:** Frontend logic.
- **Proposed fix:** Calculate total as `price * quantity`, not `price + quantity`.

---

## Bug #2: Quantity accepts invalid values
- **Where:** Send Gift screen — Quantity field
- **Steps to reproduce:**
  1. Open the Send Gift tab.
  2. Select any gift.
  3. Enter quantity = -5, 0, decimal value, or text.
  4. Submit the form.
- **Expected behavior:** Quantity should accept only integers from 1 to 100.
- **Actual behavior:** Invalid values are accepted or not properly blocked.
- **Screenshot:** Not attached.
- **Severity:** High
- **Severity reasoning:** Invalid quantity can cause wrong wallet charging or incorrect transaction behavior.
- **Suspected root cause:** Frontend validation.
- **Proposed fix:** Use `type="number"` with min/max validation and block invalid input before submit.

---

## Bug #3: Send Gift button remains disabled after successful submission
- **Where:** Send Gift screen — Send Gift button
- **Steps to reproduce:**
  1. Enter valid room ID, gift, quantity, and winners count.
  2. Click Send Gift.
  3. Wait until the success toast appears.
  4. Try to send another gift.
- **Expected behavior:** Button should return to normal state after success.
- **Actual behavior:** Button stays disabled with “Sending...” text.
- **Screenshot:** Not attached.
- **Severity:** Medium
- **Severity reasoning:** User cannot continue using the main action without refreshing the page.
- **Suspected root cause:** Frontend state handling.
- **Proposed fix:** Re-enable the button and restore button text after the async action completes.

---

## Bug #4: Header wallet balance does not update after sending gift
- **Where:** Header wallet pill and Wallet tab
- **Steps to reproduce:**
  1. Send a valid gift.
  2. Open the Wallet tab.
  3. Compare the Wallet tab balance with the header balance.
- **Expected behavior:** Both balances should show the same updated value.
- **Actual behavior:** Wallet tab balance updates, but the header balance remains old.
- **Screenshot:** Not attached.
- **Severity:** Medium
- **Severity reasoning:** Inconsistent wallet data can confuse admins during money-related actions.
- **Suspected root cause:** Frontend state/data binding.
- **Proposed fix:** Update all wallet balance UI elements from the same state after each transaction.

---

## Bug #5: Banned user remains visible in Users table
- **Where:** Users tab — Ban action
- **Steps to reproduce:**
  1. Open the Users tab.
  2. Click Ban for any user.
  3. Observe the Users table.
- **Expected behavior:** The banned user should be removed from the table or marked as banned.
- **Actual behavior:** Success toast appears, but the user remains visible.
- **Screenshot:** Not attached.
- **Severity:** Medium
- **Severity reasoning:** The UI shows stale data after a destructive action.
- **Suspected root cause:** Frontend state rendering.
- **Proposed fix:** Re-render the users table after updating the users array.

---

## Bug #6: Refresh Wallet shows success message while console error occurs
- **Where:** Wallet tab — Refresh Wallet button
- **Steps to reproduce:**
  1. Open browser DevTools Console.
  2. Go to Wallet tab.
  3. Click Refresh Wallet.
  4. Check the toast and console.
- **Expected behavior:** Wallet should refresh successfully or show a clear error message.
- **Actual behavior:** Toast says “Wallet refreshed!” while console shows a TypeError.
- **Screenshot:** Not attached.
- **Severity:** Medium
- **Severity reasoning:** The user receives false success feedback while the operation actually fails.
- **Suspected root cause:** Frontend error handling.
- **Proposed fix:** Handle the failed wallet fetch properly and show an error toast instead of success.

---

## Bug #7: Sensitive data stored in localStorage
- **Where:** Browser DevTools → Application → Local Storage
- **Steps to reproduce:**
  1. Open DevTools.
  2. Go to Application tab.
  3. Open Local Storage for the app URL.
  4. Check stored values.
- **Expected behavior:** Sensitive data such as session tokens or admin passwords should not be stored in plain text.
- **Actual behavior:** `session_token` and `admin_password` are stored in localStorage.
- **Screenshot:** Not attached.
- **Severity:** Critical
- **Severity reasoning:** This exposes sensitive authentication data and admin credentials to anyone with browser access.
- **Suspected root cause:** Frontend security / authentication storage.
- **Proposed fix:** Do not store passwords in the browser; store tokens securely using httpOnly secure cookies.

---

## Bug #8: Poor mobile responsiveness
- **Where:** Whole admin panel on mobile-sized viewport
- **Steps to reproduce:**
  1. Open the app.
  2. Resize browser to mobile width.
  3. Check navigation, tables, and form layout.
- **Expected behavior:** Page should scale correctly and remain usable on mobile.
- **Actual behavior:** Page does not scale properly on small screens.
- **Screenshot:** Not attached.
- **Severity:** Low
- **Severity reasoning:** It affects usability but does not directly affect money or security.
- **Suspected root cause:** Layout / responsive design.
- **Proposed fix:** Add viewport meta tag and responsive table/form handling.

---

## How I tested

I explored the app manually by going through the Send Gift, Users, Wallet, and Bans tabs. I tested valid and invalid inputs, including negative values, empty fields, and money-related flows. I also checked state updates after actions like sending gifts and banning users. Finally, I used browser DevTools to inspect console errors and localStorage security issues.