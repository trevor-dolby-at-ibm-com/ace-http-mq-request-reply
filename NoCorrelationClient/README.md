# NoCorrelationClient

This client sends messages to the MQBackend service without making any effort to ensure that
the replies are received by the correct server, and therefore will run into issues if scaled
beyond one replica. To demonstrate the problem, scale the server to more than one replica and 
use curl to send requests: many will time out, producing erros in the server logs about invalid
reply identifiers:
```
2022-11-15 02:27:11.345673: BIP2230E: Error detected whilst processing a message in node 'NoCorrelation.HTTP Reply'.
2022-11-15 02:27:11.345724: BIP3740E: An attempt was made to use an invalid EVHT message identifier 45564854000000000000000017a51c79c500000000000000.
```

![picture](/files/NoCorrelationClient.png)

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
kubectl apply -n cp4i -f https://github.com/trevor-dolby-at-ibm-com/ace-http-mq-request-reply/raw/main/NoCorrelationClient/IS-github-bar-NoCorrelationClient.yaml
```
where the "cp4i" namespace can be changed as appropriate. 

`curl http://http-mq-correlidclient-http-cp4i.apps.some.domain/NoCorrelation` should time out 
frequently and produce errors in the server logs.

The YAML file can be downloaded and changed as needed; the default number of replicas
is 2, but this can be changed. One replica should not produce errors unless deployed
alongside other clients from this repo; this client will also cause issues for the 
other clients due to it picking up messages intended for the other clients.
