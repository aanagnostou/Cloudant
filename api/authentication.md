---

copyright:
  years: 2015, 2020
lastupdated: "2020-04-02"

keywords: basic authentication, cookie authentication, request cookie, delete cookie, get cookie

subcollection: cloudant

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:codeblock: .codeblock}
{:pre: .pre}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:deprecated: .deprecated}
{:external: target="_blank" .external}

<!-- Acrolinx: 2020-03-17 -->

# Authentication
{: #authentication}

Authentication means proving who you are.
You prove your identity by supplying your user credentials for verification.
{: shortdesc}

You can provide your credentials (authenticate) in two ways for {{site.data.keyword.cloudantfull}}:

-	[Basic authentication](/docs/Cloudant?topic=cloudant-authentication#basic-authentication)
-	[Cookie authentication](/docs/Cloudant?topic=cloudant-authentication#cookie-authentication)

Basic authentication is similar to showing an ID at a door to be checked every time that you want to enter.
Cookie authentication is similar to having a key to the door so that you can let yourself in whenever you want.
Within {{site.data.keyword.cloudant_short_notm}},
the key is a cookie that is named `AuthSession`.

When you create or use performance-critical {{site.data.keyword.cloudant_short_notm}} applications, cookie authentication has more benefits when compared with Basic authentication. We recommend that you use cookie authentication whenever possible.
{: note}

## Basic authentication
{: #basic-authentication}

To use Basic authentication,
pass your credentials as part of every request.
You pass your credentials by adding an `Authorization` header to the request.

The header includes the authentication scheme (`Basic`),
followed by the [BASE64](https://en.wikipedia.org/wiki/Base64){: new_window}{: external} encoding of a string created by concatenating:

-	Your username
-	The `:` character
-	Your password

In practice,
many application libraries that are used for creating HTTP requests can do this encoding for you.

The following example includes basic authentication credentials in a request by using HTTP:

```http
GET /db/document HTTP/1.1
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```
{: codeblock}

The following example includes basic authentication credentials in a request by using the command line:

```sh
curl "https://$ACCOUNT:$PASSWORD@$ACCOUNT.cloudant.com"
```
{: codeblock}

<!--

The following example includes basic authentication credentials in a request, using Javascript

```javascript
var nano = require('nano');
var account = nano("https://$ACCOUNT:$PASSWORD@$ACCOUNT.cloudant.com");
account.request(function (err, body) {
	if (!err) {
		console.log(body);
	}
});
```
{: codeblock}

-->

The following example includes basic authentication credentials in a request by using Python:

```python
import cloudant
url = "https://{0}:{1}@{0}.cloudant.com".format(USERNAME, PASSWORD)
account = cloudant.Account(url)
ping = account.get()
print ping.status_code
# Expected result code: 200
```
{: codeblock}

## Cookie authentication
{: #cookie-authentication}

Cookie authentication requires that you supply a valid username and password once,
at the start of a series of tasks (the session).
The response includes a cookie,
generated by the server,
that confirms you successfully authenticated.
After the cookie is created,
you send the cookie with all requests that require authentication.

The presence of a valid cookie is sufficient to ensure that your call request is processed
rather than rejected immediately as unauthenticated.
You can use the cookie until it expires.

Using the `DELETE` method logs out the user because the cookie is considered expired by the server.

Method | Path | Description | Headers | Form Parameters
-------|------|-------------|---------|----------------------
`POST` | `/_session` | Do cookie-based user login. | `Content-Type: application/x-www-form-urlencoded` | `name`, `password`
`GET` | `/_session` | Returns cookie-based login user information. | AuthSession cookie returned by POST request. | 
`DELETE` | `/_session` | Log out cookie-based user. | AuthSession cookie returned by POST request. | 
{: caption="Table 1. Cookie authentication and methods" caption-side="top"}

### Requesting a cookie
{: #requesting-a-cookie}

With cookie authentication,
you use your credentials to acquire a cookie.
Acquire a cookie by sending a `POST` request to `/_session`.

The following example shows a request for a cookie by using HTTP:

```http
POST /_session HTTP/1.1
Content-Length: 32
Content-Type: application/x-www-form-urlencoded
Accept: */*
name=USERNAME&password=PASSWORD
```
{: codeblock}

The following example shows a request for a cookie by using the command line:

```sh
curl "https://$ACCOUNT.cloudant.com/_session" \
	-X POST \
	-c /path/to/cookiefile
	-d "name=$ACCOUNT&password=$PASSWORD"
```
{: codeblock}

<!--

The following example request for a cookie by using Javascript:

```javascript
var nano = require('nano');
var cloudant = nano("https://"+$ACCOUNT+".cloudant.com");
var cookies = {}
cloudant.auth($ACCOUNT, $PASSWORD, function (err, body, headers) {
	if (!err) {
		cookies[$ACCOUNT] = headers['set-cookie'];
		cloudant = nano({
			url: "https://"+$ACCOUNT+".cloudant.com",
			cookie: cookies[$ACCOUNT] 
		});
		// ping to ensure we're logged in
		cloudant.request({
			path: 'test_porter'
		}, function (err, body, headers) {
			if (!err) {
				console.log(body, headers);
			}
		});
	}
});
```
{: codeblock}

-->

The following example shows a request for a cookie by using Python:

```python
import cloudant
account = cloudant.Account(USERNAME)
login = account.login(USERNAME, PASSWORD)
print login.status_code
# Expected result code: 200
logout = account.logout()
print logout.status_code
# Expected result code: 200
all_dbs = account.all_dbs()
print all_dbs.status_code
# Expected result code: 401
```
{: codeblock}

If the credentials you supply in your cookie request are valid,
the response includes a cookie that stays active for 24 hours.

#### Reply to request for a cookie

```http
200 OK
Cache-Control: must-revalidate
Content-Length: 42
Content-Type: text/plain; charset=UTF-8
Date: Mon, 11 Nov 2016 12:43:15 GMT
server: CouchDB/1.0.2 (Erlang OTP/R14B)
Set-Cookie: AuthSession="d2FybWFuYTo1ODI1QkM2NzpZelovo2epvx9cfaDdxJGNLuzBzw"; Expires=Sat, 12-Nov-2016 12:43:15 GMT; Max-Age=86400; Path=/; HttpOnly; Version=1
x-couch-request-id: a638431d
```
{: codeblock}

The following example shows the JSON part of the response:

```json
{
	"ok": true,
	"name": "USERNAME",
	"roles": []
}
```
{: codeblock}

### Getting cookie-authenticated information
{: #getting-cookie-authenticated-information}

When a cookie is set,
information about the authenticated user can be retrieved with a `GET` request.

The following example shows the request for cookie information by using HTTP:

```http
GET /_session HTTP/1.1
Cookie: AuthSession="d2FybWFuYTo1ODI1QkM2NzpZelovo2epvx9cfaDdxJGNLuzBzw"; Expires=Sat, 12-Nov-2016 12:43:15 GMT; Max-Age=86400; Path=/; HttpOnly; Version=1
Accept: application/json
```
{: codeblock}

The following example shows the request for cookie information by using the command line:

```sh
curl "https://$ACCOUNT.cloudant.com/_session" \
	-X GET \
	-b /path/to/cookiefile
```
{: codeblock}

The response includes the username,
the user's roles,
and which authentication mechanism was used.

The following example shows the response to a request for cookie information:

```json
{
	"userCtx": {
		"roles": [
			"_admin",
			"_reader",
			"_writer"
		],
		"name": "USERNAME"
	},
	"ok": true,
	"info": {
		"authentication_db": "_users",
		"authentication_handlers": [
			"delegated",
			"cookie",
			"default",
			"local"
		],
		"authenticated": "cookie"
	}
}
```
{: codeblock}

### Deleting a cookie
{: #deleting-a-cookie}

You can end the cookie authentication session by sending a `DELETE` request to the same URL used to create the cookie.
The `DELETE` request must include the cookie that you want to delete.

The following example shows a cookie `DELETE` request by using HTTP:

```http
DELETE /_session HTTP/1.1
Cookie: AuthSession="d2FybWFuYTo1ODI1QkM2NzpZelovo2epvx9cfaDdxJGNLuzBzw"; Expires=Sat, 12-Nov-2016 12:43:15 GMT; Max-Age=86400; Path=/; HttpOnly; Version=1
Accept: application/json
```
{: codeblock}

The following example shows a cookie `DELETE` request by using the command line:

```sh
curl "https://$ACCOUNT.cloudant.com/_session" \
	-X DELETE \
	-b /path/to/cookiefile
```
{: codeblock}

The response confirms deletion of the session,
and sets the `AuthSession` Cookie to `""`.

The following example shows the response to a cookie `DELETE` request:

```http
200 OK
Cache-Control: must-revalidate
Content-Length: 12
Content-Type: application/json
Date: Mon, 11 Nov 2016 14:06:12 GMT
server: CouchDB/1.0.2 (Erlang OTP/R14B)
Set-Cookie: AuthSession=""; Expires=Fri, 02-Jan-1970 00:00:00 GMT; Max-Age=0; Path=/; HttpOnly; Version=1
x-couch-request-id: e02e0333
```
{: codeblock}

The following example shows the JSON part of the response:

```json
{
	"ok": true
}
```
{: codeblock}
