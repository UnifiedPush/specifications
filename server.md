# UnifiedPush Provider Interface v0

## Endpoint

The endpoint MUST be a IRI (RFC 3987) with a scheme of HTTP or HTTPS that MAY use query parameters, a path, or both to identify the push registration. An endpoint's length MUST be less than or equal to 1000 bytes.

## Discovery

On sending a GET request to the full endpoint URL (including query parameters), the push provider MUST return the following JSON:
```json
["UnifiedPush Provider"]
```

Application servers or gateways SHOULD check this when registering push to avoid SSRF. This should be parsed as a JSON array and clients should only check the 0th index, so that it remains extensible for future versions.

## Push

### Request

The pusher sends an HTTP/1.1 (?) request to the endpoint using the POST method. The contents of the POST body will be the message received by the connector library.


### Responses

#### 202

The push provider MUST return a 202 if it successfully accepts the notification. Note that this is independent of the end-user application receiving the push.

In this case, the distributor MUST send EXACTLY the contents of the HTTP POST request to the connector library.

### 3xx

Redirects MUST NOT be followed on push endpoints.

#### 400

Unknown error with the request, don't try again later.

#### 404

The endpoint does not exist and SHOULD NOT be used anymore. Implementing this is OPTIONAL for both the application server and push provider.

#### 413

The push provider MAY return a 413 if the notification payload is too large. Push providers MUST support a minimum payload size of 4000 bytes. Applications SHOULD NOT rely on push providers supporting larger payloads for critical functionality.

#### 429 

The push provider MAY return a 429 if the endpoint is receiving too many requests. Application servers SHOULD honor limits per endpoint, not host. On receipt of a 429, the application server SHOULD slow down or risk losing notifications.

#### Other 
Any other HTTP responses mean an unknown error occured and the request MAY be tried later. //when's later?
