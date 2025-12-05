# Authorization vs. capture: understanding the difference

This guide explains the key differences between payment authorization and capture, and why each step is important in payment processing.

When someone pays, the first step is to authorize the payment. This checks if their account is real and has enough money. The money is not taken yet, but it’s set aside for the purchase. Only after this check passes does the next step (capture) happen.

Capture is the next step, when the money is actually taken from the shopper and given to the merchant. Sometimes, both steps happen together. Sometimes, capture is done later. In short, Authorization holds the money, capture collects the money.

Common payment flows:

- **Immediate capture**: Both steps happen at once (e.g., digital goods).

- **Delayed capture**: Authorization happens first, capture comes later (e.g., physical goods shipping). If authorization fails, we return an error explaining the reason.

**Important**: Do not share the error code with the customer. Doing so may aid parties attempting to commit fraudulent activities.

Some common reasons for authorization declines include incorrect credit card numbers, invalid security codes, and insufficient funds. In the event of a [hard or soft authorization decline](#hard-declines-vs-soft-declines), you maintain responsibility for communicating with the customer and following the [retry policies](#retry-policies).

## Hard declines vs. soft declines

There are two major types of authorization declines: hard and soft.

Hard declines are permanent authorization failures.  In other words, retrying the payment method won't succeed. These failures happen when the payment method is no longer valid, such as if the account is closed or the card is stolen. Customers will need to provide another payment method or fix any wrong payment information.

Some common reasons for authorization declines include incorrect credit card numbers, invalid security codes, and insufficient funds. In the event of a hard or soft authorization decline, you remain responsible for communicating with the customer and following the [retry policies](#retry-policies). 

Soft declines are temporary problems, like not enough money or a daily limit being reached. If you try the payment again, it might work. Just make sure to follow the retry rules.

## Decline codes 

The following table explains whether the error code you receive indicates a [hard or soft decline](#hard-declines-vs-soft-declines) for customer-initiated and merchant-initiated transactions. When it's a soft decline, make sure you follow the retry policies when submitting additional authorization requests. See the complete list of error codes for more information.

| Decline code                   | Description                                |
|--------------------------------|--------------------------------------------|
| `account_closed`               | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `account_frozen`               | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `authentication_required`      | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| blacklisted_card`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `card_expired`                 | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `card_limit_exceeded`          | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `card_not_active`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `card_type_block`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `card_velocity_exceeded`       | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `declined`                     | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `declined_can_retry`           | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| `do_not_honor`                 | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| `duplicate_transaction`        | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `fraud`                        | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
! `fraud_block`                  | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `illegal_action`               | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `insufficient_funds`           | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `invalid_address`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_amount`               | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_card_bin`             | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_card_number`          | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_currency`             | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `invalid_expiration_date`      | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `invalid_field_data`           | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_merchant`             | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_payment_method`       | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_pin`                  | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_security_code`        | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_security_field`       | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `invalid_transaction_type`     | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `issuer_invalid_card`          | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `issuer_not_found`             | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `issuer_unavailable`           | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| `limit_exceeded`               | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `lost_stolen_card`             | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `mid_limit_exceeded`           | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `new_card_issued`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `no_response`                  | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| `pin_try_exceeded`             | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `restricted_card`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `sca_not_completed`            | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| `stop_recurring                | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |
| `suspected_fraud`              | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Soft |
| `unidentified_error`           | * Customer-initiated decline type: Soft
* Merchant-initiated decline type: Soft |
| `voice_authorization_required` | * Customer-initiated decline type: Hard
* Merchant-initiated decline type: Hard |

## Retry policies

Follow these steps when resubmitting an authorization request: First, attempt authorization a maximum of once per day for each subscription. Second, do not exceed four total attempts within a 30-day period for any subscription. Track the dates of your retries to ensure you do not exceed these limits.

Additionally, we recommend avoiding scheduling retries 7 days apart. This prevents them from falling on the same day of the week, thereby helping you obtain a higher authorization success rate.
