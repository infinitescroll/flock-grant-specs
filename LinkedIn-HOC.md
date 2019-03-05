# Pulling LinkedIn Data HOC Documentation

The LinkedIN API will be required by the autark/profiles app and used to get user info from LinkedIN. Must be used in tandem with the auth-server (coming soon)

This will be a higher order component that passes the linkedIn access token to the callback passed to the HOC

## Setup

### Install autark-linkedin-hoc

```
$ npm install autark-linkedin-hoc
```

### Import methods from autark-linkedin-hoc into your project

```js
import LinkedIn from 'autark-linkedin-hoc';
```

## Process

When a user wants to integrate their LinkedIn information into, the following steps need to happen:

1. The user gets sent to a specific linkedIn permissions page
2. the user "accepts" permission for us to access their linkedIn info
3. The user gets redirected to a new URL, with a special "code" in the URL bar
4. Take the special "code" from the new URL, and send it to the auth server
5. auth server receives special "code" from client, and sends a request for an access token to LinkedIn
6. If special "code" is valid, server receives an access token
7. Server sends client access code

8. With an access token, the client can send a request to the server (with access token)
9. The server can fetch information from linkedin with the client's access token
10. return the information to the client

This library handles 1, 4, and 8

## Usage

The LinkedIn HOC takes two arguments - a component that you want to get rendered (like a button) and a callback to get invoked with the access token.

```js
const Component = <Button />;
const callback = data => console.log(data);
const IntegrateLinkedIn = LinkedIn(<Component>, <callback>);
```

then simply render the component on your page:

```js
<div>
  {...}
  <IntegrateLinkedIn />
</div>
```

When the access token is granted by LinkedIn and received by the client, we invoke your callback passing an object with the access token:

```js
{
  access_token: <String>,
  expires: <String>
}
```
