# Assignment2.14-Serverless-Architecture2

1. Does SNS guarantee exactly-once delivery to subscribers?
   
   No, SNS does not guarantee exactly-once delivery. Instead, it provides at-least-once delivery semantics, meaning messages are delivered at least once (and potentially multiple times due to retries or network issues), but duplicates are
   possible in rare cases (e.g., due to retries, network issues, subscriber unavailability).

   SNS FIFO topics provide exactly-once delivery by using deduplication IDs within a 5-minute window.
   For applications requiring strict exactly-once guarantees, use FIFO topics or pair SNS with downstream services like Amazon SQS FIFO queues for additional deduplication.
_______________________________________________________________________________________________________________________________________________________________
2. What is the purpose of the Dead-letter Queue (DLQ)?
   
   A Dead-Letter Queue (DLQ) is a secondary Amazon SQS queue used to capture and isolate messages that fail after the maximum allowed delivery or processing attempts. Its primary purposes include:
   - Debugging and Analysis: Store failed messages for inspection of logs, payloads, and error patterns to identify root causes without losing data.
   - Isolation: Prevent failed messages from clogging the main queue or topic, allowing healthy processing to continue.
   - Reprocessing: Enable manual or automated re-injection of messages back into the main workflow after fixes.
   - Auditing and Compliance: Retain problematic messages for long-term storage and review.

   For SNS: if a subscriber (e.g., Lambda or SQS) fails consistently, SNS can move the undeliverable messages to a DLQ.

   For SQS: if a consumer keeps failing to process a message, SQS moves it to a DLQ after the maximum receive count is exceeded.

   For EventBridge: if a target service cannot process the event, the event can be sent to a DLQ.
_________________________________________________________________________________________________________________________________________________________________
3. How would you enable a notification to your email when messages are added to the DLQ?

   An email notification can be enabled when messages are added to a DLQ (which is an Amazon SQS queue) using Amazon CloudWatch and Amazon SNS.

   The general process:
  
   i) Create an SNS Topic for Notifications:
     - In the AWS SNS console, create a new topic (e.g., DLQ-Alerts).
     - Subscribe the desired Email Address to the newly created SNS topic (A confirmation email will be received that must be clicked to activate the subscription.).
     - Set the protocol to "Email" and ensure raw message delivery is disabled for simple text alerts.

   ii) Set Up a CloudWatch Alarm on DLQ Metrics:
     - In the CloudWatch console, navigate to Alarms > Create alarm.
     - Select the DLQ (SQS queue) as the metric source.
     - Choose the metric "ApproximateNumberOfMessagesVisible" (or "ApproximateNumberOfMessagesNotVisible" for in-flight messages).
     - Set the Threshold to be greater than 0 (or ≥1) for one evaluation period. This means the alarm will trigger as soon as one or more messages land in the DLQ.
     - In "Notification" actions, select "In alarm" and choose the created SNS topic (DLQ-Alerts).
     - Test by sending a message to DLQ; confirm email receipt.

  When a message lands in the DLQ, the "ApproximateNumberOfMessagesVisible" metric rises, triggering a CloudWatch Alarm. The alarm then publishes to an SNS Topic, which forwards an email notification to the subscribed recipient.
