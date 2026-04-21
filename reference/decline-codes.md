# Payment decline codes reference

This reference lists all payment authorization decline codes and indicates whether each represents a hard or soft decline for different transaction types.

## How to use this reference

### Decline type definitions

* **Hard decline**: This reference lists all payment authorization decline codes and indicates whether each represents a hard or soft decline for different transaction types.
* **Soft decline**: A temporary failure. Retrying the payment, either immediately or after the temporary issue is resolved, may result in a successful transaction.

### Transaction types

* **Customer-initiated**: Customer actively making a purchase at checkout.
* **Merchant-initiated&&: Automated charges like subscription renewals or saved card payments.

For implementation guidance, see [Handling authorization failures](../how-to-guides/handling-authorization-failures.md).

## Decline codes table

| Decline code | Customer-initiated | Merchant-initiated |
|------------------------------|--------------------|--------------------|
| `account_closed` | Hard | Hard |
| `account_frozen` | Hard | Hard |
| `authentication_required` | Soft | Soft |
| `blacklisted_card` | Hard | Hard |
| `card_expired` | Hard | Soft |
| `card_limit_exceeded` | Hard | Soft |
| `card_not_active` | Hard | Soft |
| `card_type_block` | Hard | Hard |
| `card_velocity_exceeded` | Hard | Soft |
| `declined` | Hard | Hard |
| `declined_can_retry` | Soft | Soft |
| `do_not_honor` | Soft | Soft |
| `duplicate_transaction` | Hard | Soft |
| `fraud` | Hard | Hard |
| `fraud_block` | Hard | Hard |
| `illegal_action` | Hard | Hard |
| `insufficient_funds` | Hard | Soft |
| `invalid_address` | Hard | Hard |
| `invalid_amount` | Hard | Hard |
| `invalid_card_bin` | Hard | Hard |
| `invalid_card_number` | Hard | Hard |
| `invalid_currency` | Hard | Soft |
| `invalid_expiration_date` | Hard | Soft |
| `invalid_field_data` | Hard | Hard |
| `invalid_merchant` | Hard | Hard |
| `invalid_payment_method` | Hard | Hard |
| `invalid_pin` | Hard | Hard |
| `invalid_security_code` | Hard | Hard |
| `invalid_security_field` | Hard | Hard |
| `invalid_transaction_type` | Hard | Hard |
| `issuer_invalid_card` | Hard | Hard |
| `issuer_not_found` | Hard | Hard |
| `issuer_unavailable` | Soft | Soft |
| `limit_exceeded` | Hard | Hard |
| `lost_stolen_card` | Hard | Hard |
| `mid_limit_exceeded` | Hard | Soft |
| `new_card_issued` | Hard | Soft |
| `no_response` | Soft | Soft |
| `pin_try_exceeded` | Hard | Soft |
| `restricted_card` | Hard | Hard |
| `sca_not_completed` | Soft | Soft |
| `stop_recurring` | Hard | Hard |
| `suspected_fraud` | Hard | Soft |
| `unidentified_error` | Soft | Soft |
| `voice_authorization_required` | Hard | Hard |

## Code example for decline codes

```python
    soft_decline_codes = {
        'customer_initiated': [
            'authentication_required',
            'declined_can_retry',
            'do_not_honor',
            'issuer_unavailable',
            'no_response',
            'sca_not_completed',
            'unidentified_error'
        ],
        'merchant_initiated': [
            'authentication_required',
            'card_expired',
            'card_limit_exceeded',
            'card_not_active',
            'card_velocity_exceeded',
            'declined_can_retry',
            'do_not_honor',
            'duplicate_transaction',
            'insufficient_funds',
            'invalid_currency',
            'invalid_expiration_date',
            'issuer_unavailable',
            'mid_limit_exceeded',
            'new_card_issued',
            'no_response',
            'pin_try_exceeded',
            'sca_not_completed',
            'suspected_fraud',
            'unidentified_error'
        ]
    }
```

## Decline code categories

### Account-related declines

#### account_closed

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: The account has been permanently closed

#### account_frozen

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: The account is temporarily frozen by the bank

### Authentication & verification declines

#### authentication_required

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: Strong Customer Authentication was not completed

#### sca_not_completed

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: Strong Customer Authentication was not completed

### Card status declines

#### card_expired

* Customer-initiated: Hard (customer must use a different card)
* Merchant-initiated: Soft (customer may have received a new card)
* Meaning: The card expiration date has passed

#### card_not_active

* Customer-initiated: Hard (not activated yet)
* Merchant-initiated: Soft (may have been activated since saved)
* Meaning: The card has not been activated for use

#### new_card_issued

* Customer-initiated: Hard (customer must use the new card)
* Merchant-initiated: Soft (may need card details updated)
* Meaning: The bank issued a replacement card

#### lost_stolen_card

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Card reported as lost or stolen

### Limit & funds declines

#### card_limit_exceeded

* Customer-initiated: Hard (over credit limit now)
* Merchant-initiated: Soft (limit may reset by next billing)
* Meaning: Transaction exceeds the card’s credit limit

#### card_velocity_exceeded

* Customer-initiated: Hard (too many transactions today)
* Merchant-initiated: Soft (velocity may reset tomorrow)
* Meaning: Too many transactions in a short time period

#### insufficient_funds

* Customer-initiated: Hard (not enough money now)
* Merchant-initiated: Soft (funds may be available later)
* Meaning: Account balance too low for the transaction

#### limit_exceeded

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: General limit exceeded

#### mid_limit_exceeded

* Customer-initiated: Hard (merchant limit reached)
* Merchant-initiated: Soft (limit may reset)
* Meaning: Merchant ID transaction limit exceeded

### Fraud & security declines

#### fraud

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Transaction flagged as fraudulent

#### fraud_block

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Card blocked due to fraud concerns

#### suspected_fraud

* Customer-initiated: Hard (blocked now)
* Merchant-initiated: Soft (may be cleared later)
* Meaning: Transaction suspected of being fraudulent

#### blacklisted_card

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: The card is on the merchant or network blacklist

### Invalid data declines

#### invalid_address

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Billing address verification failed

#### invalid_amount

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Transaction amount is invalid

#### invalid_card_bin

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Card BIN (first 6 digits) is invalid

#### invalid_card_number

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Card number failed validation

#### invalid_currency

* Customer-initiated: Hard (wrong currency for this card)
* Merchant-initiated: Soft (may work with a different currency)
* Meaning: Currency not supported for this card

#### invalid_expiration_date

* Customer-initiated: Hard (date format wrong)
* Merchant-initiated: Soft (stored date may be wrong)
* Meaning: Expiration date format is invalid

#### invalid_pin

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Incorrect PIN entered

#### invalid_security_code

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: CVV/CVC code is incorrect

### System & network declines

#### issuer_unavailable

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: The card issuer’s system is temporarily down

#### no_response

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: No response received from the card networ

#### unidentified_error

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: Unknown error occurred

### General declines

#### `declined`

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Generic decline with no specific reason

#### `declined_can_retry`

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: Decline that explicitly allows retry

#### `do_not_honor`

* Customer-initiated: Soft
* Merchant-initiated: Soft
* Meaning: Bank declined without a specific reason (may work later)

### Transaction type declines

#### `duplicate_transaction`

* Customer-initiated: Hard (appears to be a duplicate now)
* Merchant-initiated: Soft (may be valid on retry)
* Meaning: Appears to be a duplicate of a recent transaction

#### `card_type_block`

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: This card type is not accepted

#### `stop_recurring`

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Customer requested a stop to recurring charges

#### `voice_authorization_required`

* Customer-initiated: Hard
* Merchant-initiated: Hard
* Meaning: Requires phone authorization from the card issuer

## Next steps

### Explanation

* [Understanding authorization and capture](../explanation/understanding-authorization-capture.md) - Learn the concepts

### How-to guides

* [Handle authorization failures](../how-to-guides/handling-authorization-failures.md) - Implement retry logic

---

Last updated:  April 2026
