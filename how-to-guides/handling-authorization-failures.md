# Handling authorization failures

This guide explains how to implement retry logic for soft declines and handle hard declines gracefully in your payment integration.

## Prerequisites

Before you start, make sure you:

* Understand the difference between hard and soft declines (see [Understanding authorization and capture](../explanation/understanding-authorization-capture.md)).
* Have access to the [decline codes reference](../reference/decline-codes.md).
* Can identify transaction types (customer-initiated vs merchant-initiated).

## Step 1: Detect the decline type

Not all authorization failures should be retried. Retry a soft decline, but do not retry a hard decline. Always check the decline type before implementing retry logic. See [When not to retry](../explanation/understanding-authorization-failures#when-not-to-retry) for more information.

### Decision tree: Should you retry?

Use this logic to determine your next action:

```python
def should_retry_authorization(decline_code, transaction_type):
    """
    Determine if an authorization failure is eligible for retry.
    
    Args:
        decline_code: The decline code from the authorization response
        transaction_type: 'customer_initiated' or 'merchant_initiated'
    
    Returns:
        bool: True if retry is appropriate, False otherwise
    """
    from decline_codes import SOFT_DECLINE_CODES
    # See decline-codes.md for the full list of soft decline codes
    return decline_code in SOFT_DECLINE_CODES.get(transaction_type, [])


# Example usage
if should_retry_authorization('insufficient_funds', 'merchant_initiated'):
    # Schedule retry - this is a soft decline for recurring payments
    schedule_retry()
else:
    # Hard decline - notify customer to update payment method
    request_new_payment_method()
```

## Step 2: Implement retry logic for soft declines

For soft declines, which are temporary issues, follow these retry limits to prevent overwhelming the payment system and annoying customers:

See [Retry limits](../explanation/understanding-authorization-failures#retry-limits) for additional information.

### Implementation example

```python
from datetime import datetime, timedelta
import random

class AuthorizationRetryManager:
    def __init__(self):
        self.retry_attempts = {}  # subscription_id -> list of attempt dates
    
    def can_retry(self, subscription_id):
        """
        Check if we can retry authorization for this subscription.
        
        Returns:
            tuple: (can_retry: bool, next_retry_date: datetime or None)
        """
        attempts = self.retry_attempts.get(subscription_id, [])
        
        # Check if we've exceeded max attempts in the last 30 days
        thirty_days_ago = datetime.now() - timedelta(days=30)
        recent_attempts = [a for a in attempts if a > thirty_days_ago]
        
        if len(recent_attempts) >= 4:
            return False, None
        
        # Check if we've attempted in the last 24 hours
        if recent_attempts and (datetime.now() - recent_attempts[-1]) < timedelta(hours=24):
            # Calculate next allowed retry time
            next_retry = recent_attempts[-1] + timedelta(hours=24)
            return False, next_retry
        
        return True, None
    
    def schedule_retry(self, subscription_id, attempt_number):
        """
        Schedule the next retry attempt.
        
        Args:
            subscription_id: The subscription identifier
            attempt_number: Current attempt number (1-4)
        
        Returns:
            datetime: Scheduled retry date with jitter
        """
        if attempt_number >= 4:
            return None  # Max attempts reached
        
        # Record this attempt
        if subscription_id not in self.retry_attempts:
            self.retry_attempts[subscription_id] = []
        self.retry_attempts[subscription_id].append(datetime.now())
        
        # Schedule next attempt for tomorrow with random jitter
        # Jitter prevents all retries from happening at exactly the same time
        base_delay = timedelta(days=1)
        jitter = timedelta(hours=random.randint(0, 12))
        
        next_retry = datetime.now() + base_delay + jitter
        return next_retry
```

See [Avoid predictable retry patterns](../explanation/understanding-authorization-failures#avoid-predictable-retry-patterns) for additional information.

## Step 3: Handle hard declines

For permanent authorization failures, promptly notify customers with clear, helpful instructions.

### Customer notification example

```python
def handle_hard_decline(customer, decline_code):
    """
    Handle a permanent authorization failure.
    
    Args:
        customer: Customer object
        decline_code: The specific decline code received
    """
    # DO NOT share the actual decline code with the customer
    # This could aid fraudulent activity
    
    # Send a generic, helpful message
    send_notification(
        customer,
        subject="Payment method needs attention",
        message=(
            "We couldn't process your payment. Please update your "
            "payment information to avoid service interruption.\n\n"
            "Visit your account settings to add a new payment method."
        ),
        action_url=f"https://example.com/account/payment-methods"
    )
    
    # Log the actual decline code for internal investigation
    log_payment_failure(
        customer_id=customer.id,
        decline_code=decline_code,
        severity='high',
        requires_customer_action=True
    )
    
    # Update subscription status if applicable
    if customer.has_active_subscription():
        update_subscription_status(
            customer.id,
            status='payment_failed',
            grace_period_days=7
        )
```

See the [API error message guidelines](../reference/api-error-message-guidelines.md) to learn how to format a good error message.

## Step 4: Track and monitor retry performance

Log each retry attempt to fine-tune your strategy and identify issues.

### Logging example

> **Note the following**:
>
> * To capture extra fields in log output, configure a custom log formatter. See the Python logging documentation for details.
> * The following Python example uses `payment_api.authorize()` as a representative example. Replace it with your actual payment provider's SDK or API client.

```python
import logging

# Configure a custom formatter to capture 'extra' fields in log output.
# Example: logging.basicConfig(format='%(message)s %(payment_id)s')
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def attempt_authorization_with_retry(subscription_id, payment_method, attempt_number=1):
    """
    Attempt authorization with comprehensive logging.
    """
    logger.info(
        f"Authorization attempt {attempt_number}/4 for subscription {subscription_id}",
        extra={
            'subscription_id': subscription_id,
            'attempt_number': attempt_number,
            'payment_method_type': payment_method.type,
            'payment_method_last4': payment_method.last4
        }
    )
    
    try:
        response = payment_api.authorize(
            subscription_id=subscription_id,
            payment_method=payment_method
        )
        
        if response.success:
            logger.info(
                f"Authorization succeeded on attempt {attempt_number}",
                extra={
                    'subscription_id': subscription_id,
                    'attempt_number': attempt_number,
                    'authorization_id': response.authorization_id
                }
            )
            return response
        else:
            is_soft_decline = should_retry_authorization(
                response.decline_code,
                'merchant_initiated'
            )
            
            logger.warning(
                f"Authorization declined: {response.decline_code}",
                extra={
                    'subscription_id': subscription_id,
                    'attempt_number': attempt_number,
                    'decline_code': response.decline_code,
                    'decline_type': 'soft' if is_soft_decline else 'hard'
                }
            )
            
            return response
            
    except Exception as e:
        logger.error(
            f"Authorization attempt failed with exception",
            extra={
                'subscription_id': subscription_id,
                'attempt_number': attempt_number,
                'error': str(e)
            },
            exc_info=True
        )
        raise
```

See [Metric to track](../explanation/understanding-authorization-failures#metrics-to-track) for additional information.

## Complete implementation example

Here's a full working example that integrates all the steps:

```python
from datetime import datetime, timedelta
import random
from typing import Optional, Tuple

class PaymentRetryService:
    def __init__(self, payment_api, notification_service):
        self.payment_api = payment_api
        self.notification_service = notification_service
        self.retry_manager = AuthorizationRetryManager()
    
    def process_authorization(self, subscription_id, customer, payment_method):
        """
        Process authorization with intelligent retry logic.
        
        Returns:
            tuple: (success: bool, authorization_id: str or None)
        """
        # Attempt authorization
        response = self.payment_api.authorize(
            subscription_id=subscription_id,
            payment_method=payment_method
        )
        
        if response.success:
            return True, response.authorization_id
        
        # Authorization failed - determine if we should retry
        transaction_type = 'merchant_initiated'  # Subscription payment
        is_soft_decline = should_retry_authorization(
            response.decline_code,
            transaction_type
        )
        
        if not is_soft_decline:
            # Hard decline - notify customer immediately
            self.handle_hard_decline(customer, response.decline_code)
            return False, None
        
        # Soft decline - check if we can retry
        can_retry, next_retry_date = self.retry_manager.can_retry(subscription_id)
        
        if can_retry:
            # Schedule retry
            attempts_so_far = len(
                self.retry_manager.retry_attempts.get(subscription_id, [])
            )
            next_attempt = self.retry_manager.schedule_retry(
                subscription_id,
                attempts_so_far + 1
            )
            
            logger.info(
                f"Soft decline - retry scheduled for {next_attempt}",
                extra={
                    'subscription_id': subscription_id,
                    'decline_code': response.decline_code,
                    'next_retry_date': next_attempt
                }
            )
            
            # Queue for retry
            self.queue_retry(subscription_id, next_attempt)
            return False, None
        else:
            # Max retries exceeded - treat as hard decline
            logger.warning(
                f"Max retries exceeded for soft decline",
                extra={
                    'subscription_id': subscription_id,
                    'decline_code': response.decline_code
                }
            )
            self.handle_hard_decline(customer, response.decline_code)
            return False, None
    
    def handle_hard_decline(self, customer, decline_code):
        """Handle permanent authorization failure."""
        # Implementation from Step 3
        pass
    
    def queue_retry(self, subscription_id, retry_date):
        """Queue subscription for retry at specified date."""
        # Add to retry queue (implementation depends on your system)
        pass
```

## Troubleshooting

### Problem: Too many retries are being scheduled

**Solution:** Verify your retry limits are being enforced:

```python
# Add assertion to catch bugs
attempts = self.retry_attempts.get(subscription_id, [])
assert len(attempts) <= 4, f"Too many retry attempts for {subscription_id}"
```

### Retries are not improving the success rate

**Possible causes:**

1. Not adding enough time between retries (increase base delay)
2. Retrying hard declines by mistake (check decline code classification)
3. Not adding jitter (all retries hitting the same time window)

**Solution:** Analyze your retry logs to identify the pattern:

```python
# Find subscriptions with multiple soft declines
SELECT subscription_id, COUNT(*) as retry_count
FROM authorization_attempts
WHERE decline_type = 'soft'
GROUP BY subscription_id
HAVING retry_count > 2
ORDER BY retry_count DESC
```

### Problem: Customers are complaining about multiple charge attempts

**Solution:** Ensure you're only attempting authorization (not capture) on retries:

```python
# Correct: Authorization only
response = payment_api.authorize(amount=1000, ...)

# Incorrect: Would attempt to capture immediately
response = payment_api.charge(amount=1000, ...)  # Don't do this on retry!
```

## Next steps

### Explanation

* [Understanding payment authorization and capture](../explanation/understanding-authorization-capture.md): Explains the two-step payment process, authorization holds, and decline types.
* [Understanding payment authorization failures](../explanation/understanding-authorization-failures): Explains the two-step payment process, authorization holds, and decline types.

### How-to guides

* [Processing a refund](./processing-a-refund.md) - Handle refunds when needed.

### Reference

* [API error message guidelines](../reference/api-error-message-guidelines.md) - Look up specific decline code meanings.
* [Decline codes reference](../reference/decline-codes.md) - Look up specific decline codes.
* [Preventing duplicate charges with idempotency keys](../reference/payment-idempotency-keys.md)

---

Last updated: April 2026
