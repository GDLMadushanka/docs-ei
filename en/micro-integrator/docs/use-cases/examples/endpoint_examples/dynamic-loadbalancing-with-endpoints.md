# Dynamic Load Balancing between Three Nodes

### Introduction

This sample demonstrates how to use the dynamic load balancing mechanism
of the ESB to support load balancing between three nodes.

### Prerequisites

-   Enable clustering and group management in the
    `          <ESB_HOME>/repository/conf/axis2/axis2.xml         `
    file. To do this,  
    1.  Set the `            enable           ` attribute of the
        `            clustering           ` and
        `            groupManagement           ` elements to
        `            true           ` .
    2.  Provide the IP address of your machine as the value of the
        `            mcastBindAddress           ` and
        `            localMemberHost           ` parameters. For more
        information on clustering, see [WSO2 Clustering and Deployment
        Guide](https://docs.wso2.org/display/CLUSTER420/WSO2+Clustering+and+Deployment+Guide)
        .  
          
-   Enable clustering in the
    `          <ESB_HOME>/samples/axis2Server/repository/conf/axis2.xml         `
    file. To do this,  
    1.  Set the `            enable           ` attribute of the
        `            clustering           ` elements to
        `            true           ` .
    2.  Provide the IP address of your machine as the value of the
        `            mcastBindAddress           ` and
        `            localMemberHost           ` parameters.
    3.  Make sure that the `            applicationDomain           ` of
        the `            membershipHandler           ` is the same as
        the domain name specified in the
        `            axis2.xml           ` file of the Axis2 servers.  
          
-   For a list of general prerequisites, see [Prerequisites to Start the
    ESB
    Samples](https://docs.wso2.com/display/EI650/Setting+Up+the+ESB+Samples#SettingUptheESBSamples-ESBSamplePrerequisites)
    .

### Building the sample

The XML configuration for this sample is as follows:

``` 
    <definitions xmlns="http://ws.apache.org/ns/synapse">
        <sequence name="main" onError="errorHandler">
            <in>
                <send>
                    <endpoint name="dynamicLB">
                        <dynamicLoadbalance failover="true" algorithm="org.apache.synapse.endpoints.algorithms.RoundRobin">
                            <membershipHandler class="org.apache.synapse.core.axis2.Axis2LoadBalanceMembershipHandler">
                                <property name="applicationDomain" value="apache.axis2.application.domain"/>
                            </membershipHandler>
                        </dynamicLoadbalance>
                    </endpoint>
                </send>
                <drop/>
            </in>
            <out>
                <!-- Send the messages where they have been sent (i.e. implicit To EPR) -->
                <send/>
            </out>
        </sequence>
        <sequence name="errorHandler">
            <makefault response="true">
                <code value="tns:Receiver" xmlns:tns="http://www.w3.org/2003/05/soap-envelope"/>
                <reason value="COULDN'T SEND THE MESSAGE TO THE SERVER."/>
            </makefault>
            <send/>
        </sequence>
    </definitions>
```

This configuration file `         synapse_sample_57.xml        ` is
available in the `         <ESB_HOME>/repository/samples        `
directory.

**To build the sample**

1.  Start three instances of the sample Axis2 server on HTTP ports 9001,
    9002 and 9003 respectively and give unique names to each server. For
    instructions on starting the Axis2 server, see [Starting the Axis2
    server](https://docs.wso2.com/display/ESB500/Setting+Up+the+ESB+Samples#SettingUptheESBSamples-Axis2server)
    .

2.  Deploy the back-end service
    `           LoadbalanceFailoverService          ` . For instructions
    on deploying sample back-end services, see [Deploying sample
    back-end
    services](https://docs.wso2.com/display/EI650/Setting+Up+the+ESB+Samples#SettingUptheESBSamples-Backend)
    .

### Executing the sample

The sample client used here is the **Load Balance and Failover Client**
.

**To execute the sample client**

1.  Run the following command from the
    `           <ESB_HOME>/samples/axis2Client          ` directory.

    ``` bash
            ant loadbalancefailover -Di=100
    ```

2.  Run the client again using the above command without the
    `          -Di=100         ` parameter.
3.  While running the client, stop the server named MyServer1.
4.  Restart MyServer1.

### Analyzing the output

When the client is run for the first time, the client sends 100 requests
to the `         LoadbalanceFailoverService        ` through the ESB.
The ESB will distribute the load among the three nodes in a round-robin
manner. The `         LoadbalanceFailoverService        ` appends the
name of the server to the response, so that the client can determine
which server has processed the message.

When you analyze the output on the client console, you will see that the
requests are processed by three servers.

The output on the client console will be as follows:

``` java
    [java] Request: 1 ==> Response from server: MyServer1
    [java] Request: 2 ==> Response from server: MyServer2
    [java] Request: 3 ==> Response from server: MyServer3
    [java] Request: 4 ==> Response from server: MyServer1
    [java] Request: 5 ==> Response from server: MyServer2
    [java] Request: 6 ==> Response from server: MyServer3
    [java] Request: 7 ==> Response from server: MyServer1
    ...
```

When you run the client without the `         -Di=100        `
parameter, the client sends infinite requests. When you stop the server
named MyServer1 while running the client, you will see that the requests
are distributed only among MyServer2 and MyServer3.

You can see this by analyzing the log output on the client console,
which will be as follows:

``` java
    ...
    [java] Request: 61 ==> Response from server: MyServer1
    [java] Request: 62 ==> Response from server: MyServer2
    [java] Request: 63 ==> Response from server: MyServer3
    [java] Request: 64 ==> Response from server: MyServer2
    [java] Request: 65 ==> Response from server: MyServer3
    [java] Request: 66 ==> Response from server: MyServer2
    [java] Request: 67 ==> Response from server: MyServer3
    ...
```

By analysing the above output, you will understand that MyServer1 was
stopped after request 63. You can come to this conclusion because all
requests coming after request 63 are distributed only among MyServer2
and MyServer3.

When you restart MyServer1, you will see that the requests are now sent
again to all three servers.
