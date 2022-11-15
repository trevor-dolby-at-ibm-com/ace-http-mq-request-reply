# CorrelIdClientUsingBody

This client sets the MQ CorrelId field to be a fixed value on the request 
message and asks the back-end service to pass the same CorrelId back again
on the reply message so that the MQInput node (which is configured to get
only messages with that CorrelId) can receive them successfully.

The HTTP reply identifier is passed as part of the message body with the
back-end service passing it back again in the same way reply flow can 
provide it to the HTTPReply node.

In this case, the CorrelId value is set to a user variable provided by a
startup script that derives the value from the HOSTNAME variable, which
should be uniqe for each container.

![picture](/files/CorrelIdClientUsingBody.png)


## Running this client

This client depends on
- A queue manager
- The [MQBackend](/MQBackend) service from this repo
- An MQEndpoint policy for the queue manager
- A server.conf.yaml stanze setting the policy to be the remote default queue manager
  and also specifying the startup script.

The queue manager can be created by following the instructions at 
https://community.ibm.com/community/user/integration/blogs/amar-shah1/2021/12/13/deploying-a-queue-manager-from-the-openshift-web-c
and the MQEndpoint is based on a follow-on blog article at 
https://community.ibm.com/community/user/integration/blogs/amar-shah1/2022/01/05/moving-an-integration-that-uses-ibm-mq-onto-contai
with the policy itself at https://github.com/amarIBM/hello-world/tree/master/Samples/Scenario-5 .

The server.conf.yaml setup is contained in [script-and-remote-mq.yaml](script-and-remote-mq.yaml) and
enables the MQEndpoint as a remote default queue manager while also specifying the script:
```
---
remoteDefaultQueueManager: '{ACEMQEndpoint}:RemoteMQ'
StartupScripts:
  EncodedHostScript:
    command: 'export ENCODED_VAR=`echo $HOSTNAME | sha1sum | tr -d "-" | tr -d " "` && echo UserVariables: && /bin/echo -e "  script-encoded-hostname: \\x27$ENCODED_VAR\\x27"' 
    readVariablesFromOutput: true
```

Once the dependencies are in place, this client can be deployed by running
```
kubectl apply -n cp4i -f https://github.com/trevor-dolby-at-ibm-com/ace-http-mq-request-reply/raw/main/CorrelIdClientUsingBody/IS-github-bar-CorrelIdClientUsingBody.yaml
```
where the "cp4i" namespace can be changed as appropriate. 

`curl http://http-mq-correlidclientusingbody-http-cp4i.apps.some.domain/CorrelIdClientUsingBody` should 
work successfully regardless of the number of replicas.

Note that timeouts may occur if the NoCorrelationClient is deployed at the same
time due to it picking up messages intended for the other clients.
