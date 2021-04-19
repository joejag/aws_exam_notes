# Applications

## SQS

- Oldest service. Webservice gives access to a message queue
- The queue is temporary while they await processing
- decouples components with a fail-safe queue
- upto 256k text in queue, upto 2Gb in S3 if you go over
- consumeres can be inttermitedtly connected to the network
- Two types:
  - Standard Queues
    - Nearly unlimited per second, at-least-once delivery. Order not guaranteed, best efforts
  - FIFO Queues
    - order and once delivery
    - 300 per second max
    - message groups
- Summary:
  - Pull based
  - 256k size
  - Messages kept for 1m-14 days. Default is 4 days
  - Visibility timeout: how long does the message disappear from when it is picked up
    - could cause double processing if timeout is too low
    - max 12 hours for timeout
  - guaranteed at least once processing
  - SQS Long Polling: Doesn't return anything until a message appears of a timeout. Default can get empty
    - reduces charges since you aren't generating as much traffic

## Simple Workflow Service (SWF)

- coordinates work across a bunch of components
- task based - code with manual human steps
- Amazon use this inside their warehouse when picking things
- vs SQS
  - Retention: SQS:14d, SWF:365d
  - SWF is task orientated, SQS is message orientated
  - SWF tasks are assigned and never duplciated. SQS you must handle this and not handled twice
  - SWF tracks all tasks in an App. SQS you need to do this yourself if you have multiple queues
- SWF Actors:
  - Workflow Starters (could be app)
  - Deciders (controls flow)
  - Activity Workers

## Simple Notification Service (SNS)

-
