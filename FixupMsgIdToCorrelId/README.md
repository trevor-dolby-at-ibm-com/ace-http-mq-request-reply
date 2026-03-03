# FixupMsgIdToCorrelId

This application exists to provide a way to allow container clients such as 
[Sha1HostnameWithTimeoutClient](../Sha1HostnameWithTimeoutClient) to interact with back-end
services that only handle the MsgId-to-CorrelId pattern and will not preserve both MsgId
and CorrelId. This is achieved by putting a pair of flows between the client and the
service in order to store the relevant metadata on a queue so it can be restored on reply:

![picture](/files/fixup-request-reply.png)

This implementation does not rely on anything other than the service returning the 
request MsgId as the CorrelId in the reply, and deliberately sets the request CorrelId
to MQMI_NONE and sets `Report = MQRO_COPY_MSG_ID_TO_CORREL_ID` to demonstrate the lack
of reliance on anything but the MsgId. There are many different IDs in flight at once 
and so trace nodes have been left in the flow at helpful points to aid understanding.

![picture](/files/fixup-msgid-to-correlid-outbound.png)

The outbound flow is designed to be service-agnostic, and other input nodes could be 
added to the flow without needing to change anything else as long as the "real" back-end
queue name can be determined from the "fix" input queue name. In this example, the "fix"
queue is called "BACKEND.SHARED.FIX" and the "real" queue is "BACKEND.SHARED.INPUT" and 
so the conversion is simple. The ReplyToQ is set to be a shared queue that can handle
any reply from any service as long there is a matching message on the match queue.

The various MQ nodes are all part of the same transaction to ensure all of the messages
are put on the correct queues and nothing goes missing. This works because MQ itself 
assigns the MsgId for the back-end request message before it has been committed, so the
match message setup code can use it to ensure the reply flow can find the match.

Reply handling uses the match queue message information to set the correct MsgId, CorrelId,
ReplyToQ, and ReplyToQMgr for the "real" reply back to the original client:

![picture](/files/fixup-msgid-to-correlid-reply.png)

The [ReplyFixupFlow_SetupMQGetParms](ReplyFixupFlow_SetupMQGetParms.esql) Compute node 
sets up one value for the MQGet node:
```
SET OutputLocalEnvironment.MatchMQMD.CorrelId = InputRoot.MQMD.CorrelId;
```
and the MQGet node puts the resulting message in the LocalEnvironment so the relevant
values can be used to construct the final reply in [CreateReply](ReplyFixupFlow_CreateReply.esql)

Notes:
- The MQGet node is designed to be able to leave the Message tree untouched and place the
  message data into the LocalEnvironment in order to make this sort of scenario simpler.
- WrittenDestination data fields use different capitalization for MQMD fields, and it's 
  easy to get confused. For example, `SET OutputRoot.MQMD.CorrelId = InputLocalEnvironment.WrittenDestination.MQ.DestinationData.msgId;` 
  in [OutboundFixupFlow_CreateMatchMessage.esql](OutboundFixupFlow_CreateMatchMessage.esql)
  shows `msgId` with a lower-case "m" while the MQMD parser would use upper-case.
- Expiry for the match messages needs to be set high enough that timeouts shoulw not occur
  during normal operation. For HTTP-based scenarios, the expiry can be set to the same
  value as the HTTP Input node timeout because the HTTP flow will not be able to send a
  response to the client after that point so there is no need to find the match message.

## Running this application

This application depends on
- Using ACE 12.0.6 or later
- A queue manager
- The [MQBackend](/MQBackend) service from this repo
- An MQEndpoint policy for the queue manager
- A server.conf.yaml stanze setting the policy to be the remote default queue manager
