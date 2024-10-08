---
title: "Implementing PKCE Authorization Code Flow"
category: ''
pubDate: 'Feb 17 2024 21:23'
---

Authorization is a process of granting a user or an app access permissions to certain data and/or features.

There are several ways to implement authorization logic and PKCE is one of them.

Proof Key for Code Exchange, or PKCE (pronounced '_pixie_'), is an extension to the authorization code flow to prevent CSRF and authorization code injection attacks.

PKCE authorization code flow is a recommended way to use for applications running on mobile devices, single-page web applications, or any other types of applications where the client secret **cannot be stored safely**.

## Code Challenge

### generateCodeVerifier

The first step in PKCE auth flow is generating a _Code Challenge_ from a _Code Verifier_ which is a randomly generated cryptographic string of length between 48 and 128 that can consist of letters, digits, periods, tildes, and hyphens. 

```js
function generateCodeVerifier(length) {
  const possible =
    'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  const values = crypto.getRandomValues(new Uint8Array(length));
  return values.reduce((acc, x) => acc + possible[x % possible.length], '');
}
```

### generateCodeChallenge

Once the code verifier is generated, this string is hashed using SHA256 algorithm. And then, the resulting hash value is transformed into the base64 representation of its digest.

```js
async function generateCodeChallenge(codeVerifier) {
  const data = new TextEncoder().encode(codeVerifier);
  const digest = await window.crypto.subtle.digest('SHA-256', data);
  return btoa(String.fromCharCode.apply(null, [...new Uint8Array(digest)]))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=+$/, '');
}
```

## Request User Authorization

Once the code challenge is generated, we can use this value to send a request to `/authorize` endpoint.

```js
async function redirectToAuthCodeFlow() {
  const clientId = '<CLIENT_ID>';
  const verifier = generateCodeVerifier(128);
  const challenge = await generateCodeChallenge(verifier);

  localStorage.setItem('verifier', verifier);

  const params = new URLSearchParams({
    client_id: clientId,
    response_type: 'code',
    redirect_uri: '<REDIRECT_URI>'
    scope: '<SCOPE>',
    code_challenge_method: 'S256',
    code_challenge: challenge,
  });

  document.location = `${BASE_URL}/authorize?${params.toString()}`;
}
```

If the user accepts the requested permissions, the OAuth service redirects the user back to the URL specified in the `redirect_uri` field. This callback contains two parameters within the URL: `code` and `state`.

The `code` parameter in the URL can be parsed using `URLSearchParams`.

```js
const params = new URLSearchParams(window.location.search);
const code = params.get('code');
```

## Request an Access Token

We can exchange the authorization code, `code`, for an access token by sending a POST request to `/api/token` endpoint.

```js
async function getAccessToken(code) {
  const code_verifier = localStorage.getItem('verifier');

  const payload = {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: new URLSearchParams({
      client_id,
      grant_type: 'authorization_code',
      code,
      redirect_uri: '<REDIRECT_URI>',
      code_verifier,
    }),
  };
  
  const url = `${BASE_URL}/api/token`;
  const body = await fetch(url, payload);
  const { access_token } = await body.json();
  return access_token;
}
```

On success, the response to the request will contain a JSON object with the following key-value pairs:

- `access_token` - The token granting access to the requested resources.
- `token_type` - The type of the token, typically set to _Bearer_.
- `scope` - The scope of the access token.
- `expires_in` - The expiration time of the access token in seconds.
- `refresh_token` - A token that can be used to obtain a new access token when the current one expires.

## Get User Profile

You can include the token in the authorization header when making API requests.

```js
async function fetchProfile(token) {
  const request = {
    method: 'GET',
    headers: { Authorization: `Bearer ${token}` },
  };

  const USER_PROFILE_URL = 'https://api.spotify.com/v1/me';
  const userData = await fetch(USER_PROFILE_URL, request);

  return await userData.json();
}
```

## References
- Authorization Code with PKCE Flow | Spotify for Developers. (n.d.). https://developer.spotify.com/documentation/web-api/tutorials/code-pkce-flow
- Protecting Apps with PKCE - OAuth 2.0 Simplified. (2021, December 16). OAuth 2.0 Simplified. https://www.oauth.com/oauth2-servers/pkce/