# Definitions

## Application Server

(e.g. Synapse, Fediverse Server, ...)

This is the server that hosts the application.

## End User Application

or User Agent

(e.g. [FluffyChat](https://fluffychat.im/), [Fedilab](https://fedilab.app/), [Tox Push Message App](https://github.com/zoff99/tox_push_msg_app))

This is the application used by the end user to connect to the [Application Server](#application-server) and to interact with it.

## Push Message

This is the content that the [Application Server](#application-server)
wants to send to the [end user Application](#end-user-application).

## Push System

This is the whole system used to deliver the [push messages](#push-message)
from the [Application Server](#application-server)
to the [end user Application](#end-user-application).

## Application Push Protocol

Or Application Server Protocol
(e.g. [Matrix push gateway api](https://spec.matrix.org/unstable/push-gateway-api/))

This is the protocol the [Application Server](#application-server) uses to send [push messages](#push-message).

## Push Gateway

Or Gateway

(e.g. [UnifiedPush-common-proxies](https://codeberg.org/UnifiedPush/common-proxies), [Nginx](https://codeberg.org/UnifiedPush/contrib/blob/main/gateways/matrix.md#nginx))

The Push Gateway is used for conversion and/or proxying of messages from [Application Server](#application-server) to the [Push Server](#push-server).
If the [Application Push Protocol](#application-push-protocol) and the [Push Server Receiving Protocol](#push-server-receiving-protocol) are the same, and the [Application Server](#application-server) can reach the [Push Server](#push-server), then the gateway is not necessary.

If the application server can not reach the push server the Push Gateway can also act as a normal Proxy, even if the [application push protocol](#application-push-protocol) and the [Push Server Receiving Protocol](#push-server-receiving-protocol) are the same.

## Push Server Receiving Protocol

This is the protocol the [Push Server](#push-server) uses to receive [push messages](#push-message).

## Distributor Receiving Protocol

This is the protocol the [Push Distributor](#push-distributor)
uses to receive [push messages](#push-message) from the [Push Server](#push-server).
If the [Push Distributor](#push-distributor)
acts as the [Push Server](#push-server),
then this is just the [Push Server Receiving Protocol](#push-server-receiving-protocol).

## Rewrite Proxy

If the Push Server Receiving Protocol needs anything else than the URI and a GET parameter to identify the end user application (eg. header, POST parameter) or need a special structure for POST data, then a rewrite proxy is used to convert the identifier in a URI or in a GET parameter and to forge the POST parameter content structure.

The rewrite proxy is application independent and push server dependant.

## Push Server

Or Push Provider
Or Provider

(e.g. [ntfy server](https://ntfy.sh/), [NextPush server](https://codeberg.org/NextPush/uppush), [Google FirebaseCloudMessaging](https://firebase.google.com/docs/cloud-messaging/))

This is the server that listens for incoming [push messages](#push-message) using its [Push Server Receiving Protocol](#push-server-receiving-protocol) and forwards it to the connected [Push Distributor](#push-distributor) using the [Distributor Receiving Protocol](#distributor-receiving-protocol) .

## Push Distributor

Or Distributor
Or Push Service

(e.g. [ntfy app](https://ntfy.sh), [NextPush Android](https://codeberg.org/NextPush/nextpush-android), [UP-FCM Distributor](https://codeberg.org/UnifiedPush/fcm-distributor))

This is the application that forwards push messages to the registered [end user Application](#end-user-application). It is the application which is connected to the [Push Server](#push-server).

## Connector Library

Or Connector

(e.g. the UnifiedPush Libraries ([Android](https://codeberg.org/UnifiedPush/android-connector), [Flutter](https://codeberg.org/UnifiedPush/flutter-connector)...))

This is the library used by the [end user Application](#end-user-application) to register for and receive forwarded push notifications at the [Push Distributor](#push-distributor).

## Endpoint

This is the URL of the Push Resource as defined for Web Push ([RFC8030](https://www.rfc-editor.org/rfc/rfc8030#section-2)) which is the URL of the [Rewrite Proxy](#rewrite-proxy) (if there is one, otherwise it is from the [Push Server](#push-server)) where push messages are sent to for a specific [end user Application](#end-user-application), from the [Push Gateway](#push-gateway).
