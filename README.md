#1.Introduction
##Prerequisites
To gain access to the system you must have a Monopond user account that is flagged with Fax API privileges. 
Please note some features described in this document will require access controls to be enacted. Examples of this include enabling specific destinations, filtering options and/or retry settings.

All of the above items can be organised through consulting with your account manager.

##Overview of the Monopond Fax API
This SOAP-based API brings you the ability to send faxes programmatically from your applications.

Using this API you are able to both send a single document to a single destination and, or broadcast a single document to multiple destinations. In either scenario you can manage fax related parameters individually, such as:

-Enabling support for specific blocking filters:

+ Australia: Do Not Call Register (DNCR)
+ UK: Fax Preference Service (FPS)

-Customising connection settings:

+ Transmitting Subscriber Identification (TSID) string to identify you as the sender of the fax.
+ The number of retries when attempting to connect.
+ Resolution setting of the fax document.
+ The header format and content that appears at the top of the transmitted fax.

-Scheduling options:

+ Specify the date and time you would like the transmission to start.

#2.API Access and General Usage
##API Web Service
The API is available as a SOAP Web Service. This service includes a WSDL which describes the functionality available in the interface, and can be used by many programming tools (such as Apache Axis/WSDL2Java and Microsoft Visual Studio) to generate client code without the need of programming a library, thus making the Monopond Fax API programming language-agnostic and OS-independent.

To access the API, use the following URLs:

Test WSDL: https://test.api.monopond.com/fax/soap/v2/?wsdl

Production WSDL: https://api.monopond.com/fax/soap/v2/?wsdl

#Building a Request
##SOAP Envelope 
If you are using the WSDL to generate your client you won’t need to worry about this per-se, but it is handy to know what is happening underneath the hood.

The SOAP envelope provides a wrapper around each API request, defining the XML document as a SOAP message. The namespace definitions in this XML element are required. If they are missing, the server will generate a fault and discard the request.

The envelope consists of two sections; a header and a body. The header contains authentication information and is explained in the next document section. The body contains the fax request information when making each function call and is described in the function definitions throughout this document.

An example of a SOAP envelope for the Monopond Fax API is shown below with the header and body information omitted.
```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:v2="https://api.monopond.com/fax/soap/v2">
    <soapenv:Header>
        ...
    </soapenv:Header>
    <soapenv:Body>
        ...
    </soapenv:Body>
</soapenv:Envelope>
```
##Authorisation Headers
The Monopond Fax API uses WS-Security to authorise users on the platform. The WS-Security specification allows users to authenticate against SOAP services using a variety of different models.
When connecting to the Monopond Fax API you must use the UsernameToken security token format whichformat, which authenticates based on your Monopond username and password.

Applying these security headers will differ based on your API integration/connection method.

When using the WSDL to generate your SOAP client you may use a WS-Security library from your programming language to apply the headers to this SOAP service. Below is an example of applying these headers in Java using WSS4J interceptors:

```java
Map<String,Object> outProps = new HashMap<String,Object>();
outProps.put(WSHandlerConstants.ACTION, WSHandlerConstants.USERNAME_TOKEN);
outProps.put(WSHandlerConstants.USER, ”username”);
outProps.put(WSHandlerConstants.PW_CALLBACK_REF, new CallbackHandler() {
	public void handle(Callback[] callbacks) throws IOException, UnsupportedCallbackException {
		WSPasswordCallback pc = (WSPasswordCallback) callbacks[0];
		pc.setPassword(“password”);
	}
});
outProps.put(WSHandlerConstants.PASSWORD_TYPE, WSConstants.PW_TEXT);
factory.getOutInterceptors().add(new WSS4JOutInterceptor(outProps));
```

Alternatively if you are sending raw XML to the API you will need to apply the security headers to the request yourself following the example below:

```xml
...
    <soapenv:Header>
        <wsse:Security soapenv:mustUnderstand="1">
            <wsse:UsernameToken>
                <wsse:Username>username</wsse:Username>
                <wsse:Password>password</wsse:Password>
            </wsse:UsernameToken>
      </wsse:Security>
    </soapenv:Header>
...
```
#3.Function Definitions
##SendFax
###Description
This is the core function in the API allowing you to send faxes on the platform. 

Your specific faxing requirements will dictate which send request type below should be used. The two common use cases would be the sending of a single fax document to one destination and the sending of a single fax document to multiple destinations.

###Sending a single fax:
To send a fax to a single destination a request similar to the following example can be used:

```xml
<v2:SendFaxRequest>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
            <SendFrom>Test Fax</SendFrom>
            <Resolution>normal</Resolution>
            <Documents>
                <Document>
                    <FileName>test.txt</FileName>
                    <FileData>VGhpcyBpcyBhIGZheA==</FileData>
                </Document>
            </Documents>
        </FaxMessage>
    </FaxMessages>
</v2:SendFaxRequest>
```

###Sending multiple faxes:
To send faxes to multiple destinations a request similar to the following example can be used. Please note the addition of another “FaxMessage”:

```xml
<v2:SendFaxRequest>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
            <SendFrom>Test Fax</SendFrom>
            <Documents>
                <Document>
                    <FileName>test.txt</FileName>
                    <FileData>VGhpcyBpcyBhIGZheA==</FileData>
                </Document>
            </Documents>
        </FaxMessage>
        <FaxMessage>
            <MessageRef>test-1-1-2</MessageRef>
            <SendTo> 6022222222</SendTo>
            <SendFrom>Test Fax</SendFrom>
            <Documents>
                <Document>
             <FileName>test.txt</FileName>
                    <FileData>VGhpcyBpcyBhIGZheA==</FileData>
                </Document>
            </Documents>
        </FaxMessage>
    </FaxMessages>
</v2:SendFaxRequest>
```

###Sending faxes to multiple destinations with the same document (broadcasting):
To send the same fax content to multiple destinations (broadcasting) a request similar to the example below can be used.

This method is recommended for broadcasting as it takes advantage of the multiple tiers in the send request. This eliminates the repeated parameters out of the individual fax message elements which are instead inherited from the parent send fax request. An example below shows “SendFrom” being used for both FaxMessages. While not shown in the example below further control can be achieved over individual fax elements to override the parameters set in the parent.

```xml
<v2:SendFaxRequest>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
        </FaxMessage>
        <FaxMessage>
            <MessageRef>test-1-1-2</MessageRef>
            <SendTo>6022222222</SendTo>
        </FaxMessage>
    </FaxMessages>
    <Documents>
        <Document>
            <FileName>test.txt</FileName>
            <FileData>VGhpcyBpcyBhIGZheA==</FileData>
        </Document>
    </Documents>
    <SendFrom>Test Fax</SendFrom>
</v2:SendFaxRequest>
```

When sending multiple faxes in batch it is recommended to group them into requests of around 600 fax messages for optimal performance. If you are sending the same document to multiple destinations it is strongly advised to only attach the document once in the root of the send request rather than attaching a document for each destination.

For detailed examples, see Section 6 of this document. of this document.Request

###SendFaxRequest Parameters:
**Name**|**Required**|**Type**|**Description**|**Default**
-----|-----|-----|-----|-----
**BroadcastRef**||String|Allows the user to tag all faxes in this request with a user-defined broadcastreference. These faxes can then be retrieved at a later point based on this reference.|
**SendRef**||String|Similar to the BroadcastRef, this allows the user to tag all faxes in this request with a send reference. The SendRef is used to represent all faxes in this request only, so naturally it must be unique.|
**FaxMessages**|**X**|FaxMessages describe each individual fax message and its destination. See below for details.|
**SendFrom**||Alphanumeric String|A customisable string used to identify the sender of the fax. Also known as the Transmitting Subscriber Identification (TSID). The maximum string length is 32 characters|Fax
**Documents**|**X**|Array of FaxDocument|Each FaxDocument object describes a fax document to be sent. Multiple documents can be defined here which will be concatenated and sent in the same message. See below for details.|
**Resolution**||Resolution|A customisable string used to identify the sender of the fax. Also known as the Transmitting Subscriber Identification (TSID). The maximum string length is 32 characters|normal
**ScheduledStartTime**||DateTime|The date and time the transmission of the fax will start.|Current time (immediate sending)
**Blocklists**||Blocklists|The blocklists that will be checked and filtered against before sending the message. See below for details.WARNING: This feature is inactive and non-functional in this (2.0.1) version of the Fax API.|Current time (immediate sending)
**Retries**||Unsigned Integer|The number of times to retry sending the fax if it fails. Each account has a maximum number of retries that can be changed by consultation with your account manager.|Account Default
**BusyRetries**||DateTime|Certain fax errors such as “NO_ANSWER” or “BUSY” are not included in the above retries limit and can be set separately. Each account has a maximum number of busy retries that can be changed by consultation with your account manager.|Account default
**HeaderFormat**||String|Allows the header format that appears at the top of the transmitted fax to be changed. See below for an explanation of how to format this field.| From: X, To: X



------------- kreigh starts here ---------------------
##FaxStatus
###Description

This function provides you with a method of retrieving the status, details and results of fax messages sent. While this is a legitimate method of retrieving results we strongly advise that you take advantage of our callback service (see Section 4), which will push these fax results to you as they are completed.

When making a status request, you must provide at least a BroadcastRef, SendRef or MessageRef. The 
function will also accept a combination of these to further narrow the request query.
- Limiting by a BroadcastRef allows you to retrieve faxes contained in a group of send requests.
- Limiting by SendRef allows you to retrieve faxes contained in a single send request.
- Limiting by MessageRef allows you to retrieve a single fax message.

There are multiple levels of verbosity available in the request; these are explained in detail below. You can also find full examples in Section 6 of this document.

###Request
**FaxStatusRequest Parameters:**

| **Name** | **Required** | **Type** | **Description** | **Default** |
|--- | --- | --- | --- | ---|
|**BroadcastRef**|  | *String* | User-defined broadcast reference. | |
|**SendRef**|  | *String* | User-defined send reference. | |
|**MessageRef**|  | *String* | User-defined message reference. | |
|**Verbosity**|  | *String* | Verbosity String The level of detail in the status response. Please see below for a list of possible values.| |

**Verbosity Levels:**	
  
| **Value** | **Description** |
| --- | --- |
| **brief** | Gives you an overall view of the messages. This simply shows very high-level statistics, consisting of counts of how many faxes are at each status (i.e. processing, queued,sending) and totals of the results of these faxes (success, failed, blocked). |
| **send** | send Includes the results from ***“brief”*** while also including an itemised list of each fax message in the request. |
| **details** | details Includes the results from ***“send”*** along with details of the parameters used to send the fax messages. |
| **results** |Includes the results from ***“send”*** along with the sending results of the fax messages. |
| **all** | all Includes the results from both ***“details”*** and ***“results”*** along with some extra uncommon fields. |

###Response
The response received depends entirely on the verbosity level specified.

**FaxStatusResponse:**

| Name | Type | Verbosity | Description |
| --- | --- | --- | --- |
| **FaxStatusTotals** | *FaxStatusTotals* | *brief* | Counts of how many faxes are at each status. See below for more details. |
| **FaxResultsTotals** | *FaxResultsTotals* | *brief* | FaxResultsTotals FaxResultsTotals brief Totals of the end results of the faxes. See below for more details. |
| **FaxMessages** | *Array of FaxMessage* | *send* | send List of each fax in the query. See below for more details. |

**FaxStatusTotals:**

Contains the total count of how many faxes are at each status. 
To see more information on each fax status, view the FaxStatus table below.

| Name | Type | Verbosity | Description |
| --- | --- | --- | --- |
| **pending** | *Long* | *brief* | Fax is pending on the system and waiting to be processed.|
| **processing** | *Long* | *brief* | Fax is in the initial processing stages. |
| **queued** | *Long* | *brief* | Fax has finished processing and is queued, ready to send out at the send time. |
| **starting** | *Long* | *brief* | Fax is ready to be sent out. |
| **sending** | *Long* | *brief* | Fax has been spooled to our servers and is in the process of being sent out. |
| **pausing** | *Long* | *brief* | Fax has been told to pause. |
| **paused** | *Long* | *brief* | Fax is currently paused. |
| **resuming** | *Long* | *brief* | Fax has been told to resume. After the resume has been confirmed, it is set back to the “sending” status. |
| **stopping** | *Long* | *brief* | Fax has been told to stop. After the stop has been confirmed, it is set to the “finalizing” status. |
| **finalizing** | *Long* | *brief* | Fax has finished sending and the results are being processed.|
| **done** | *Long* | *brief* | Fax has completed and no further actions will take place. The detailed results are available at this status. |

**FaxResultsTotals:**

Contains the total count of how many faxes ended in each result, as well as some additional totals. To view more information on each fax result, view the FaxResults table below.

| Name | Type | Verbosity | Description |
| --- | --- | --- | --- |
| **success** | *Long* | *brief* | Fax has successfully been delivered to its destination.|
| **blocked** | *Long* |  *brief* | Destination number was found in one of the block lists. |
| **failed** | *Long* | *brief* | Fax failed getting to its destination.|
| **totalAttempts** | *Long* | *brief* |Total attempts made in the reference context.|
| **totalFaxDuration** | *Long* | *brief* |totalFaxDuration Long brief Total time spent on the line in the reference context.|
| **totalPages** | *Long* | *brief* | Total pages sent in the reference context.|

**FaxMessages:**

| Name | Type | Verbosity | Description |
| --- | --- | --- | --- |
| **messageRef** | *String* | *send* | |
| **sendRef** | *String* | *send* | |
| **broadcastRef** | *String* | *send* | |
| **sendTo** | *String* | *send* | |
| **status** |  | *send* | The current status of the fax message. See the FaxStatus table above for possible status values. |
| **FaxDetails** | *FaxDetails* | *details* | Contains the details and settings the fax was sent with. See below for more details. |
| **FaxResults** | *FaxResults Array of FaxResult* | *results* | Contains the results of each attempt at sending the fax message and their connection details. See below for more details. |

