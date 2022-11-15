# SyncClient

This client sends a message to the MQBackend service with no specific message or 
correlation IDs, but does ask the service to copy the MsgId value of the request
message into the CorrelId field of the reply. The MQGet node in the flow is asked to
only get messages with that ID, and then propagates the response to the HTTPReply node.

The HTTP reply identifier maintained on the thread for the flow, and so no processing
is needed to ensure the HTTPReply can find the correct client.

Note that the client flow will block in the MQGet until the reply message is received.
This is potentially a very resource-intensive way to implement request-reply messaging,
as every message will block a thread for as long as it takes the back-end to reply, 
and each thread will consume storage while running.

![picture](/files/SyncClient.png)

## Running this client

This client depends on
- A queue manager
- The [MQBackend](/MQBackend) service from this repo
- An MQEndpoint policy for the queue manager
- A server.conf.yaml stanze setting the policy to be the remote default queue manager

The queue manager can be created by following the instructions at 
https://community.ibm.com/community/user/integration/blogs/amar-shah1/2021/12/13/deploying-a-queue-manager-from-the-openshift-web-c
and the policies are based on a follow-on blog article at 
https://community.ibm.com/community/user/integration/blogs/amar-shah1/2022/01/05/moving-an-integration-that-uses-ibm-mq-onto-contai
with the policies themselves at https://github.com/amarIBM/hello-world/tree/master/Samples/Scenario-5

Once the dependencies are in place, this client can be deployed by running
```
kubectl apply -n cp4i -f https://github.com/trevor-dolby-at-ibm-com/ace-http-mq-request-reply/raw/main/SyncClient/IS-github-bar-SyncClient.yaml
```
where the "cp4i" namespace can be changed as appropriate. 

`curl http://http-mq-syncclient-http-cp4i.apps.some.domain/SyncClient` should 
work successfully regardless of the number of replicas.

Note that timeouts may occur if the NoCorrelationClient is deployed at the same
time due to it picking up messages intended for the other clients.
