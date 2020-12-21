# Connector  Library Sepcification

## Unified Messaging Service

The connector library exposes a service to bind to, allowing the distributor to send push related messages.

### Manifest

The library itself does not declare any activity, service or receiver in the manifest. The developer integrating the library MUST add the following lines to the manifest where {PushReceiverService} is the service implementing the below specification.

```
<service android:name="{PushReceiverService}" android:exported="true">
  <intent-filter>
    <action android:name="org.unifiedpush.android.action.MESSAGING"/>
  </intent-filter>
</service>
```

### Incoming Handler

The exposed service MUST handle 3 different types of messages:
* TYPE_DISTRIBUTOR_MESSAGE
* TYPE_DISTRIBUTOR_ENDPOINT_CHANGED
* TYPE_DITRIBUTOR_UNREGISTERED

#### TYPE_DISTRIBUTOR_MESSAGE

The distributor sends this type of message to forward a push message received from the provider to the end user application.
The message data bundle MUST contain the key "push_message" with a String value representing the push message to be delivered.
The message data bundle MUST contain the key "push_message_id" with a String value representing a unique identifier for the message that can be used to acknowledge the message

#### TYPE_DISTRIBUTOR_MESSAGE_ACKNOWLEDGE

The library sends this message once processing of a push message is completed. The data bundle of the message MUST contain the field "push_message_id" with the id of the push message to acknowledge.

#### TYPE_DISTRIBUTOR_ENDPOINT_CHANGED

The distributor sends this type of message to deliver a new endpoint that MUST be used instead of any previous ones to deliver push messages.
The message data bundle MUST contain the key "endpoint" with a String value representing the new endpoint.

#### TYPE_DISTRIBUTOR_UNREGISTERED

The distributor sends this type of message to inform the client that the push message registration was terminated and that it can no longer be used.


