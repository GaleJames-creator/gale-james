# How to manage webhooks using the Admin-UI

The Admin UI lets you search for, create, edit, enable, disable, or delete webhooks, and manage secrets. Once you register URLs, event notifications are sent for processing.

You can reveal and rotate a webhook's secret in Admin UI.

## Searching for a webhook

You can easily search for an existing webhook in Admin UI to review or update its settings. Use the steps below to find a webhook by its unique ID.

To search for an existing webhook:

1. Sign in to Admin UI.
2. Select **Administration**, and then click **Webhook Service**. 
3. On the Webhook Service page, type the webhook's real ID in the **Webhook ID** field and click **Search**. A webhook will appear in the search results only if its ID matches your entry.

## Creating a webhook

Creating a webhook allows your system to receive real-time notifications from the service, enabling you to automate and enhance your processes. Whether you're tracking subscription updates, inventory statuses, or other events, setting up a webhook is straightforward: open your firewall to the service IPs, create the webhook in your service project, and configure your system to respond to the events. Follow these simple steps to get started.

You can create a webhook to send notifications in three easy steps:

### Step 1: Open your firewall to trusted service IP addresses
To receive webhook notifications from service, you must open your firewalls to all trusted service IP addresses listed in the Payment API safelist.
See the Payment API Safelist section for trusted service IP addresses.

### Step 2: Create webhooks

1. Sign in to the Admin UI.
2. Select **Administration**, and then click **Webhook Service**. The Webhook Service page appears.
3. On the Webhook Service page, click **Create Webhook**.
4. On Create Webhook page appears, enter the endpoint's URL in the **Endpoint URL** field. The endpoint URL will receive the webhook payload. The URL must start with either `https://` or `http://`.
5. Choose either **HTTP** or **OAUTH** as the **Transport Type**.
6. Enter the endpoint's credentials in the **Username** and **Password** fields.
7. Under OAuth, create the webhooks by:
   a. Seleting either **Client credentials** or **Password** from the **Grant type** dropdown list.
   b. Entering the token endpoint URL starting with either `HTTP://` or `HTTPS://` in the **Token endpoint URL field**.
   c. Providing the password or client ID and client secret in the appropriate fields.
8. (Optional) Select **Enabled** for **Status**.
    **Note**: If you do not select **Enabled**, no payment will be sent when an event occurs. Also, the system will ask you if you want to enable the webhook when you click **Create**.
9. Select the checkbox next to each event you want to associate with the endpoint, or select the checkbox next to **Event** to select all events. You must select at least one event type.
10. Scroll down and click **Create**.

By following these steps, you will successfully subscribe to selected event types, enabling your systems to receive notifications and respond to changes promptly.

### Step 3: Respond to webhook events

Your endpoint must return a `2xx` HTTP status code to acknowledge the receipt of an event. If the endpoint fails to acknowledge events over several days, your endpoint will be disabled.

You will not receive the event if the seecixw receives response codes outside this range. For example, the service treats a URL redirection as a failure.

## Using webhooks

## Editing webhooks

## Turning webhoos on or off

## Revealing a webhook's secret

## Viewing the webhook details

## Deleting a webhook
