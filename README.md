login-engine
============

Portable login/authentication system using OAuth2.

Using this core
============

Imagine a system implemented in a portable fashion, such as [Engine's Billing Module](https://github.com/LDEngine/Billing).  Since this module cannot contain any login-specific logic (lest it become unportable), there must be a system allowing it to interact with an external login system.  This system contains the following points of integration:

1. The billing module reserves an ``iframe`` component on its landing page where login controls provided by the login system will be nested.  The ``src`` attribute of the ``iframe`` tag will point to the login system, and include the ``return_url``, which is where the user should be redirected when login is completed.  
2. As a security measure, the login system should be aware of the domains to which redirect is valid, and refuse to serve the login components to anyone who does not provide a valid ``redirect_url``.
3. The login mechanism should be "OAuth2-style" (Ideally ACTUALLY OAuth2).  The billing module has an appID and appSecret it can use to identify itself.
4. After the user's browser authenticates with the iframe served from the login module, the user is given a token which their browser provides back to the billing module using the redirect link.
5. The billing module uses this token to recover the user's information directly from the login server via the ``/getLoggedInUser?token=token`` endpoint.  This call is made via HTTPS for security reasons.
6. Included in the User's profile is a userID, which is an immutable ID used to identify the user.  It need not be human-readable, but must not be the same as the login token, as the login token can expire.
7. The ID token can be expired explicitly (as in a "logout" call) with the ``/expireToken?token=token`` endpoint.

External Data
=============

Given the open-source nature of this component, there is some external configuration information which must be provided at startup.  This could come from config file or database, but should not be checked into this repository:

```
{
	"apps": {
		"<APPID>": {
			"name": "<APP NAME>",
			"secret" <SECRET>"",
			"login_redirect_domains": [
				"my.domain.one", "another.valid.com", "localhost"
			]
		}
	}
}
```

Mocking this Module
==============

It can be inconvenient to work against a functional login system, and even more so to run automated tests against such a system.  For simplicity's sake, applications built using this module can mock it by providing the following:

1. A configuration file which points authentication calls to the local system (``localhost:3000`` instead of ``id.mylivesystem.com``, for example)
2. A local endpoint that serves out a simple "login" page with dummy components and a simple field that lets the user specify their login name.  The login name should be stored in memory or database or wherever is appropriate.
3. A local endpoint that returns a dummy user profile in response to calls to ``/getLoggedInUser``.
4. A local enpoint that clears the login name set in response to calls to ``/expireToken``.

