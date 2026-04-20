# Understanding payment authorization and capture

Before reading, you should be familiar with basic payment processing terms, as this guide explains payment authorization and capture, and why processing uses two steps.

## What is authorization?

At checkout, the payment system sends an authorization request to verify the customer’s account and funds. No money moves yet; it’s set aside for the purchase.

Authorization is a temporary hold as the processor confirms:

* The account has enough available funds or credit.
* The card hasn’t been reported as lost or stolen.
* The transaction isn’t flagged as suspicious.

If the checks pass, funds are reserved but remain in the customer’s account; the merchant can’t access them yet.

## What is capture?

Capture is when money moves from the customer to the merchant. After capture, the funds transfer and the transaction are final.

## Why use two separate steps?

The separation of authorization and capture exists for these reasons:

**Business flexibility:** Merchants often need time between confirming an order and charging for it. For example:

* Ecommerce stores verify inventory before charging.
* Hotels authorize at check-in but capture at check-out (when they know the final bill).
* Custom orders may need production time before payment is collected.

**Risk management:** Two-step payment processing helps prevent:

* Charging customers for orders that can’t be fulfilled.
* Disputes over services not yet rendered.
* Refunding payments for canceled orders.

**Financial timing:** Some industries have specific billing practices:

* Subscription services may authorize monthly, but capture when the service period ends.
* Rental services authorize payment for potential damages but only capture actual charges.

## Authorization holds and expiry

An authorization hold reserves funds temporarily based on the card type. Different card networks have different hold periods:

* Credit cards: 2-5 days typically
* Debit cards: Usually shorter, 1-3 days
* Commercial cards: May allow longer holds

**Expiry windows:** Some businesses place longer holds. For example, hotels may hold funds for a 7-day stay since the final amount isn’t known until checkout.

If the merchant doesn’t capture the payment in time, the hold expires, and the funds are returned to the customer.

## Common payment flows

### Immediate capture

Authorization and capture occur simultaneously in a single transaction. The customer is charged immediately.

**When it's used:**

* Digital goods and downloads
* Small purchases
* Services delivered immediately
* Subscription renewals

**Example:** Buying an e-book online. The moment you complete the purchase, you’re charged, and you can download the book.

### Delayed capture

First, authorization happens. Capture occurs later—sometimes days or weeks after.

**When it's used:**
* Physical goods that need shipping
* Made-to-order products
* Hotel reservations
* Car rentals
* Pre-orders

**Example:** Ordering furniture online. The store authorizes your card to confirm you can pay, but doesn’t capture until the furniture ships.

## Understanding decline types

Payment authorizations sometimes fail, resulting in either hard or soft declines.

### Hard declines

A hard decline is a permanent failure. Retrying the same payment method won’t work because the issue can’t be fixed by waiting.

**Common causes:**
* Account is closed.
* The card was reported as lost or stolen.
* The card has expired.
* Invalid card number or security code.
* Card type not accepted by the merchant.
* Suspected fraud.

**What happens next:** The customer must provide updated payment information or use another payment method.

### Soft declines

A soft decline is temporary. The same payment method might work if tried later.

**Common causes:**

* Insufficient funds at this moment.
* Daily spending limit reached.
* Card network temporarily unavailable.
* The bank’s fraud system wants additional verification.
* Authorization expired and needs renewal.

**What happens next:** The payment may succeed if retried after the issue resolves. For example, insufficient funds today might not be an issue tomorrow after a paycheck deposit.

## Why decline classification matters

Knowing the decline type determines the correct next action:

**Hard declines:**

* Stop retrying immediately.
* Prompt the customer for a new payment method.
* Save system resources by not making futile retry attempts.
* Provide a better customer experience with clear messaging.

**Soft declines:**

* Implement smart retry logic.
* Give the temporary condition time to resolve.
* Balance retry attempts with respectful customer treatment.
* Maintain revenue by recovering payments that will eventually succeed.

## Security considerations

**Important:** Never share the actual decline code with customers. Error codes can help fraudsters:

* Test stolen card numbers systematically.
* Understand which verification checks are failing.
* Refine their approach to bypass fraud detection.

Instead, show customers generic, helpful messages like:

* “Your payment could not be processed. Please try a different payment method.”
* “We couldn’t complete your payment. Please contact your bank for assistance.”

Keep the specific technical decline codes in your logs for internal troubleshooting and analysis.

## Next steps

Now that you understand the concepts:

- **Reference:** [Payment decline codes](../reference/decline-codes.md) - Look up specific decline code meanings
- **How-to:** [Handle authorization failures](../how-to-guides/handling-authorization-failures.md) - Implement retry logic for soft declines
- **How-to:** [Process refunds](../how-to-guides/payment-process-refunds.md) - Handle refunds when needed

---

*Last updated: January 2026*
