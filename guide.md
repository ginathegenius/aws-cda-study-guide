# AWS Certified Developer Associate

 

## SQS

### Short & Long Polling

By default queues use short polling.

#### Short polling

 

### Visibility timeout

When a consumer receives and processes a message from the queue, the message remains in the queue. SQS does not automatically delete the message. When the message is received, it remains in the queue. To prevent other consumers from processing the message again, Amazon SQS set a visibility timeout, a period of time that prevents other consumers from receiving and processing the message. The default is 30 seconds, min is 0 sec and max is 12 hr.

 

### Dead-Letter Queues

Allows you to set aside and isolate messages that cant be processed correctly to determine why their processing didn't succeed.

#### When to use

- **Do** use dead-letter queues with standard queues. You should always take advantage of dead-letter queues when your applications don’t depend on ordering. Dead-letter queues can help you troubleshoot incorrect message transmission operations.

- **Do** use dead-letter queues to decrease the number of messages and to reduce the possibility of exposing your system to poison-pill messages (messages that can be received but can’t be processed).

- **Don’t** use a dead-letter queue with standard queues when you want to be able to keep retrying the transmission of a message indefinitely. For example, don’t use a dead-letter queue if your program must wait for a dependent process to become active or available.

- **Don’t** use a dead-letter queue with a FIFO queue if you don’t want to break the exact order of messages or operations. For example, don’t use a dead-letter queue with instructions in an Edit Decision List (EDL) for a video editing suite, where changing the order of edits changes the context of subsequent edits.