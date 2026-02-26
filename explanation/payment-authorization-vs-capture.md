# Authorization vs. capture: understanding the difference

This guide explains the key differences between payment authorization and capture, and why each step is important in payment processing.

When the customer pays during checkout, the checkout sends an authorize payment request. This step checks whether the customer's account is valid and has sufficient funds. The money is not taken yet, but it’s set aside for the purchase. Only after this check passes does the next step (capture) happen.

Capture takes the money from the customer and gives it to the merchant. Sometimes, both steps happen together. Sometimes, capture is done later. In short, authorization holds the money, and capture collects the money.

Common payment flows:

- **Immediate capture**: Both steps happen at once (e.g., digital goods).

- **Delayed capture**: Authorization happens first, capture comes later (e.g., physical goods shipping). If authorization fails, we return an error explaining the reason.

**Important**: Do not share the error code with the customer. Doing so may aid parties attempting to commit fraudulent activities.

## Hard declines vs. soft declines

There are two major types of authorization declines: hard and soft.

Hard declines are permanent authorization failures.  In other words, retrying the payment method won't succeed. These failures happen when the payment method is no longer valid, such as if the account is closed or the card is stolen. Customers will need to provide another payment method or fix any wrong payment information.

Some common reasons for authorization declines include incorrect credit card numbers, invalid security codes, and insufficient funds. In the event of a [hard or soft authorization decline](#hard-declines-vs-soft-declines), you maintain responsibility for communicating with the customer and following the [retry policies](#retry-policies).

Soft declines are temporary problems, like not enough money or a daily limit being reached. If you try the payment again, it might work. Follow the [retry steps](#retry-policies) to ensure successful authorization.

## Decline codes 

The following table explains whether the error code you receive indicates a [hard or soft decline](#hard-declines-vs-soft-declines) for customer-initiated and merchant-initiated transactions. When it's a soft decline, make sure you follow the retry policies when submitting additional authorization requests. See the complete list of error codes for more information.

| Decline code                   | Customer-initiated decline type | Merchant-initiated decline type |
|--------------------------------|---------------------------------|---------------------------------|
| `account_closed`               | Hard                            | Hard                            |
| `account_frozen`               | Hard                            | Hard                            |
| `authentication_required`      | Soft                            | Soft                            |
| `blacklisted_card`             | Hard                            | Hard                            |
| `card_expired`                 | Hard                            | Soft                            |
| `card_limit_exceeded`          | Hard                            | Soft                            |
| `card_not_active`              | Hard                            | Soft                            |
| `card_type_block`              | Hard                            | Hard                            |
| `card_velocity_exceeded`       | Hard                            | Soft                            |
| `declined`                     | Hard                            | Hard                            |
| `declined_can_retry`           | Soft                            | Soft                            |
| `do_not_honor`                 | Soft                            | Soft                            |
| `duplicate_transaction`        | Hard                            | Soft                            |
| `fraud`                        | Hard                            | Hard                            |
| `fraud_block`                  | Hard                            | Hard                            |
| `illegal_action`               | Hard                            | Hard                            |
| `insufficient_funds`           | Hard                            | Soft                            |
| `invalid_address`              | Hard                            | Hard                            |
| `invalid_amount`               | Hard                            | Hard                            |
| `invalid_card_bin`             | Hard                            | Hard                            |
| `invalid_card_number`          | Hard                            | Hard                            |
| `invalid_currency`             | Hard                            | Soft                            | 
| `invalid_expiration_date`      | Hard                            | Soft                            |
| `invalid_field_data`           | Hard                            | Hard                            |
| `invalid_merchant`             | Hard                            | Hard                            |
| `invalid_payment_method`       | Hard                            | Hard                            |
| `invalid_pin`                  | Hard                            | Hard                            |
| `invalid_security_code`        | Hard                            | Hard                            |
| `invalid_security_field`       | Hard                            | Hard                            |
| `invalid_transaction_type`     | Hard                            | Hard                            |
| `issuer_invalid_card`          | Hard                            | Hard                            |
| `issuer_not_found`             | Hard                            | Hard                            |
| `issuer_unavailable`           | Soft                            | Soft                            |
| `limit_exceeded`               | Hard                            | Hard                            |
| `lost_stolen_card`             | Hard                            | Hard                            |
| `mid_limit_exceeded`           | Hard                            | Soft                            |
| `new_card_issued`              | Hard                            | Soft                            |
| `no_response`                  | Soft                            | Soft                            |
| `pin_try_exceeded`             | Hard                            | Soft                            |
| `restricted_card`              | Hard                            | Hard                            |
| `sca_not_completed`            | Soft                            | Soft                            |
| `stop_recurring`               | Hard                            | Hard                            |
| `suspected_fraud`              | Hard                            | Soft                            |
| `unidentified_error`           | Soft                            | Soft                            |
| `voice_authorization_required` | Hard                            | Hard                            |

## Retry policies

Follow these steps when resubmitting an authorization request: 

1. Attempt authorization a maximum of once per day for each subscription. 
2. Do not exceed four total attempts within a 30-day period for any subscription.
3. Track the retry dates to ensure it does not exceed these limits.

Additionally, we recommend avoiding scheduling retries 7 days apart. This prevents them from falling on the same day of the week, thereby helping you obtain a higher authorization success rate.

## Next steps

- [Implement retry logic for failed payments](../how-to-guides/payment-retry-logic.md)
