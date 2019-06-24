# Introduction

## Prerequisites
To gain access to the system you must have a Monopond user account that is flagged with Fax API privileges. 
Please note some features described in this document will require access controls to be enacted. Examples of this include enabling specific destinations, filtering options and/or retry settings.

All of the above items can be organised through consulting with your account manager.

## Overview of the Monopond Outgoing Fax API
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

**For details on using the API, please see the [Outgoing Fax API page](https://github.com/Monopond/fax-api/wiki/Outgoing-Fax-SOAP-API).**

## Overview of the Monopond Incoming Fax API
This API brings you the ability to receive faxes programmatically. To be able to receive faxes, you must have a fax number. This API also gives you the ability subscribe to fax numbers programatically.

Using this REST API, you are able to view a list of available fax numbers and select numbers for purchase. Once subscribed to a fax number, you can receive requests to your application each time a fax message is received.

