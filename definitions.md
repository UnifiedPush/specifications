# Definitions

## Application Server
(e.g. Synapse, Fediverse Server, ...)

This is the server that hosts the application.

## End User Application
(e.g. [FluffyChat](https://fluffychat.im/), [Fedilab](https://fedilab.app/), [Tox Push Message App](https://github.com/zoff99/tox_push_msg_app))

This is the application used by the end user to connect to the [Application Server](#application-server) and to interact with it.

## Push Message

This is the content that the [Application Server](#application-server)
wants to send to the [end user Application](#end-user-application).

## Push System

This is the whole system used to deliver the [push messages](#Push-Message) 
from the [Application Server](#application-server) 
to the [end user Application](#end-user-application).

## Application Push Protocol
Or Application Server Protocol
(e.g. [Matrix push gateway api](https://spec.matrix.org/unstable/push-gateway-api/))

This is the protocol the [Application Server](#application-server) uses to send [push messages](#push-message). 

## Push Gateway
Or Gateway

(e.g. [UnifiedPush-common-proxies](https://github.com/UnifiedPush/common-proxies), [Nginx](https://github.com/UnifiedPush/contrib/blob/main/gateways/matrix.md#nginx))

The Push Gateway is used for conversion and/or proxieing of messages from [Application Server](#application-server) to the [Push Provider](#push-provider).
If the [Application Push Protocol](#application-push-protocol) and the [Provider Receiving Protocol](#provider-receiving-protocol) are the same, and the [Application Server](#application-server) can reach the [Push Provider](#push-provider), then the gateway is not necessary.

If the application server can not reach the push provider the Push Gateway can also act as a normal Proxy, even if the [application push protocol](#application-push-protocol) and the [Provider Receiving Protocol](#provider-receiving-protocol) are the same.

## Provider Receiving Protocol

This is the protocol the [Push Provider](#push-provider) uses to receive [push messages](#push-message).

## Provider Push Protocol

This is the protocol the [Push Provider](#push-provider) uses to send [push messages](#push-message) to the [Distributor Application](#distributor-application).

## Distributor Receiving Protocol

This is the protocol the [Distributor Application](#distributor-application) 
uses to recive [push messages](#push-message) from the [Push Provider](#push-provider). 
So it is the same as the [Provider Push Protocol](#provider-push-protocol), 
except if the [Distributor Application](#distributor-application) 
acts as the [Push Provider](#push-provider), 
then there is no [Provider Push Protocol](#provider-push-protocol) 
and this is just the [Provider Receiving Protocol](#provider-receiving-protocol).

## Rewrite Proxy

If the Provider Receiving Protocol needs anything else than the URI and a GET parameter to identify the end user application (eg. header, POST parameter) or need a special structure for POST data, then a rewrite proxy is used to convert the identifier in a URI or in a GET parameter and to forge the POST parameter content structure.

The rewrite proxy is application independant and provider dependant.

## Push Provider
Or Provider

Or Push Notification Provider

(e.g. [Gotify](https://gotify.net/), [Google FirebaseCloudMessaging](https://firebase.google.com/docs/cloud-messaging/))

This is the server that listen for incoming [push messages](#Push-Message) using its [Provider Receiving Protocol](#provider-receiving-protocol) and forwards it to the connected [Distributor Application](#distributor-application) using the [Provider Push Protocol](#provider-push-protocol) .

## Distributor Application
Or Distributor

(e.g. [Gotify-UnifiedPush-android](https://github.com/UnifiedPush/gotify-android), [UP-FCM Distributor](https://github.com/UnifiedPush/fcm-distributor))

This is the application that forwards push messages to the registered [end user Application](#end-user-application). It is the application which is connected to the [Push Provider](#push-provider).

## Connector Library
Or Connector

(e.g. the UnifiedPush Libraries ([Android](https://github.com/UnifiedPush/android-connector), [Flutter](https://github.com/UnifiedPush/flutter-connector)...))

This is the library used by the [end user Application](#end-user-application) to register for and recive forwarded push notifications at the [Distributor Application](#distributor-application).

## Endpoint

This is the URL of the [Rewrite Proxy](#rewrite-proxy) (if there is one, otherwise it is from the [Push Provider](#push-provider)) where push messages are sent to for a specific [end user Application](#end-user-application), from the [Push Gateway](#push-gateway).

