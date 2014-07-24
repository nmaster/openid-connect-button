openid-connect-button
==

An OpenID Connect Button to instrument Web pages with OpenID Connect authentication and access to user information using an external OpenID Connect Provider via the OpenID Connect [implicit flow](http://openid.net/specs/openid-connect-core-1_0.html#ImplicitFlowAuth). 

Adding the OpenID Connect Button to a Web Page
==
In this small tutorial we demonstrate how to instrument an arbitrary Web page client with the OpenID Connect Button to enable authentication and access to user information. The button is automatically rendered based on a couple of attributes that define OpenID Connect-relevant configuration details. Developers receive access to different OpenID Connect related information such as provider configuration, tokens, and user information. 

Try it
--
TODO: add link to working online demo.

Add the OpenID Connect Button
--
When users of a client sign in with an OpenID Connect provider, the client gets an access token, which can be further used to make calls to service APIs capable of interacting with the OpenID Connect provider.

Adding the OpenID Connect button to a Web page client is done in four simple steps:

- __Step 1: Register page as OpenID Connect client.__
- __Step 2: Include OpenID Connect Button script on client page.__
- __Step 3: Add HTML element representing button to the client page.__
- __Step 4: Handle sign in with JavaScript callback.__

###Step 1: Register page as OpenID Connect client
First step is to make the OpenID Connect server aware of your client page. Necessary prerequisite is that your page is deployed under a publicly accessible URL, ideally hosted on a secure HTTP server. Therefore, you need to provide a couple of details about your client to the OpenID Connect server (most importantly a redirect URI). In turn, the server generates a client ID and a secret to be used in later steps.

OpenID Connect client registration looks different on different server implementations. In this section, we walk you through the dialogs provided by the Open Source [MITREid Connect Server](https://github.com/mitreid-connect/) version 1.1.8 we have been using to develop.

1. Log in to the OpenID Connect server (register for an account, if necessary).
1. In the *Developer* section in the menu on the left choose __*Self-service client registration*__.
1. Click the button __*Register a new client*__.
1. A page __*New Client*__ with six tabs *Main*, *Access*, *Credentials*, *Crypto*, *Other*, and *JSON* will open. In the next steps, configure your client on each of the tabs. Be sure to press *__Save__* after completing every tab!
-- 1. Tab *Main* 
---- 1. enter an arbitrary *Client name*
---- 1. paste the deploy URL of your client page as *Redirect URI* and click the "+" button
---- 1. optionally fill in all other fields
-- 1. Tab *Access*
---- 1. for *Grant Types* choose __*implicit*__
---- 1. for *Response Types* check fields __*token*__, __*id_token*__, and __*token id_token*__ (uncheck all other boxes)
-- 1. Tab *Credentials*: keep *Token Endpoint Authentication Method* on __*Client Secret over HTTP Basic*__
-- 1. Tab *Crypto*: leave all fields on __*Use server default*__
-- 1. Tab *Other*: nothing to do here for now
-- 1. Tab *JSON*: shows a JSON representation of your client configuration
1. Go back to tab *Main* and copy values for *Client ID* and *Registration Access Token*.
1. __Store values for Client ID and Registration Access Token in a safe place! You will need them for using the OpenID Connect Button and for any re-configuration of your OpenID Connect client!__
1. In case you have to re-configure your client, select __*Self-service client registration*__ from the *Developer* section in the menu on the left, enter Client ID and Registration Access Token in the fields on the right and click the button __*Edit an existing client*__. If you are an administrator of the OpenID Connect server, you can make use of the menu entry __*Manage Clients*__ in the *Administrative* section in the menu on the left.

###Step 2: Include OpenID Connect Button script on client page

Include the following script just before the closing body tag:
```html
<!-- Place this asynchronous JavaScript just before your </body> tag -->
<script type="text/javascript">
  (function() {
    var po = document.createElement('script'); 
    po.type = 'text/javascript'; 
    po.async = true;
    po.src = '';
    var s = document.getElementsByTagName('script')[0]; 
    s.parentNode.insertBefore(po, s);
  })();
</script>
```

###Step 3: Add HTML element representing button to the client page

Include a HTML element that represents the OpenID Connect Button. The script included in the previous step will transform the element into a button appearance.
```html
<span class="oidc-signin"
	data-callback="signinCallback"
	data-name="Learning Layers"
	data-logo="http://learning-layers.eu/wp-content/themes/learninglayers/images/logo.png"
	data-server="http://cloud15.dbis.rwth-aachen.de:9085/oidc"
	data-clientid="e51787a0-8ffd-480a-a0c8-66a627fe6e24"
	data-scope="openid phone email address profile">
</span>
```
The HTML element must define the following data attributes:

| Attribute Name       | Description |
| ----------------     |-------------|
| *data-name*     | Name of the OpenID Connect Provider |
| *data-logo*     | URL of an OpenID Connect Provider logo |
| *data-server*   | URL of the OpenID Connect Provider server | 
| *data-clientid* | OpenID Connect client ID as retrieved in Step 1 |
| *data-scope*    | Space-separated OpenID Connect scopes. The standard scope is simply "openid", but other scopes are usually also available (e.g. email, address, profile). A full list of scopes supported by the OpenID Connect provider is available via OpenID Connect discovery of provider configuration (see below for more information) |
| *data-callback* | Name of a callback function defined in a script tag of the client page handling the outcome of the sign in process done by the button (see next step). |

Step 4: Handle sign in with JavaScript callback
--
In a script of your client page define a JavaScript function that is triggered after the OpenID Connect Button is loaded. The name of the function must match the value of the *data-callback* attribute of the HTML element defined in Step 3. The function is passed an object that represents the authorization result.

When sign in is successful, the result simply contains the string "success". At this point, you have access to the following variables representing OpenID Connect-related information:

| Variable Name  | Description |
|----------------|-------------|
| *oidc_server*  | OpenID Connect Provider server URL |
| *oidc_name*    | OpenID Connect Provider name |
| *oidc_logo*    | OpenID Connect Provider logo URL |
| *oidc_clientid*| OpenID Connect client ID |
| *oidc_scope*   | OpenID Connect scope |
| *oidc_callback*| OpenID Connect Button sign in callback function |
| *oidc_provider_config* | OpenID Connect Provider configuration as retrieved via [OpenID Connect Discovery](http://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationResponse)
| *oidc_userinfo* | OpenID Connect user info claim as retrieved from the [OpenID Connect User Info](http://openid.net/specs/openid-connect-core-1_0.html#UserInfoResponse) endpoint.
| *oidc_idtoken* | OpenID Connect ID token payload (cf. [OpenID Connect Core](http://openid.net/specs/openid-connect-core-1_0.html#IDToken))|
If the user is not signed-in, the result represents the respective error message describing the cause of the failed sign in. Causes include authentication errors, denial of access to user information expressed by user in OpenID Connect consent dialog, invalid tokens, etc.

The following example of a callback function greets the signed in user with a welcome message displayed in an HTML element with id "status" in case sign in succeeded and in case of an error logs the cause on the console.

```js
function signinCallback(result) {
	if(result === "success"){
	    // after successful sign in, display a welcome string for the user
		$("#status").html("Hello, " + oidc_userinfo.name + "!");
	} else {
	    // if sign in was not successful, log the cause of the error on the console
		console.log(result);
	}
}
```
License
--
The OpenID Connect Button is released under a 3-clause BSD license.