# Specifications

UnifiedPush Spec: AND_2.0.0-beta3

## Index

* [General](#general)
* [Distributor Application](#distributor-application)
* [Connector Library](#connector-library)
* [Registration Broadcast Receiver](#registration-broadcast-receiver-1)
* [Messaging Broadcast Receiver](#messaging-broadcast-receiver-1)

## General

All extras typed String MUST be UTF-8 encoded.


## Distributor Application

### Registration Broadcast Receiver

The distributor application MUST expose the registration broadcast receiver, allowing end user applications to register for push related messages.

### Common manifest

The distributor MUST expose a broadcast receiver with the following actions:
* org.unifiedpush.android.distributor.REGISTER
* org.unifiedpush.android.distributor.UNREGISTER

This broadcast receiver is the Registration Broadcast Receiver.

### Optional features

#### Sending bytes messages

The distributor MUST expose the following action if it supports sending messages as ByteArray instead of String:
* org.unifiedpush.android.distributor.feature.BYTES_MESSAGE


## Connector Library

### Messaging Broadcast Receiver

The connector library MUST expose the messaging broadcast receiver, allowing the distributor to send push related messages.

### Manifest

The library itself does not declare any activity, service or receiver in the manifest. The end user application integrating the library MUST expose a broadcast receiver with the following actions:
* org.unifiedpush.android.connector.NEW_ENDPOINT
* org.unifiedpush.android.connector.UNREGISTERED
* org.unifiedpush.android.connector.MESSAGE
* org.unifiedpush.android.connector.REGISTRATION_FAILED

This broadcast receiver is the Messaging Broadcast Receiver.

## Registration Broadcast Receiver

The exposed broadcast receiver of the distributor application MUST handle 2 differents actions:
* org.unifiedpush.android.distributor.REGISTER
* org.unifiedpush.android.distributor.UNREGISTER

### org.unifiedpush.android.distributor.REGISTER

The connector sends this action to register to push messages. The intent MUST contain 2 extras:
* application (String): the end user application package name. The distributor MUST be able to handle many registrations with a single application.
* token (String): a randomly generated token to identify the registration from the connector and the distributor. It MUST be unique on distributor side.

It MAY be sent with the following 2 extras:
* features (ArrayList\<String\>): indicate the connector is requesting a set of optional features to be enabled. It MUST be the qualified name of the action declared to advertise this feature. The connector MUST check that the action is declared before requesting an optional feature.
* message (String): a short description of the purpose of the registration that the distributor MAY show to the user.

The distributor MUST send a broadcast intent to one of the following actions when it handles this action:
* org.unifiedpush.android.connector.NEW_ENDPOINT
* org.unifiedpush.android.connector.REGISTRATION_FAILED

### org.unifiedpush.android.distributor.UNREGISTER

The connector sends this action to unregister from push messages. The intent MUST contain 2 extras:
* token (String): the token supplied by the end user application during registration
* application (String): the end user application package name.

The distributor MUST send a broadcast intent to the following action when it handles this action:
* org.unifiedpush.android.connector.UNREGISTERED.

### org.unifiedpush.android.distributor.MESSAGE_ACK

Whenever the connector receives a message with the extra id it MUST reply with this action to the distributor with the String extra id received to acknowledge the message reception.

## Messaging Broadcast Receiver

The exposed broadcast receiver of the connector library MUST handle 2 differents actions:
* org.unifiedpush.android.connector.NEW_ENDPOINT
* org.unifiedpush.android.connector.MESSAGE

There is a third action the connector SHOULD handle:
* org.unifiedpush.android.connector.UNREGISTERED

### org.unifiedpush.android.connector.NEW_ENDPOINT

The distributor MUST send this action to the registered application in the following cases:
* confirm the registration of an end user application
* a registered application sends an action with the intent org.unifiedpush.android.distributor.REGISTER and a token for an existing registration
*  the endpoint changes

The intent MUST contain the following 2 extras:
* token (String): the token supplied by the end user application during registration
* endpoint (String): the endpoint URL

### org.unifiedpush.android.connector.REGISTRATION_FAILED

The distributor MUST send this action to the registered application if:
* the token is already registered for another application
* the registration can not be processed (for instance when the distributor is not connected to its provider server)
* a requested feature is not supported by the distributor

The action MUST contain the following 2 extras:
* token (String): the token supplied by the end user application during registration
* message (String): this extra MAY be sent to give an error message

The connector MUST change the registration token received with this action for the next registration.

### org.unifiedpush.android.connector.MESSAGE

The distributor MUST send this action to the registered application to forward a push message received from the provider to the end user application.

If the BYTES_MESSAGE feature was requested, it MUST send the following extras:
* token (String): the token supplied by the end user application during registration
* bytesMessage (ByteArray): the push message sent by the application server, as an array of bytes. It MUST be the raw POST data received by the rewrite proxy.

If the BYTES_MESSAGE feature was requested, it MAY additionally send the message as a string:
* message (String): the push message sent by the application server, as a string.

If the BYTES_MESSAGE feature was not requested, it MUST send the following extras:
* token (String): the token supplied by the end user application during registration
* message (String): the push message sent by the application server, as a string.

If the BYTES_MESSAGE feature was not requested, it MAY additionally send the message as a byte array:
* bytesMessage (ByteArray): the push message sent by the application server, as a string.
  
It MAY be sent with the following extra:
* id (String): to identify the message


### org.unifiedpush.android.connector.UNREGISTERED

The distributor MUST send this action to the registered application to confirm unregistration or to inform the application about unregistration.

The intent MUST have the following extra:
* token (String): the token supplied by the end user application during registration

