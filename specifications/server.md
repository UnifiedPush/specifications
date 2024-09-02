# UnifiedPush Server Interface

UnifiedPush Spec: SERV_1.2.0

## Endpoint

The endpoint MUST be a IRI (RFC 3987) with a scheme of HTTP or HTTPS that MAY use query parameters, a path, or both to identify the push registration. An endpoint's length MUST be less than or equal to 1000 bytes.

The endpoint contains an unguessabled identifier. This SHOULD be a 160 bits (20 bytes) random value URL-safe base64 encoded string.

## Push

### Request

The pusher sends an HTTP request to the endpoint using the POST method. The contents of the POST body will be the message received by the connector library. The length of the message MUST be between 1 byte and 4096 bytes (inclusive).


### Responses

The body and headers of a push response can be ignored.

#### 2xx

The push server MUST return a status code 201 if it successfully accepts the push message. The application server SHOULD accept status code from 200-299 as a 201.

The push server MUST add the header `TTL: 0` to its response. The application server MAY ignore it.

In this case, the distributor MUST send EXACTLY the contents of the HTTP POST request to the connector library.

### 3xx

Redirects MUST NOT be followed on push endpoints.

#### 400

Unknown error with the request, don't try again later.

#### 404

The endpoint does not exist and SHOULD NOT be used anymore. Implementing this is OPTIONAL for both the application server and push server.

#### 413

The push server MAY return a 413 if the push message payload is too large. The payload length MUST be less than or equal to 4096 bytes.

#### 429

The push server MAY return a 429 if the endpoint is receiving too many requests. Application servers SHOULD honor limits per endpoint, not host. On receipt of a 429, the application server SHOULD slow down or risk losing notifications.

#### Other

Any other HTTP responses mean an unknown error occurred. The push request MAY be tried later at the application server's discretion.

## Appendix: Implementation practices

### SSRF

We strongly recommend implementing basic SSRF protection on your application:

1. As specified in the endpoint section, push server must only be accessed over HTTP or HTTPS: application servers and gateways should restrict schemes to http:// and https://.
2. The domain name should be resolved and IPs should be checked such that they're not in a private range. For example 127.0.0.1/8, ::1, 192.168.0.0/24, 10.0.0.0/8, fc00::/7.
3. As defined in the section about responses, 3xx response codes and redirections must not be followed.
