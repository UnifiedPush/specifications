# Specifications

UnifiedPush Spec: DBUS_0.3.0

## Index

<!--toc:start-->

* [Resources](#resources)
* [Connector API](#connector-api)
    * [org.unifiedpush.Connector2.Message](#orgunifiedpushconnector2message)
    * [org.unifiedpush.Connector2.NewEndpoint](#orgunifiedpushconnector2newendpoint)
    * [org.unifiedpush.Connector2.Unregistered](#orgunifiedpushconnector2unregistered)
* [Distributor API](#distributor-api)
    * [org.unifiedpush.Distributor2.Register](#orgunifiedpushdistributor2register)
    * [org.unifiedpush.Distributor2.Unregister](#orgunifiedpushdistributor2unregister)
* [Appendix: Implementation practices](#appendix-implementation-practices)
    * [Registration](#registration)
    * [D-Bus Service](#d-bus-service)
    * [Choosing a distributor](#choosing-a-distributor)
    * [On Application Startup](#on-application-startup)
    * [D-Bus Interface Introspection XML files](#d-bus-interface-introspection-xml-files)
* [References](#references)
    * [Internal References](#internal-references)
    * [Normative References](#normative-references)
    * [Informative References](#informative-references)

<!--toc:end-->

## Resources

* All String argument MUST be UTF-8 encoded.
* Any call containing a parameter field that do not follow the specification MUST be ignored. For example, a call with "not_an_url" for the "endpoint" must be ignored.
* End user application: This is the application that will receive notifications with UnifiedPush; All the logic required for the end user application can be done with a library.
* Connection Token: This is a randomly generated token to identify the registration from the connector and the distributor. This token is generated by the connector, and send for the first time during the registration intent ([org.unifiedpush.Distributor2.Register]). This token must contain sufficient entropy so it cannot be guessed. To generate this token, UUIDv4 ([RFC9562]) is suggested. It is unique on distributor and connector side. Every requests to the connector contains this token. If the connector doesn't know this token, the request is ignored. This is a string of maximum 100 bytes.
* VAPID public key: This is a public key on the P-256 curve encoded in the uncompressed form [SEC 1] (section 2.3.3, replicated from X9.62), and base64url encoded [RFC7515]. This public key is generated by the end user application, and send during the registration intent ([org.unifiedpush.Distributor2.Register]). It is used by the application server to identify itself to the push server, following the [RFC8292]. Some distributors MAY require a valid VAPID key, and response with a registration fail if not present ([org.unifiedpush.android.connector.REGISTRATION_FAILED]). This is a 87 bytes long string.
* Push message: This is an array of bytes (Array\<Byte\>) sent by the application server to the push server. The distributor sends this message to the end user application. It MUST be the raw POST data received by the push server (or the rewrite proxy if present). The message MUST be an encrypted content that follows [RFC8291]. Its size is between 1 and 4096 bytes (inclusive).
* Endpoint: This is the URL of the push resource as defined by [RFC8030]. This url point to the push server and is distributed to the end user application by the distributor. This MUST be at most 1000 bytes. As defined by [RFC8030], authorization is managed using capability URLs ([CAP-URI]). Therefore, the endpoint MUST contain enough random entropy to ensure it is diffucult to successfully guess a valid URL. We recommend using a 160 bits (20 bytes) random value URL-safe base64 encoded string.
* Short description of the registration: This is a string send by the end user application during registration describing the registration purpose (eg. the account name of the application) the distributor may show on its user interface. It is at most 100 bytes long string.

## Connector API

The connector MUST provide a service with a name identifying the application (standard reverse DNS style application ID, which is also used in D-Bus APIs). This value is passed to the distributor during the registration when [org.unifiedpush.Distributor2.Register] is called.

The connector MUST implement the `org.unifiedpush.Connector2` interface at the object path `/org/unifiedpush/Connector`.

The caller of the methods in this interface MUST NOT wait for a response from them.

### org.unifiedpush.Connector2.Message

The distributor MUST call this method to send a new push message to the connector.

Arguments MUST be a variant dictionary (`a{sv}`) with the field below:

* key: "token", value: (String `s`) the connection token as defined in the [Resources]
* key: "message", value: (Byte array `ay`) the push message as defined in the [Resources], which is the raw POST data received by the push server.
* (Optional) key: "id", value: (String `s`) the message id as defined in the [Resources], or an empty string.

This method MUST return a variant dictionary (`a{sv}`) with the field below:

* (If key was send) key: "id", value: (String `s`) the message id received during the call.

The distributor SHOULD follow the push message urgency as defined in [RFC8030], section 5.3. If the push server does not send an urgency header, the urgency is considered as normal. Else, only messages more urgent than the minimum urgency SHOULD be send to the end user application. The minimum urgency depending on the device state is as follow, in order of increasing urgency:

| Urgency  | Device State                | Example Application                         |
|----------|-----------------------------|---------------------------------------------|
| very-low | On power and Wi-Fi          | Advertisements                              |
| low      | On either power or Wi-Fi    | Topic updates                               |
| normal   | On neither power nor Wi-Fi  | Chat or Calendar Message                    |
| high     | Low battery                 | Incoming phone call or time-sensitive alert |

### org.unifiedpush.Connector2.NewEndpoint

The distributor MUST call this method to inform the connector of the endpoint URL, both after registering for the first time, and if the endpoint changes afterwards.

The distributor MUST call this method in the following cases:

* the end user application requested a registration ([org.unifiedpush.Distributor2.Register]) with a token it uses for the first time, and the registration succeed
* the end user application requested a registration ([org.unifiedpush.Distributor2.Register]) with a token it is already using
* the endpoint for the registration changed

Arguments MUST be a variant dictionary (`a{sv}`) with the field below:

* key: "token", value: (String `s`) the connection token as defined in the  [Resources]
* key: "endpoint", value: (String `s`) the endpoint as defined in the [Resources]

This method MUST return an empty variant dictionary (`a{sv}`).

### org.unifiedpush.Connector2.Unregistered

The distributor MUST call this method to confirm unregistration or to inform the application that it has been unregistered.

If this action is send to inform the application, the intent MUST have the following argument:

Arguments MUST be a variant dictionary (`a{sv}`) with the field below:

* key: "token", value: (String `s`) the token of the connection

This method MUST return an empty variant dictionary (`a{sv}`).

If the app wants to remain registered, it MUST register again.

## Distributor API

Distributors MUST provide a service with a name beginning with `org.unifiedpush.Distributor.` followed by a string to identify the distributor (for example, `org.unifiedpush.Distributor.distributorname` where `distributorname` is the distributor's name)

The distributor MUST implement the `org.unifiedpush.Distributor2` interface at the object path `/org/unifiedpush/Distributor`.

### org.unifiedpush.Distributor2.Register

The connector MUST call this method to register for push messages or to retreive the push endpoint.

Arguments MUST be a variant dictionary (`a{sv}`) with the field below:

* key: "service", value: (String `s`) the connector service name for the application
* key: "token", value: (String `s`) a random token to identify the connection between the connector and the distributor. It MUST be unique on distributor side
* (Optional) key: "description", value: (String `s`) a description of the app and its reason for using push notifications. This MAY be shown to the user by the distributor.
* (Optional) key: "vapid", value: (String `s`) a vapid key as defined in the [Resources]

When it has not registered before, the connector MUST generate a random string to use as its token, and call this method with that token as its argument to register. At every subsequent startup of the app, it SHOULD call this method with the same token as before, to fetch the newest endpoint from the connector.

This method MUST return a variant dictionary (`a{sv}`) with the field below.

* key: "success", value: (String `s`) "REGISTRATION_FAILED" or "REGISTRATION_SUCCEEDED". "REGISTRATION_SUCCEEDED" if registration succeeded. It is "REGISTRATION_FAILED" in case the registration failed.
* (Required if REGISTRATION_FAILED) key: "reason", value: (String `s`) a reason string that MUST be:
    * "INTERNAL_ERROR": This is a generic error type, the connector can try again later
    * "NETWORK": The registration failed because of missing network connection, try again when network is back.
    * "ACTION_REQUIRED": The distributor requires a user action to work. For instance, the distributor may be log out of the push server and requires the user to log in. If the distributor has a limit of number of registrations and this limit has been reached, the distributor sends this reason.
    * "VAPID_REQUIRED": If the distributor requires a VAPID key and the end user application doesn't send one, the distributor respond with this reason.
    * "UNAUTHORIZED": If the distributor allows registrations only from sandboxes applications and the calling process doesn't follow the requirements.

If registration succeeded, the distributor MUST call the connector's [org.unifiedpush.Connector2.NewEndpoint] method to deliver the new push endpoint to it.

The distributor MAY restrict registrations to sandboxed applications only. The distributor SHOULD then control the service is correctly exposed by the application if the supported sandboxing technology allows it.

For example, a distributor may be used by flatpak applications only. It can then get the caller PID of the registration request using the SO_PEERCRED socket option then get flatpak information on `/proc/<pid>/root/.flatpak-info`. The distributor will allow registrations only if the service is explicitely allowed.

### org.unifiedpush.Distributor2.Unregister

The connector MUST call this method to unregister for push messages.

Arguments MUST be a variant dictionary (`a{sv}`) with the field below:

* key: "token", value: (String `s`) the token of the connection

This method MUST return an empty variant dictionary (`a{sv}`).

## Appendix: Implementation practices

These are some practices and hints that may be useful to you if you want to implement a part of this API.

### Registration

When a connector first wants to register, it should fetch all service names currently registered on the session bus, and filter it to only those which begin with `org.unifiedpush.Distributor.`. It should then choose one of these (see below) to use as its distributor. It must then generate a random string to use as a token (UUIDv4 is recommended), and call the [org.unifiedpush.Distributor2.Register] method with this token as the argument. If the return value indicates "NEW_ENDPOINT", the distributor will then call the connector's [org.unifiedpush.Connector2.NewEndpoint] method to deliver the endpoint to it. When this has happened, the connector saves the token and the  endpoint received in its persistent storage. Registration is now complete.

On subsequent startups, the connector should call [org.unifiedpush.Distributor2.Register] with the *same* token, and store the new endpoint returned, if it has changed. The connector should also implement the [org.unifiedpush.Connector2.NewEndpoint] method to receive notification about a changed endpoint from the distributor at any time.

### D-Bus Service

The application must provide a DBus service file so the DBus daemon can activate it as a service on the session bus when the distributor invokes any of the above methods. For the location of the service file see the [DBus daemon manpage], typical locations are `/usr/share/dbus-1/services/` and for Flatpaks `/app/share/dbus-1/services/`. The name of the service file needs to match the well known bus name of your application (usually the app ID)  with `.service` appended. If your app has the app ID `tld.yourdomain.YourApp` the service file `tld.yourdomain.YourApp.service` would look like:

```ini
[D-BUS Service]
Name=tld.domain.YourApp
Exec=/usr/bin/yourapp <command line arguments>
```

After processing the push notification (and e.g. sending out a desktop notification to notify the user) the application should terminate again.

Note that for GUI apps the activation by the DBus daemon should not open any user visible windows, it should be in "daemon" mode waiting for any DBus calls. If your application already supports DBus activation as described in the
[XDG Desktop entry specification] you likely don't need to do anything.

For example, for GTK/GLib based applications the `<command line arguments>` is `--gapplication-service` (see [DBus launching in GNOME's wiki]). The application invokes `g_appliction_hold()` after the first request from the distributor to ensure the application keeps running long enough to process all the requests. When done processing the push notifications it invokes `g_application_release()`. This allows the application to shut down again until the next push notification arrives or the user clicks on a desktop notification.

### Choosing a distributor

When you list all services that begin with `org.unifiedpush.Distributor.` on the session bus, you may get zero, one, or multiple results. If there are zero, you can't register to get push notifications. If there is one, you can simply choose that one. If there are multiple, then you can choose one of them by presenting a UI to select one distributor (since the user likely knows why they have multiple different distributors running). You can also get the "preferred" distributor from an environment variable, although the name and behavior of this variable have not been specced (TODO).

### On Application Startup

The application should call [org.unifiedpush.Distributor2.Register] on every startup to request the newest endpoint URL, in case it missed this call otherwise.

### D-Bus Interface Introspection XML files

* [org.unifiedpush.Distributor2.xml](https://codeberg.org/UnifiedPush/specifications/src/branch/main/specifications/dbus/org.unifiedpush.Distributor2.xml)
* [org.unifiedpush.Connector2.xml](https://codeberg.org/UnifiedPush/specifications/src/branch/main/specifications/dbus/org.unifiedpush.Connector2.xml)

## References

### Internal References

[Resources]

[org.unifiedpush.Distributor2.Register]

[org.unifiedpush.Connector2.NewEndpoint]

[Resources]: #resources
[org.unifiedpush.Distributor2.Register]: #orgunifiedpushdistributor2register
[org.unifiedpush.Connector2.NewEndpoint]: #orgunifiedpushconnector2newendpoint

### Normative References

[CAP-URI] Capability URLs

[SEC 1] SEC 1: Elliptic Curve Cryptography

[RFC7515] JSON Web Signature (JWS)

[RFC9562] Universally Unique IDentifiers (UUIDs)

[RFC8030] Generic Event Delivery Using HTTP Push

[RFC8292] Voluntary Application Server Identification (VAPID) for Web Push

[RFC8291] Message Encryption for Web Push

[XDG Desktop entry specification] XDG Desktop entry specification

[CAP-URI]: https://www.w3.org/TR/capability-urls/ "Capability URLs"
[SEC 1]: https://www.secg.org/sec1-v2.pdf "SEC 1: Elliptic Curve Cryptography"
[RFC7515]: https://www.rfc-editor.org/rfc/rfc7515 "JSON Web Signature (JWS)"
[RFC9562]: https://www.rfc-editor.org/rfc/rfc9562 "Universally Unique IDentifiers (UUIDs)"
[RFC8030]: https://www.rfc-editor.org/rfc/rfc8030 "Generic Event Delivery Using HTTP Push"
[RFC8292]: https://www.rfc-editor.org/rfc/rfc8292 "Voluntary Application Server Identification (VAPID) for Web Push"
[RFC8291]: https://www.rfc-editor.org/rfc/rfc8291 "Message Encryption for Web Push"

[XDG Desktop entry specification]: https://specifications.freedesktop.org/desktop-entry-spec/latest/dbus.html

### Informative References

[DBus daemon manpage] The DBus daemon manpage

[DBus launching in GNOME's wiki] Setting up an application for D-Bus Launching in GNOME's wiki

[DBus daemon manpage]: https://dbus.freedesktop.org/doc/dbus-daemon.1.html
[DBus launching in GNOME's wiki]: https://wiki.gnome.org/HowDoI/DBusApplicationLaunching
