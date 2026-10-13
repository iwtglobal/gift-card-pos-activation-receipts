# POS Activation Receipt Checklist

Build and store checklist for gift card / closed-loop voucher activation at retail POS.

## Host activation

- [ ] Idempotent `client_request_id` from POS
- [ ] Serial ownership check against store allocation
- [ ] Parent reseller not suspended
- [ ] States: INACTIVE → ACTIVATING → ACTIVE | fail back to INACTIVE
- [ ] Status inquiry API for reprints and timeouts (no second activate)

## Receipt (customer copy)

- [ ] Store name, local date/time with zone
- [ ] Terminal ID, masked serial/PAN last-4
- [ ] Face value or load amount + currency
- [ ] `activation_id` or auth code
- [ ] Expiry if policy allows display
- [ ] Support phone/URL
- [ ] No full PIN/CVV print

## Void rules

| Condition | Typical gate |
|-----------|--------------|
| Time since activation | Within configured minutes |
| Balance | Still equal to face/load |
| Location | Same store/terminal preferred |
| Approver | Manager creds if above threshold |

## Dispute pack

Keep `activation_id`, `store_id`, `terminal_id`, `cashier_id`, `product_code`, amounts, timestamps, `void_id`, `settlement_batch_id`.
