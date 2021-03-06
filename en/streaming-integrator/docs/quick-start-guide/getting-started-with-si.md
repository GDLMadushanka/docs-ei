# Getting Started with Streaming Integrator in Five Minutes

## Introduction

This quick start guide gets you started with the Streaming Integrator (SI), in just 5 minutes.

In this guide, you will download the SI distribution, start it and then try out a simple Siddhi application.

## Tutorial Outline

- [Downloading Streaming Integrator](#downloading-streaming-integrator)
- [Starting the server](#starting-the-server)
- [Deploying a simple Siddhi app](#deploying-a-simple-siddhi-app)

## Downloading Streaming Integrator

Download the Streaming Integrator distribution from TODO and extract it to a location of your choice. Hereafter, the extracted location is referred to as `<SI_HOME>`.

## Starting the server

Navigate to the `<SI_HOME>/bin` directory in the console and issue one of the following commands: <br/>
    - For Windows: `server.bat`
    - For Linux: `./server.sh`

## Deploying a simple Siddhi application

Let's create a simple Siddhi application that receives an HTTP message, does a simple transformation to the message, and then publishes the output to the SI console and to a user-specified file.
![Scenario](../images/quick-start-guide-101/quick-start.png)
Open a text file and copy-paste following Siddhi application into it.

```
@App:name('MySimpleApp')

@App:description('Receive events via HTTP transport and view the output on the console')

@Source(type = 'http', receiver.url='http://localhost:8006/productionStream', basic.auth.enabled='false',
    @map(type='json'))
define stream SweetProductionStream (name string, amount double);

@sink(type='file', @map(type='xml'), file.uri='/Users/foo/low_productions.txt')
@sink(type='log')
define stream TransformedProductionStream (nameInUpperCase string, roundedAmount long);

-- Simple Siddhi query to transform the name to upper case.
from SweetProductionStream
select str:upper(name) as nameInUpperCase, math:round(amount) as roundedAmount
insert into TransformedProductionStream;
```

Note: The output of this application will be written into a the file, specified in parameter `file.uri`.  Change this parameter accordingly. 

Save this file as `MySimpleApp.siddhi` in the `<SI_HOME>/wso2/server/deployment/siddhi-files` directory.

!!info
    Once you deploy the above Siddhi application, it creates a new `HTTP` endpoint at `http://localhost:8006/productionStream` and starts listening to the endpoint for incoming messages. The incoming messages will then be published to,
    1. Streaming Integrator logs
    2. User-specified file, in XML format
    The next step is to publish a message to the endpoint that we created, via a CURL command.

Now execute following `CURL` command on the console.
```
curl -X POST -d "{\"event\": {\"name\":\"sugar\",\"amount\": 20.5}}"  http://localhost:8006/productionStream --header "Content-Type:application/json"
```  
Notice that you published a message with a lower case name: `sugar`. 

However, the output you observe in the SI console is similar to following.
```
INFO {io.siddhi.core.stream.output.sink.LogSink} - MySimpleApp : TransformedProductionStream : Event{timestamp=1563539561686, data=[SUGAR, 21], isExpired=false}
```
Notice that the output message has an uppercase name: `SUGAR`. In addition to that, the `amount` has being rounded. This is because of the simple message transformation carried out by the Siddhi application.

In addition to this, open `low_productions.txt` file (this is the file which you specified using the parameter `file.uri`). You will notice following content inside the file.
```
<events><event><nameInUpperCase>SUGAR</nameInUpperCase><roundedAmount>21</roundedAmount></event></events>
``` 
 