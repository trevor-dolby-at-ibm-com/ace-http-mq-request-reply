# NoCorrelationWithPrototypeDQClient (prototype!)

**Warning! This is prototype code and not to be used in production!**

This client sends messages to the MQBackend service without any correlation ID being set, but
does set the ReplyToQ to a dynamic queue created by the server so the replies will be received
by the correct flow.

![picture](/files/NoCorrelationClientWithPrototypeDQ.png)

## Running this client

This client depends on
- A queue manager
- The [MQBackend](/MQBackend) service from this repo
- An MQEndpoint policy for the queue manager
- A server.conf.yaml stanze setting the policy to be the remote default queue manager

It also depends on server.conf.yaml containing
```
DynamicQueues:
  NoCorrelationTDQueue:
    modelQueue: 'SYSTEM.BROKER.MODEL.QUEUE'
    pattern: 'NOCORRELATION.TDQ.*'
    deleteOnShutdown: false
    purgeOnDelete: false
```
and the queue manager having the ACE-defined `SYSTEM.BROKER.MODEL.QUEUE` for the server to use.

If the server has been configured correctly, messages should appear at server startup:
```
2026-02-09 21:46:43.974156: BIP6130I: Creating IBM MQ dynamic queue using model queue 'SYSTEM.BROKER.MODEL.QUEUE' and DynamicQName 'NOCORRELATION.TDQ.*'.
2026-02-09 21:46:44.006548: BIP6131I: IBM MQ dynamic queue 'NOCORRELATION.TDQ.69828447218CBD01' created and user variable '[iib.user-mq-dynamic-queue-NoCorrelationTDQueue]' set.
```
