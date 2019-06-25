## Monopond Outgoing Fax API

### Overview
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

### Outgoing Fax API Usage
**For details on using the API, please see the [Outgoing Fax API page](https://github.com/Monopond/fax-api/wiki/Outgoing-Fax-SOAP-API).**

## Monopond Incoming Fax API

### Overview
This API brings you the ability to receive faxes programmatically. Since a number is needed to be able to receive messages, this API also gives you the ability subscribe to fax numbers programatically.

Using this REST API, you are able to view a list of available fax numbers and select numbers for purchase. Once subscribed to a fax number, the service allows you to receive documents that have been faxed to your fax number directly to your email inbox. Whenever a fax is received, we send an email to each of the recipients with the fax document attached as TIFF, PDF or PNG. The service also allows our platform to post fax results to you via http.

### Incoming Fax API Usage
**For details on using the API, please see the [Incoming Fax API page](https://github.com/Monopond/fax-api/wiki/Incoming-Fax-REST-API).**
