# Specifications

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
* org.unifiedpush.android.connector.MESSAGE

This broadcast receiver is the Messaging Broadcast Receiver.


## Registration Broadcast Receiver

The exposed broadcast receiver of the distributor application MUST handle 2 differents actions:
* org.unifiedpush.android.distributor.REGISTER
* org.unifiedpush.android.distributor.UNREGISTER

### org.unifiedpush.android.distributor.REGISTER

The connector send this action to register to push messages. The intent MUST contain 2 String extras:
* application: with the end user application package name
* token: with a random token to identify the connection between the connector and the distributor

When the distributor handles this action :
* MUST send a broadcast intent to the action org.unifiedpush.android.connector.NEW_ENDPOINT of the registered app, with 2 String extras :
	* token: the token supplied by the end user application during registration
	* endpoint: the endpoint

### org.unifiedpush.android.distributor.UNREGISTER

When the distributor handles this action :
* MUST send a broadcast intent to the action org.unifiedpush.android.connector.UNREGISTERED of the registered app.


## Messaging Broadcast Receiver

The exposed broadcast receiver of the distributor application MUST handle 3 differents actions:
* org.unifiedpush.android.distributor.NEW_ENDPOINT
* org.unifiedpush.android.distributor.MESSAGE

There is a third action the connector SHOULD handle:
* org.unifiedpush.android.distributor.UNREGISTERED

### org.unifiedpush.android.distributor.NEW_ENDPOINT

The distributor MUST send this action to the registered application to confirm the registration of an end user application or when the endpoint change with the 2 following extras:
* token: the token supplied by the end user application during registration
* endpoint: the endpoint

### org.unifiedpush.android.distributor.MESSAGE

The distributor MUST send this action to the registered application to forward a push message received from the provider to the end user application with the 2 following extras:
* token: the token supplied by the end user application during registration
* message: the push message sent by the application server

### org.unifiedpush.android.distributor.UNREGISTERED

The distributor MUST send this action to the registered application to confirm unregistration or to inform the application about unregistration.

If this action is send to inform the application, the intent MUST have the following extra:
* token: the token supplied by the end user application during registration

