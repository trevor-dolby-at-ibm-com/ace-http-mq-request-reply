# Sha1HostnameClient

This client sets the MQ CorrelId field to be a fixed value on the request message and asks
the back-end service to pass the same CorrelId back again on the reply message so that 
the MQInput node (which is configured to get only messages with that CorrelId) can receive
them successfully. 

The HTTP reply identifier is passed as the MsgId in the request message, with the back-end
service also being asked to pass it back again so the reply flow can provide it to the 
HTTPReply node.

![picture](/files/Sha1HostnameClient.png)

## Running this client

This client depends on
- Using ACE 12.0.6 or later
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
kubectl apply -n cp4i -f https://github.com/trevor-dolby-at-ibm-com/ace-http-mq-request-reply/raw/main/Sha1HostnameClient/IS-github-bar-Sha1HostnameClient.yaml
```
where the "cp4i" namespace can be changed as appropriate. 

`curl http://http-mq-sha1hostname-http-cp4i.apps.some.domain/Sha1HostnameClient` should 
work successfully regardless of the number of replicas.

Note that timeouts may occur if the NoCorrelationClient is deployed at the same
time due to it picking up messages intended for the other clients.
