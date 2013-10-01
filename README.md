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



