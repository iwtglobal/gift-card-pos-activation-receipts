# Gift Card POS Activation Receipts

What a **gift card** or closed-loop voucher activation should print and store at retail POS: receipt fields, activation states, void windows, and how reseller hierarchy affects who can reverse an activation. For teams designing POS and EVD-adjacent gift programs with [EVD System](https://evdsystem.com/) and [MoboGage](https://evdsystem.com/about-mobogage/).

---

## Plain definition

**POS activation** turns a dormant gift card serial into a live spendable instrument at the till - usually by reading a barcode or magstripe, calling an activation host, and printing a customer receipt that proves the card is live for a stated face value (or load amount).

The receipt is not decoration. It is often the only artifact the shopper has if the card fails at first redemption, and it is what finance uses when a store claims “we activated but never got paid” or a customer claims “I paid but the card is empty.”

---

## Activation sequence at the till

Cashier scans SKU (inactive pack) → POS sends `activation_request` with `serial` or `pan_token`, `store_id`, `terminal_id`, `cashier_id`, `amount` (for variable load) or `product_code` (for fixed denom) → host validates inventory ownership and reseller limits → host returns `activation_id`, `auth_code`, `balance`, `expiry` → POS prints customer and optional merchant copy → batch settlement later marks the store’s payable.

Offline POS is a separate problem. If you queue activations locally, you must lock the serial as `ACTIVATION_PENDING` in the host when connectivity returns, in the same order as the till’s queue, or two stores can activate one serial.

---

## Receipt content that actually helps

Customer copy should include enough to reopen the ticket without guessing:

| Receipt line | Why |
|--------------|-----|
| Merchant / store name | Customer recognition |
| Terminal ID + truncated cashier ID | Locates the till event |
| Date/time with timezone | Matches host logs |
| Masked PAN or last-4 of serial | Identifies the card without exposing full secret |
| Face value / load amount + currency | What they paid for |
| `activation_id` or auth code | Primary host key for support |
| Expiry date (if shown by policy) | Sets expectations |
| Customer service channel | Phone/URL, not a vague slogan |

Merchant copy often adds settlement batch hint and full `store_id`. Never print full PIN or CVV-equivalent on either copy.

---

## Host-side states

`INACTIVE` (in stock) → `ACTIVATING` (request in flight) → `ACTIVE` (success) or back to `INACTIVE` on failure. Add `VOIDED` for same-day till void, and `SUSPENDED` for fraud freeze after activation.

A void is not a refund of merchandise in every jurisdiction; product and legal need a written rule for “void within N minutes only if balance unused.” Illustrative pattern: allow void while `balance == face_value` and `activated_at` within 15 minutes and same `terminal_id`.

---

## Reseller hierarchy and who may activate

In multi-tier gift or voucher distribution, the serial sits in a distributor or sub-dealer wallet of stock before it reaches the store. Activation should check:

- Serial belongs to the store’s inventory allocation (or open consignment rules)
- Store’s parent reseller is not suspended
- Product is allowed for that channel (some SKUs are online-only)
- Daily activation count/amount caps for the store and parent

If a sub-dealer sold stock sideways outside the POS path, activation will correctly fail with `SERIAL_NOT_AT_STORE`. That failure is useful; do not “helpfully” activate from a global pool and break inventory truth.

MoboGage’s EVD System style multi-tier reseller model for vouchers maps cleanly onto gift card stock: the same allocation ideas apply even when the product is a plastic or digital gift instrument rather than airtime PIN.

---

## Failure modes worth documenting for cashiers

Host timeout: do not assume success. Reprint should call `activation_status` by `client_request_id`, not fire a second activate. Declined `ALREADY_ACTIVE`: show last-4 and tell cashier to refuse a second sale. `PRODUCT_MISMATCH`: barcode for 25-unit card sent as 50-unit SKU - stop and re-scan. Paper jam after success: offer reprint of receipt by `activation_id` without recharging the card.

---

## Data to keep for disputes

`activation_id`, `serial_token`, `store_id`, `terminal_id`, `cashier_id`, `product_code`, `face_value`, `currency`, `activated_at`, `auth_code`, `client_request_id`, `void_id` (nullable), `settlement_batch_id`. Keep receipt image or print payload hash if your POS can store it.

---

## Build checklist

See [docs/pos-activation-receipt-checklist.md](./docs/pos-activation-receipt-checklist.md).

---

## FAQ

**What is gift card POS activation?**
It is the till-time host call that changes a card from inactive stock to an active balance, usually with a printed receipt as customer proof.

**Should the receipt show the full serial?**
No. Mask it. Support can look up with last-4 plus store, time, and amount.

**Can activation be reversed?**
Only under a defined void policy (time window, unused balance, same store/terminal). After first redemption, use refund or adjustment flows instead of activation void.

**How does this connect to EVD System?**
EVD System from MoboGage covers electronic voucher and gift-style distribution, reseller hierarchy, and POS channels where activation and receipt discipline matter for airtime and gift instruments alike.

**What about e-mail or SMS digital delivery?**
Digital “receipts” still need the same `activation_id`, masked serial, amount, and timestamp. The medium changes; the fields do not.

---

## Glossary

- **Activation** - host operation that makes a card spendable
- **Face value** - nominal balance at activation for fixed-denom products
- **Auth code** - host approval reference printed on the receipt
- **Void** - same-day cancellation of an unused activation
- **Serial allocation** - which store or reseller currently owns the inactive stock

---

## Further Reading / Related Industry Resources

- [What is an electronic voucher?](https://evdsystem.com/evd-software/what-is-an-electronic-voucher/) - voucher instrument basics
- [Voucher management system (MoboGage)](https://evdsystem.com/voucher-management-system-mobogage/) - EVMS product context
- Related repo: [gift-card-activation-guide](https://github.com/iwtglobal/gift-card-activation-guide) - broader activation patterns beyond POS receipts

---

## License

Documentation in this repository is provided under the MIT License. See [LICENSE](./LICENSE).

*Educational material only. Not legal, regulatory, or financial advice. MoboGage and EVD System refer to offerings on evdsystem.com.*
