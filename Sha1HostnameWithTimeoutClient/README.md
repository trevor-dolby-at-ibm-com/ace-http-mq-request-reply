# Sha1HostnameWithTimeoutClient

This client sets the MQ CorrelId field to be a fixed value on the request message and asks
the back-end service to pass the same CorrelId back again on the reply message so that 
the MQInput node (which is configured to get only messages with that CorrelId) can receive
them successfully. 

This version of the Sha1Hostname client allows for explicit control of timeout behaviour
in the same way the SyncClient does but without the performance and scaling issues of the
synchronous flow. The Group nodes autmatically restore the HTTP request identifier (and 
do the same for SOAP nodes) and also handle timeouts automatically (including sub-second
timeout values). They also preserve user context for use in the nodes downstream from the
GroupComplete node.

One aspect of using the Group nodes is that they expect the outbound MsgId to be returned
to the GroupGather node in the CorrelId field. This application is relying on the CorrelId
to allow the reply messages to find their way to the correct server, so the value needed
by the Group nodes is actually in the MsgId field (because the `Create_Outbound_Message` 
Compute node specified `Report = MQRO_PASS_CORREL_ID + MQRO_PASS_MSG_ID`). This is fixed 
by the `Fix_reply_ID` Compute node, which copies the received MsgId to the CorrelId field.

![picture](/files/Sha1HostnameWithTimeoutClient.png)

## Running this client

This client depends on
- Using ACE 12.0.6 or later
- A queue manager
- The [MQBackend](/MQBackend) service from this repo
- An MQEndpoint policy for the queue manager
- A server.conf.yaml stanze setting the policy to be the remote default queue manager

Note that timeouts may occur if the NoCorrelationClient is deployed at the same
time due to it picking up messages intended for the other clients.
