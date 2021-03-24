# Specifications

## Index

* [Connector API](#connector-API)
* [Distributor API](#distributor-api)
* [Appendix: Implementation practices](#appendix-implementation-practices)

## Connector API

Connectors MUST provide a service with their application ID, that is, the same name as the name that they sent their `org.unifiedpush.Distributor1.Register` call from.

Connectors MUST implement the `org.unifiedpush.Connector1` interface at the object path `/org/unifiedpush/Connector`. The caller of the methods in this interface MUST NOT wait for a response from them!

### org.unifiedpush.Connector1.Message (String, String, String) → nothing

The distributor calls this method to send a new push message to the connector.

The first argument is the token of the connection, the second argument is the raw POST data as received by the distributor, and the third argument is optionally an ID identifying the message (otherwise it is an empty string). This method does not return anything.

### org.unifiedpush.Connector1.NewEndpoint (String, String) → nothing

The distributor MUST call this method to inform the connector of the endpoint URL, both after registering for the first time, and if the endpoint changes afterwards. In addition to implementing this method, the connector SHOULD also call [org.unifiedpush.Distributor1.Register](#orgunifiedpushdistributor1register-string--string-string) on every startup to request the newest endpoint URL, in case it missed this call otherwise.

The first argument is the token of the connection, and the second argument is the new endpoint. This method does not return anything.

### org.unifiedpush.Connector1.Unregister (String) → nothing

The distributor calls this method to inform the connector that it has been unregistered for some server-side or distributor-side reason. If the app wants to remain registered, it SHOULD simply call [org.unifiedpush.Distributor1.Register](#orgunifiedpushdistributor1register-string--string-string) again after getting unregistered. A response indicating "REGISTRATION_REFUSED" would mean that the app is no longer allowed to register.

The argument of the method is the token of the connection. This method does not return anything.

## Distributor API

Distributors MUST provide a service with a name beginning with `org.unifiedpush.Distributor.` followed by the name of the distributor (for example, `org.unifiedpush.Distributor.distributorname` where `distributorname` is the distributor's name)

The distributor MUST implement the `org.unifiedpush.Distributor1` interface at the object path `/org/unifiedpush/Distributor`.

### org.unifiedpush.Distributor1.Register (String, String) → (String, String)

The connector calls this method to register for push messages, or to retreive the newest push endpoint (as it may change at any time). See also the `org.unifiedpush.Connector1.NewEndpoint` method.

The method takes two string arguments. The first one is the connector's application ID (standard reverse DNS style application ID, which is also used elsewhere in D-Bus APIs), while the second one is a token to identify the connection between the connector and the distributor.

When it has not registered before, the connector MUST generate a random string to use as its token, and call this method with that token as its argument to register. At every subsequent startup of the app, it SHOULD call this method with the same token as before, to fetch the newest endpoint from the connector.

The method returns two strings. The first string MUST be either "NEW_ENDPOINT", "REGISTRATION_REFUSED", or "REGISTRATION_FAILED", and the second string MAY contain a reason string, or it may be empty.

The first string is "NEW_ENDPOINT" if registration succeeded. It is "REGISTRATION_REFUSED" in case the registration attempt was refused, and the connector SHOULD NOT try to register again. It is "REGISTRATION_FAILED" in case the registration failed for some other reason, and the connector MAY try to register again.

If registration succeeded, the distributor MUST call the connector's [org.unifiedpush.Connector1.NewEndpoint](#orgunifiedpushconnector1newendpoint-string-string--nothing) method to deliver the new push endpoint to it.

### org.unifiedpush.Distributor1.Unregister (String) → nothing

The connector calls this method to unregister for push messages, with the connection token as its argument. The method does not return anything.

## Appendix: Implementation practices

These are some practices and hints that may be useful to you if you want to implement a part of this API.

### Registration

When a connector first wants to register, it should fetch all service names currently registered on the session bus, and filter it to only those which begin with `org.unifiedpush.Distributor.`. It should then choose one of these (see below) to use as its distributor. It must then generate a random string to use as a token, and call the [org.unifiedpush.Distributor1.Register](#orgunifiedpushdistributor1register-string--string-string) method with this token as the argument. If the return value indicates "NEW_ENDPOINT", the distributor will then call the connector's [org.unifiedpush.Connector1.NewEndpoint](#orgunifiedpushconnector1newendpoint-string-string--nothing) method to deliver the endpoint to it. When this has happened, the connector saves the token and the  endpoint received in its persistent storage. Registration is now complete.

On subsequent startups, the connector should call [org.unifiedpush.Distributor1.Register](#orgunifiedpushdistributor1register-string--string-string) with the *same* token, and store the new endpoint returned, if it has changed. The connector should also implement the [org.unifiedpush.Connector1.NewEndpoint](#orgunifiedpushconnector1newendpoint-string-string--nothing) method to receive notification about a changed endpoint from the distributor at any time.

### D-Bus Activation

The application should provide an activatable D-Bus service, so that it can be activated by a new push message when it is not running.

To have your service be "activatable", so that the distributor can send it push messages while it's not running and it'll automatically start up, your app should install a D-Bus service file inside one of the D-Bus session service directories. These usually include `/usr/share/dbus-1/services/` and `$XDG_RUNTIME_DIR/dbus-1/services/`, `$XDG_DATA_DIR/dbus-1/services/`, and so on, or if you're packaging in a Flatpak for example, `/app/share/dbus-1/services/`. (See the D-Bus docs for the full info.) The file should be named after your app ID, for example `tld.yourdomain.YourApp.service`, and look like the following:

```
[D-BUS Service]
Name=tld.domain.YourApp
Exec=/usr/bin/yourapp --push-message
```

The "--push-message" flag can be anything you want here, the point of it is just that when this command is executed, it starts up in an "invisible" or "background" way so that it doesn't jump into the user's face just because a push message arrived. Instead of a command line flag, this could be an environment variable (`env SOME_VARIABLE=1 /usr/bin/yourapp`). It would be a good idea to end the process after "doing its thing" in the background (for example sending a desktop notification), so that the app does not stay running forever in the background after receiving one notification.

### Choosing a distributor

When you list all services that begin with `org.unifiedpush.Distributor.` on the session bus, you may get zero, one, or multiple results. If there are zero, you can't register to get push notifications. If there is one, you can simply choose that one. If there are multiple, then you can choose one of them by presenting a UI to select one provider (since the user likely knows why they have multiple different providers running). You can also get the "preferred" distributor from an environment variable, although the name and behavior of this variable has not been specced (TODO).
