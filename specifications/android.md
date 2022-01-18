# Specifications

UnifiedPush Spec: AND_2.0.0-beta1

## Index

* [Distributor Application](#distributor-application)
* [Connector Library](#connector-library)
* [Registration Broadcast Receiver](#registration-broadcast-receiver-1)
* [Messaging Broadcast Receiver](#messaging-broadcast-receiver-1)

## Distributor Application

### Registration Broadcast Receiver

The distributor application MUST expose the registration broadcast receiver, allowing end user applications to register for push related messages.

### Manifest

The distributor MUST expose a broadcast receiver with the following actions:
* org.unifiedpush.android.distributor.REGISTER
* org.unifiedpush.android.distributor.UNREGISTER

This broadcast receiver is the Registration Broadcast Receiver.


## Connector Library

### Messaging Broadcast Receiver

The connector library MUST expose the messaging broadcast receiver, allowing the distributor to send push related messages.

### Manifest

The library itself does not declare any activity, service or receiver in the manifest. The end user application integrating the library MUST expose a broadcast receiver with the following actions:
* org.unifiedpush.android.connector.NEW_ENDPOINT
* org.unifiedpush.android.connector.UNREGISTERED
* org.unifiedpush.android.connector.MESSAGEv2
* org.unifiedpush.android.connector.REGISTRATION_FAILED

This broadcast receiver is the Messaging Broadcast Receiver.


## Registration Broadcast Receiver

The exposed broadcast receiver of the distributor application MUST handle 2 differents actions:
* org.unifiedpush.android.distributor.REGISTER
* org.unifiedpush.android.distributor.UNREGISTER

### org.unifiedpush.android.distributor.REGISTER

The connector send this action to register to push messages. The intent MUST contain 3 extras:
* application (String): with the end user application package name. The distributor MUST be able to handle many connections with a single application.
* token (String): with a random token to identify the connection between the connector and the distributor. It MUST be unique on distributor side.
* versions (Array\<Int\>): an array of the major versions number supported by the connector.

The distributor MUST send a broadcast intent to one of the following action when it handles this action:
* org.unifiedpush.android.connector.NEW_ENDPOINT
* org.unifiedpush.android.connector.REGISTRATION_FAILED

### org.unifiedpush.android.distributor.UNREGISTER

The connector send this action to unregister to push messages. The intent MUST contain 1 extra:
* token (String): the token supplied by the end user application during registration

The distributor MUST send a broadcast intent to the following action when it handles this action:
* org.unifiedpush.android.connector.UNREGISTERED.

### org.unifiedpush.android.distributor.MESSAGE_ACK

The connector MUST reply with this action to the distributor with the String extra id received to acknowledge the message reception.


## Messaging Broadcast Receiver

The exposed broadcast receiver of the distributor application MUST handle 3 differents actions:
* org.unifiedpush.android.connector.NEW_ENDPOINT
* org.unifiedpush.android.connector.MESSAGEv2

There is a third action the connector SHOULD handle:
* org.unifiedpush.android.connector.UNREGISTERED

### org.unifiedpush.android.connector.NEW_ENDPOINT

The distributor MUST send this action to the registered application to confirm the registration of an end user application, when a registered application send again an action with the intent org.unifiedpush.android.distributor.REGISTER and a valid token, or when the endpoint change with the 2 following extras:
* token (String): the token supplied by the end user application during registration
* endpoint (String): the endpoint

### org.unifiedpush.android.connector.REGISTRATION_FAILED

The distributor MUST send this action to the registered application if the token is already registered for another application or if the registration can not be processed (for instance when the distributor is not connected to its provider server) with the 2 following extras:
* token (String): the token supplied by the end user application during registration
* message (String): this extra MAY be sent to gives an error message

The connector MUST change the connection token received with this action for the next registration.

### org.unifiedpush.android.connector.MESSAGEv2

The distributor MUST send this action to the registered application to forward a push message received from the provider to the end user application.

It MUST be send with the 2 following extras:
* token (String): the token supplied by the end user application during registration
* message (ByteArray): the push message sent by the application server. It MUST be the raw POST data received by the rewrite proxy.

It MAY be send with the following extra:
* id (String): to identify the message

### org.unifiedpush.android.connector.UNREGISTERED

The distributor MUST send this action to the registered application to confirm unregistration or to inform the application about unregistration.

If this action is send to inform the application, the intent MUST have the following extra:
* token (String): the token supplied by the end user application during registration

