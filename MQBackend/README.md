# MQBackend

This service provides a common back-end service for the clients in this repo.

The MQReply node honors the MQMD “Report” options (described at 
https://www.ibm.com/docs/en/ibm-mq/9.3?topic=mqmd-report-mqlong) and so clients
can decide how they would like the reply identifiers to be set. Different report
options (such as MQRO_PASS_CORREL_ID) will result in different reply messages
without the back-end flow itself needing to change, which means the examples 
below can all share the same back-end. 

The flow itself does not modify the MQMD at all, and the Compute node creates
the reply body only. The reply body is a JSON message with the relevant MQMD
fields copied in, plus the request message body and the application name.

![picture](/files/MQBackend.png)

## Running this service

This service depends on
- A queue manager
- An MQEndpoint policy for the queue manager
- A server.conf.yaml stanze setting the policy to be the remote default queue manager

The queue manager can be created by following the instructions at 
https://community.ibm.com/community/user/integration/blogs/amar-shah1/2021/12/13/deploying-a-queue-manager-from-the-openshift-web-c
and the policies are based on a follow-on blog article at 
https://community.ibm.com/community/user/integration/blogs/amar-shah1/2022/01/05/moving-an-integration-that-uses-ibm-mq-onto-contai
with the policies themselves at https://github.com/amarIBM/hello-world/tree/master/Samples/Scenario-5

Once the dependencies are in place, this service can be deployed by running
```
kubectl apply -n cp4i -f https://github.com/trevor-dolby-at-ibm-com/ace-http-mq-request-reply/raw/main/MQBackend/IS-github-bar-MQBackend.yaml
```
where the "cp4i" namespace can be changed as appropriate. 

Once the service is deployed successfully, the clients in this repo can be run to
demonstrate success or failure.
