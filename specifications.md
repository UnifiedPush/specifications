# Specifications


## Distributor Application

### Registration Service

The distributor application MUST expose the registration service to bind to, allowing end user applications to register for push related messages.

### Manifest

The distributor MUST expose an exported service with an intent filter for the `org.unifiedpush.android.action.REGISTRATION` action. This service is the Registration Service.


## Connector Library

### Messaging Service

The connector library MUST expose the messaging service to bind to, allowing the distributor to send push related messages.

### Manifest

The library itself does not declare any activity, service or receiver in the manifest. The end user application integrating the library MUST expose an exported service with an intent filter for the `org.unifiedpush.android.action.MESSAGING` action. This service is the Messaging Service.


## Registration Service

The exposed service of the distributor application MUST handle 2 differents type of messages:
* TYPE_CONNECTOR_REGISTER
* TYPE_CONNECTOR_UNREGISTER

### TYPE_CONNECTOR_REGISTER message

The connector send this type of message to register to push messages. The message data bundle SHOULD NOT contain anything.

When the distributor handles this type of message it :
* MUST return a message with type TYPE_CONNECTOR_REGISTER_SUCCESS if the registration was successful.
* MUST return a message with type TYPE_CONNECTOR_REGISTER_FAILED if the registration was not successful.

### TYPE_CONNECTOR_REGISTER_SUCCESS message

The distributor MUST reply this message to the TYPE_CONNECTOR_REGISTER message during registration if the registration was successful.
The message data bundle MUST contain "endpoint" containing the endpoint and MAY contain anything else. 

### TYPE_CONNECTOR_REGISTER_FAILED message

The distributor MUST reply this message to the TYPE_CONNECTOR_REGISTER message during registration if the registration was not successful.

The message data bundle MUST contain "reason" containing one of the following strings :
* "DISABLED" if the registrations are disabled on the distributor.
* "BLOCKED" if the request was not accepted by the distributor.
* "DEFERRED" if the distributor could not achieve the registration but is committing to do so once it is able. (due to a miss connection with one of its remote part.) The endpoint MUST be sent to the end user application with the endpoint update function (cf. coming).
The message data bundle MAY contain any other value.

### TYPE_CONNECTOR_UNREGISTER message

The connector send this type of message to unregister to push messages. The message data bundle SHOULD NOT contain anything.

When the distributor handles this type of message it :
* MUST reply a message with type TYPE_CONNECTOR_UNREGISTER_ACKNOWLEDGE.

### TYPE_CONNECTOR_UNREGISTER_ACKNOWLEDGE message

The distributor MUST reply this message to the TYPE_CONNECTOR_UNREGISTER message when the unregistration is acknowledged.


## Messaging Service

The connector MUST handle 2 different types of messages:
* TYPE_DISTRIBUTOR_MESSAGE
* TYPE_DISTRIBUTOR_ENDPOINT_CHANGED

There is a third type of message the connector SHOULD handle:
* TYPE_DITRIBUTOR_UNREGISTERED

### TYPE_DISTRIBUTOR_MESSAGE

The distributor MUST send this type of message to forward a push message received from the provider to the end user application.

The message data bundle MUST contain the key "push_message" with a String value representing the push message sent by the application server to be delivered to the end user application.

The message data bundle MUST contain the key "push_message_id" with a String value representing a unique identifier for the message that can be used to acknowledge the message.

### TYPE_DISTRIBUTOR_MESSAGE_ACKNOWLEDGE

The library MUST reply with this message once processing of a push message is completed. The data bundle of the message MUST contain the field "push_message_id" with the id of the push message to acknowledge.

### TYPE_DISTRIBUTOR_ENDPOINT_CHANGED

The distributor MUST send this type of message to deliver a new endpoint that MUST be used instead of any previous one to deliver push messages.

The message data bundle MUST contain the key "endpoint" with a String value representing the new endpoint.

### TYPE_DISTRIBUTOR_UNREGISTERED

The distributor MUST send this type of message to inform the client that the push message registration was terminated and that it can no longer be used.

