---
layout: docs_page
title: Sessions
redirect_from: "/docs/api/rest/sessions.html"
---

## Overview

Okta uses a cookie-based authentication mechanism to maintain a user's authentication session across web requests.  The Okta Sessions API provides operations to create and manage authentication sessions for users in your Okta organization.

### Session Cookie

Okta utilizes a HTTP session cookie to provide access to your Okta organization and applications across web requests for an interactive user-agents such as a browser.  Session cookies have an expiration configurable by an administrator for the organization and are valid until the cookie expires or the user closes the session (logout) or browser application.

### Session Token

A [session token](./authn.html#session-token) is bearer token that provides proof of authentication and may be redeemed for an interactive SSO session in Okta in a user agent.  Session tokens can only be used **once** to establish a session for a user and are revoked when the token expires.

Okta provides a very rich [Authentication API](./authn.html) to validate a [user's primary credentials](./authn.html#primary-authentication) and secondary [MFA factor](./authn.html#verify-factor). A one-time [session token](./authn.html#session-token) is returned after successful authentication which can be later exchanged for a session cookie using one of the following flows:

- [Retrieving a session cookie by visiting a session redirect link](/docs/examples/session_cookie.html#retrieving-a-session-cookie-by-visiting-a-session-redirect-link)
- [Retrieving a session cookie by visiting an application embed link](/docs/examples/session_cookie.html#retrieving-a-session-cookie-by-visiting-an-application-embed-link)
- [Retrieving a session cookie embedding a hidden image](/docs/examples/session_cookie.html#retrieving-a-session-cookie-with-a-hidden-image)

> **Session Tokens** are secrets and should be protected at rest as well as during transit. A session token for a user is equivalent to having the user's actual credentials

## Session Model

### Example

~~~ json
{
    "id": "000najcYVnjRS2aZG50MpHL4Q",
    "userId": "00ubgaSARVOQDIOXMORI",
    "mfaActive": false
}
~~~

### Session Properties

Sessions have the following properties:

|-----------+-----------------------------------------------------------------------------------------------+----------+----------+--------+----------+-----------+-----------+------------|
| Property  | Description                                                                                   | DataType | Nullable | Unique | Readonly | MinLength | MaxLength | Validation |
| --------- | --------------------------------------------------------------------------------------------- | -------- | -------- | ------ | -------- | --------- | --------- | ---------- |
| id        | unique key for the session                                                                    | String   | FALSE    | TRUE   | TRUE     |           |           |            |
| userId    | unique key for the [user](users.html#get-user-with-id)                                        | String   | FALSE    | TRUE   | TRUE     |           |           |            |
| mfaActive | indicates whether the user has [enrolled a MFA factor](./factors.html#list-enrolled-factors)  | Boolean  | FALSE    | FALSE  | TRUE     |           |           |            |
|-----------+-----------------------------------------------------------------------------------------------+----------+----------+--------+----------+-----------+-----------+------------|

#### Optional Session Properties

The [Create Session](#create-session) operation can optionally return the following properties when requested.

|----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Property       | Description                                                                                                                                                                        |
| -------------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| cookieToken    | Another one-time token which can be used to obtain a session cookie by visiting either an application's embed link or a session redirect URL.                                      |
| cookieTokenUrl | URL for a a transparent 1x1 pixel image which contains a one-time session token which when visited sets the session cookie in your browser for your organization.                  |
|----------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|

> The `cookieTokenUrl` is deprecated as modern browsers block cookies set via embedding images from another origin (cross-domain)

## Session Operations

### Create Session with Session Token
{:.api .api-operation}

<span class="api-uri-template api-uri-post"><span class="api-label">POST</span> /sessions</span>

Creates a new session for a user with a valid session token.  Only use this operation if you need the session `id`, otherwise you should use one of the following flows to obtain a SSO session with a `sessionToken`

- [Retrieving a session cookie by visiting a session redirect link](/docs/examples/session_cookie.html#retrieving-a-session-cookie-by-visiting-a-session-redirect-link)
- [Retrieving a session cookie by visiting an application embed link](/docs/examples/session_cookie.html#retrieving-a-session-cookie-by-visiting-an-application-embed-link)

> This operation can be performed anonymously without an API Token

##### Request Parameters
{:.api .request-parameters}

Parameter        | Description                                                   | Param Type | DataType                        | Required | Default
---------------- | ------------------------------------------------------------- | ---------- | ------------------------------- | -------- | -------
additionalFields | Optional [session properties](#optional-session-properties)   | Query      | String (comma separated values) | FALSE    |
sessionToken     | Session token obtained via [Authentication API](./authn.html) | Body       | String                          | TRUE     |

> Creating a session with `username` and `password` has been deprecated.  Use the [Authentication API](./authn.html) to obtain a `sessionToken`

##### Response Parameters
{:.api .api-response .api-response-params}

The new [Session](#session-model) for the user if the `sessionToken` was valid.

Invalid `sessionToken` will return a `401 Unauthorized` status code.

~~~ http
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
    "errorCode": "E0000004",
    "errorSummary": "Authentication failed",
    "errorLink": "E0000004",
    "errorId": "oaeVCVElsluRpii8PP4GeLYxA",
    "errorCauses": []
}
~~~

##### Request Example
{:.api .api-request .api-request-example}

~~~sh
curl -v -X POST \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-d '{
  "sessionToken": "00HiohZYpJgMSHwmL9TQy7RRzuY-q9soKp1SPmYYow"
}' "https://${org}.okta.com/api/v1/sessions"
~~~

##### Response Example
{:.api .api-response .api-response-example}

~~~ json
{
  "id": "000rWcxHV-lQUOzBhLJLYTl0Q",
  "userId": "00uld5QRRGEMJSSQJCUB",
  "mfaActive": false
}
~~~

### Extend Session
{:.api .api-operation}

<span class="api-uri-template api-uri-put"><span class="api-label">PUT</span> /sessions/*:id*</span>

Extends the lifetime of a user's session.

##### Request Parameters
{:.api .api-request .api-request-params}

Parameter | Description                            | Param Type | DataType | Required | Default
--------- | -------------------------------------- | ---------- | -------- | -------- | -------
id        | `id` of a valid session                | URL        | String   | TRUE     |

##### Response Parameters
{:.api .api-response .api-response-params}

[Session](#session-model)

Invalid sessions will return a `404 Not Found` status code.

~~~ http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
    "errorCode": "E0000007",
    "errorSummary": "Not found: Resource not found: 000rWcxHV-lQUOzBhLJLYTl0Q (AppSession)",
    "errorLink": "E0000007",
    "errorId": "oaeAu0LCZaeRMaJqzQ3OzFuow",
    "errorCauses": []
}
~~~

##### Request Example
{:.api .api-request .api-request-example}

~~~sh
curl -v -X PUT \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: SSWS ${api_token}" \
"https://${org}.okta.com/api/v1/sessions/000NyyOduusQ2ibzaJUTPUqhQ"
~~~

##### Response Example
{:.api .api-response .api-response-example}

~~~ json
{
  "id": "000rWcxHV-lQUOzBhLJLYTl0Q",
  "userId": "00uld5QRRGEMJSSQJCUB",
  "mfaActive": false
}
~~~

### Close Session
{:.api .api-operation}

<span class="api-uri-template api-uri-delete"><span class="api-label">DELETE</span> /sessions/*:id*</span>

Closes a user's session (logout).

##### Request Parameters
{:.api .api-request .api-request-params}

Parameter | Description             | Param Type | DataType | Required | Default
--------- | ----------------------- | ---------- | -------- | -------- | -------
id        | `id` of a valid session | URL        | String   | TRUE     |

##### Response Parameters
{:.api .api-response .api-response-params}

~~~ http
HTTP/1.1 204 No Content
~~~

Invalid sessions will return a `404 Not Found` status code.

~~~ http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
    "errorCode": "E0000007",
    "errorSummary": "Not found: Resource not found: 000rWcxHV-lQUOzBhLJLYTl0Q (AppSession)",
    "errorLink": "E0000007",
    "errorId": "oaeAu0LCZaeRMaJqzQ3OzFuow",
    "errorCauses": []
}
~~~

##### Request Example
{:.api .api-request .api-request-example}

~~~sh
curl -v -X DELETE \
-H "Accept: application/json" \
-H "Content-Type: application/json" \
-H "Authorization: SSWS ${api_token}" \
"https://${org}.okta.com/api/v1/sessions/000NyyOduusQ2ibzaJUTPUqhQ"
~~~

##### Response Example
{:.api .api-response .api-response-example}

~~~ http
HTTP/1.1 204 No Content
~~~
