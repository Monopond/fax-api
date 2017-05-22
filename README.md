# 1.Introduction
## Prerequisites
To gain access to the system you must have a Monopond user account that is flagged with Fax API privileges. 
Please note some features described in this document will require access controls to be enacted. Examples of this include enabling specific destinations, filtering options and/or retry settings.

All of the above items can be organised through consulting with your account manager.

## Overview of the Monopond Fax API
This SOAP-based API brings you the ability to send faxes programmatically from your applications.

Using this API you are able to both send a single document to a single destination and, or broadcast a single document to multiple destinations. In either scenario you can manage fax related parameters individually, such as:

Enabling support for specific blocking filters:

+ Australia: Do Not Call Register (DNCR)
+ UK: Fax Preference Service (FPS)

Customising connection settings:

+ Transmitting Subscriber Identification (TSID) string to identify you as the sender of the fax.
+ The number of retries when attempting to connect.
+ Resolution setting of the fax document.
+ The header format and content that appears at the top of the transmitted fax.

Scheduling options:

+ Specify the date and time you would like the transmission to start.

# 2.API Access and General Usage
## API Web Service
The API is available as a SOAP Web Service. This service includes a WSDL which describes the functionality available in the interface, and can be used by many programming tools (such as Apache Axis/WSDL2Java and Microsoft Visual Studio) to generate client code without the need of programming a library, thus making the Monopond Fax API programming language-agnostic and OS-independent.

To access the API, use the following URLs:

Test WSDL: https://test.api.monopond.com/fax/soap/v2.1/?wsdl

Production WSDL: https://api.monopond.com/fax/soap/v2.1/?wsdl

# Building a Request
## SOAP Envelope 
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
## Authorisation Headers
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
# 3.Function Definitions
## SendFax
### Description
This is the core function in the API allowing you to send faxes on the platform. 

Your specific faxing requirements will dictate which send request type below should be used. The two common use cases would be the sending of a single fax document to one destination and the sending of a single fax document to multiple destinations.

### Sending a single fax:
To send a fax to a single destination a request similar to the following example can be used:

```xml
<v2:SendFaxRequest>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
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

You can visit the following properties of Document, FaxMessage, and SendFaxRequest to know its definitions:
* [FaxDocument Parameters](#faxdocument-parameters)
* [FaxMessage Parameters](#faxmessage-parameters)
* [SendFaxRequest Parameters](#sendfaxrequest-parameters)

### Sending a Fax with Retries inside a FaxMessage
To set-up a fax to have retries a request similar to the following example can be used.
```
<v2:SendFaxRequest>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
            <Retries>2</Retries>
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

You can visit the following properties of Document and FaxMessage to know its definitions:
* [FaxDocument Parameters](#faxdocument-parameters)
* [FaxMessage Parameters](#faxmessage-parameters)

### Sending a Fax with Retries inside a SendFaxRequest
To set-up a fax to have retries a request similar to the following example can be used.
```
<v2:SendFaxRequest>
    <Retries>2</Retries>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
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

You can visit the following properties of Document and SendFaxRequest to know its definitions:
* [FaxDocument Parameters](#faxdocument-parameters)
* [SendFaxRequest Parameters](#sendfaxrequest-parameters)

### Sending a Fax with BusyRetries inside a FaxMessage
To set-up a fax to have busyRetries a request similar to the following example can be used.
```
<v2:SendFaxRequest>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
            <BusyRetries>2</BusyRetries>
            <Documents>
                <Document>
                    <FileName>test.txt</FileName>
                    <FileData>VGhpcyBpcyBhIGZheA==</FileData>
                </Document>
            </Documents>
        </FaxMessage>
    </FaxMessages>
</v2:SendFaxRequest>

You can visit the following properties of Document and FaxMessage to know its definitions:
* [FaxDocument Parameters](#faxdocument-parameters)
* [FaxMessage Parameters](#faxmessage-parameters)

```
### Sending a Fax with BusyRetries inside a SendFaxRequest
To set-up a fax to have busyRetries a request similar to the following example can be used.
```
<v2:SendFaxRequest>
	<BusyRetries>2</BusyRetries>
    <FaxMessages>
        <FaxMessage>
            <MessageRef>test-1-1-1</MessageRef>
            <SendTo>6011111111</SendTo>
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
You can visit the following properties of Document and SendFaxRequest to know its definitions:
* [FaxDocument Parameters](#faxdocument-parameters)
* [SendFaxRequest Parameters](#sendfaxrequest-parameters)

### SendFaxRequest Parameters

**Name**|**Required**|**Type**|**Description**|**Default**
-----|-----|-----|-----|-----
**BroadcastRef**||String|Allows the user to tag all faxes in this request with a user-defined broadcastreference. These faxes can then be retrieved at a later point based on this reference.|
**SendRef**||String|Similar to the BroadcastRef, this allows the user to tag all faxes in this request with a send reference. The SendRef is used to represent all faxes in this request only, so naturally it must be unique.|
**FaxMessages**|**X**| Array of FaxMessage |FaxMessages describe each individual fax message and its destination. See below for details.|
**SendFrom**||Alphanumeric String|A customisable string used to identify the sender of the fax. Also known as the Transmitting Subscriber Identification (TSID). The maximum string length is 20 characters|Fax
**Documents**|**X**|Array of FaxDocument|Each FaxDocument object describes a fax document to be sent. Multiple documents can be defined here which will be concatenated and sent in the same message. See below for details.|
**Resolution**||Resolution|Resolution setting of the fax document. Refer to the resolution table below for possible resolution values.|normal
**ScheduledStartTime**||DateTime|The date and time the transmission of the fax will start.|Current time (immediate sending)
**Blocklists**||Blocklists|The blocklists that will be checked and filtered against before sending the message. See below for details. |
**Retries**||Unsigned Integer|The number of times to retry sending the fax if it fails. Each account has a maximum number of retries that can be changed by consultation with your account manager.|Account Default
**BusyRetries**||Unsigned Integer|Certain fax errors such as “NO_ANSWER” or “BUSY” are not included in the above retries limit and can be set separately. Each account has a maximum number of busy retries that can be changed by consultation with your account manager.|Account default
**HeaderFormat**||String|Allows the header format that appears at the top of the transmitted fax to be changed. See below for an explanation of how to format this field.| From: X, To: X
**MustBeSentBeforeDate** | | DateTime |  Specifies a time the fax must be delivered by. Once the specified time is reached the fax will be cancelled across the system. | 
**MaxFaxPages** | | Unsigned Integer |  Sets a limit on the amount of pages allowed in a single fax transmission. Especially useful if the user is blindly submitting their customer's documents to the platform. | 20

### FaxMessage Parameters
This represents a single fax message being sent to a destination.

**Name** | **Required** | **Type** | **Description** | **Default** 
-----|-----|-----|-----|-----
**MessageRef** | **X** | String | A unique user-provided identifier that is used to identify the fax message. This can be used at a later point to retrieve the results of the fax message. |
**SendTo** | **X** | String | The phone number the fax message will be sent to. |
**SendFrom** | | Alphanumeric String | A customisable string used to identify the sender of the fax. Also known as the Transmitting Subscriber Identification (TSID). The maximum string length is 32 characters | Empty
**Documents** | **X** | Array of FaxDocument | Each FaxDocument object describes a fax document to be sent. Multiple documents can be defined here which will be concatenated and sent in the same message. See below for details. | 
**Resolution** | | Resolution|Resolution setting of the fax document. Refer to the resolution table below for possible resolution values.| normal
**ScheduledStartTime** | | DateTime | The date and time the transmission of the fax will start. | Start now
**Blocklists** | | Blocklists | The blocklists that will be checked and filtered against before sending the message. See below for details. |
**Retries** | | Unsigned Integer | The number of times to retry sending the fax if it fails. Each account has a maximum number of retries that can be changed by consultation with your account manager. | Account Default
**BusyRetries** | | Unsigned Integer | Certain fax errors such as “NO_ANSWER” or “BUSY” are not included in the above retries limit and can be set separately. Please consult with your account manager in regards to maximum value.|account default
**HeaderFormat** | | String | Allows the header format that appears at the top of the transmitted fax to be changed. See below for an explanation of how to format this field. | From： X, To: X
**MustBeSentBeforeDate** | | DateTime |  Specifies a time the fax must be delivered by. Once the specified time is reached the fax will be cancelled across the system. | 
**MaxFaxPages** | | Unsigned Integer |  Sets a limit on the amount of pages allowed in a single fax transmission. Especially useful if the user is blindly submitting their customer's documents to the platform. | 20
**CLI**| | String| Allows a customer called ID. Note: Must be enabled on the account before it can be used.


### FaxDocument Parameters
Represents a fax document to be sent through the system. Supported file types are: PDF, TIFF, PNG, JPG, GIF, TXT, PS, RTF, DOC, DOCX, XLS, XLSX, PPT, PPTX.

**Name**|**Required**|**Type**|**Description**|**Default**
-----|-----|-----|-----|-----
**FileName**|**X**|String|The document filename including extension. This is important as it is used to help identify the document MIME type.|
**FileData**|**X**|Base64|The document encoded in Base64 format.|
**Order** | | Integer|If multiple documents are defined on a message this value will determine the order in which they will be transmitted.|0
**DitheringTechnique** | | FaxDitheringTechnique | Applies a custom dithering method to their fax document before transmission. | 
**DocMergeData**|||An Array of MergeFields|
**StampMergeData**|||An Array of MergeFields|

