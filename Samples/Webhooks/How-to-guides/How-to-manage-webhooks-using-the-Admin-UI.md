# How to manage webhooks using the Admin UI

The Admin UI lets you search for, create, edit, enable, disable, or delete webhooks, and manage secrets. Once you register URLs, event notifications are sent for processing.

You can reveal and rotate a webhook's secret in Admin UI.

## Searching for a webhook

You can easily search for an existing webhook in Admin UI to review or update its settings. Use the steps below to find a webhook by its unique ID.

1. Sign in to Admin UI.
2. Select **Administration**, then click **Webhook Service**. 
3. On **Webhook Service**, enter the webhook ID and click **Search**. Exact matches show up.

## Creating a webhook

Creating a webhook allows your system to receive real-time notifications from the service, enabling you to automate and enhance your processes. Whether you're tracking subscription updates, inventory statuses, or other events, setting up a webhook is straightforward: open your firewall to the service IPs, create the webhook in your service project, and configure your system to respond to the events. Follow these simple steps to get started.

You can create a webhook to send notifications in three easy steps:

### Step 1: Open your firewall to trusted service IP addresses
To receive webhook notifications from the service, you must open your firewalls to all trusted service IP addresses listed in the Payment API safelist. See the Payment API Safelist section for trusted service IP addresses. See the Payment API Safelist section for trusted service IP addresses.

### Step 2: Create webhooks

1. Sign in to Admin UI.
2. Select **Administration**, then click **Webhook Service**. The Webhook Service page appears.
3. Click **Create Webhook** on **Webhook Service**.
4. On the Create Webhook page, enter the endpoint URL. It must begin with `https://` or `http://` and will receive the webhook payload.
5. Choose **HTTP** or **OAUTH** as **Transport Type**.
6. Enter Username and Password for the endpoint.
7. Under OAuth, create webhooks by:

   a. Selecting **Client credentials** or **Password** in from the **Grant type** list. 
   
   b. Entering the token endpoint URL starting with either `HTTP://` or `HTTPS://` in the **Token endpoint URL field**.
   
   c. Providing the password or client ID and client secret in the appropriate fields.
8. (Optional) Select **Enabled** for **Status**. If not enabled, no payment is sent. The system also prompts you to enable the webhook when you click **Create**.
9. Select event checkboxes you want for the endpoint. At least one required.
10. Click **Create**.

By following these steps, you will successfully subscribe to selected event types, enabling your systems to receive notifications and respond to changes promptly.

### Step 3: Respond to webhook events

Your endpoint must return a `2xx` HTTP status code to acknowledge the receipt of an event. If the endpoint fails to acknowledge events for several days, it will be disabled.

You will not receive the event if the service receives response codes outside this range. For example, the service treats a URL redirection as a failure.

## Editing webhooks

## Turning webhooks on or off

## Revealing a webhook's secret

## Viewing the webhook details

## Deleting a webhook
