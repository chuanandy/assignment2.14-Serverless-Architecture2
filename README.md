# Assignment2.14-Serverless-Architecture2

1. Does SNS guarantee exactly-once delivery to subscribers?
   No, SNS does not guarantee exactly-once delivery. Instead, it provides at-least-once delivery semantics, meaning messages are delivered at least once (and potentially multiple times due to retries or network issues), but duplicates are
   possible in rare cases (e.g., due to retries, network issues, subscriber unavailability).
   SNS FIFO topics provide exactly-once delivery by using deduplication IDs within a 5-minute window. For stricter guarantees, use FIFO topics or pair SNS with SQS FIFO queues.For applications requiring strict exactly-once guarantees, use FIFO     topics or pair SNS with downstream services like Amazon SQS FIFO queues for additional deduplication.

2. What is the purpose of the Dead-letter Queue (DLQ)?
A DLQ is used to capture messages that could not be successfully processed/delivered after a defined number of retries.

For SNS: if a subscriber (e.g., Lambda or SQS) fails consistently, SNS can move the undeliverable messages to a DLQ.

For SQS: if a consumer keeps failing to process a message, SQS moves it to a DLQ after the maximum receive count is exceeded.

For EventBridge: if a target service cannot process the event, the event can be sent to a DLQ.

👉 Purpose: Prevent message loss, make it easier to debug, inspect, and reprocess problematic messages later.


3. How would you enable a notification to your email when messages are added to the DLQ?

You can chain DLQ → SNS → Email subscription. Steps:

Create an SNS topic (e.g., DLQ-Alerts).

Subscribe your email to the topic (SNS supports email protocol).

Configure the DLQ (SQS) to trigger the SNS topic:

For SQS DLQ: use an SQS → Lambda or SQS → EventBridge rule that forwards new DLQ messages to the SNS topic.

For SNS DLQ: attach an SQS DLQ, then add an EventBridge rule or Lambda to monitor the DLQ and publish to the DLQ-Alerts topic.

This way, every time a message is placed in the DLQ, you’ll get an email alert.
