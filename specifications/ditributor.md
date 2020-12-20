# Distributor Application Specifications

## Registration Service

The distributor application should expose a service to bind to, allowing end user applications to register for push notifications.

### Manifest

The distributor MUST expose an exported service with an intent filter for the `org.unifiedpush.android.action.REGISTRATION` action.

### Incoming Handler

The exposed service of the distributor application MUST handle 2 differents type of messages:
* TYPE_CLIENT_REGISTER
* TYPE_CLIENT_UNREGISTER

#### TYPE_CLIENT_REGISTER message

When the distributor handles this type of message it :
* MAY ask for user confirmation.
* SHOULD store information about the end user application if the registrations are open.
* MUST return a message with type TYPE_CLIENT_REGISTERED if the registration was successful.
* MUST return a message with type TYPE_CLIENT_REGISTER_FAILED if the registration was not successful.

#### TYPE_CLIENT_REGISTERED message

The distributor sends this message during registration if the registration was successful.
The message data bundle MUST contain "endpoint" containing the endpoint and CAN contain anything else. 

#### TYPE_CLIENT_REGISTER_FAILED message

The distributor sends this message during registration if the registration was not successful.
The message data bundle MUST contain "reason" containing one of the following strings :
* "DISABLED" if the registrations are disabled on the distributor.
* "BLOCKED" if the request was not accepted by the distributor.
* "DEFERRED" if the distributor could not achieve the registration but is committing to do so once it is able. (due to a miss connection with one of its remote part.) The endpoint MUST be sent to the end user application with the endpoint update function (cf. coming).
The message data bundle CAN contain any other value.

#### TYPE_CLIENT_UNREGISTER message

When the distributor handles this type of message it :
* MUST remove information about the end user application.
* MUST return a message with type TYPE_CLIENT_UNREGISTERED

#### TYPE_CLIENT_UNREGISTERED message

The distributor sends this message when the unregistration is acknowledged.


