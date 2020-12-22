# Definitions

## Application Server

This is the server that hosts the application.

## End User Application

This is the application used by the end user to connect to the application server and to interact with it.

## Push Message

This is the content that the application server is sending to the end user application.

## Push System

This is the whole system used to deliver the push message from the application server to the end user application.

## Application Push Protocol
Or Application Server Protocol

This is the protocol the application server use to send push message. This is usually either the Provider Receiving Protocol or the Gateway Receiving Protocol.

## Provider Receiving Protocol

This is the protocol the Push Provider use to receive push message.

## Provider Push Protocol

This is the protocol the Push Provider use to send push message to the distributor application.

## Distributor Receiving Protocol

This refers to the provider push protocol except if the distributor application act as a push provider, then this refers to the provider receiving protocol.

## Push Gateway
Or Gateway

This is the server or programm the application server sends push messages to with its application push protocol. 

It is used to convert the application push protocol to the provider receiving protocol. 

If the application push protocol and the provider receiving protocol are the same, and the application server can reach the push provider, then the gateway is not necessary, and the gateway will refer to the push provider.

If the application push protocol and the provider receiving protocol are the same, but the application server can not reach the push provider, then the gateway do not have to modify the request but have to forward it. In this case it can be called a proxy.

## Rewrite Proxy

If the Provider Receiving Protocol needs anything else than the URI and a GET parameter to identify the end user application (eg. header, POST parameter) or need a special structure for POST data, then a rewrite proxy is used to convert the identifier in a URI or in a GET parameter and to forge the POST parameter content structure.

The rewrite proxy is application independant and provider dependant.

## Push Provider
Or Provider
Or Push Notification Provider

This is the server that listen for incoming push messages using its provider receiving protocol and forward it to the connected phone using the provider push protocol.

## Distributor Application
Or Distributor

This is the application that forward push messages to the registered end user application. It is the application which is connected to the push provider.

## Connector Library
Or Connector
Or UnifiedPush Library

This is the library used by the end user application to register for push notifications to the distributor applicaton and to receive push messages forwarded from the ditributor application.

## Endpoint

This is the URL of the rewrite proxy (if there is one, of the push provider else) where push messages are sent for a specific end user application, from the gateway.

