# solid-client-authn-browser

::: tip
Before you read on, consider the prerequisites below:

1. Create your own [pod](https://solidproject.org/users/get-a-pod) in order to store/access data to and from it.

2. Create a new react app using `npx create-react-app your-awesome-app` so you can start using the libraries hands on based on some handy examples.
:::

To access private data on Solid Pods, you must authenticate as a user/agent who has been granted appropriate access to that data.

> For the full details on the `auth` flow read the official inrupt docs on [solid-client-authn-browser"](https://docs.inrupt.com/developer-tools/javascript/client-libraries/authentication/)

In order to authenticate in a client side application we want to do three things.
1. Define the login function that will be called upon a users interaction with the login button
2. Handle the redirect for the user coming back from the Solid identity provider (IDP)
3. Get the current session info and set some user data

For all of these processes we have functions provided by `solid-client-authn-browser`

### *example*

```js{4,5,6,11,12,13,18,19}
// login.js

import {
  login,
  handleIncomingRedirect,
  getDefaultSession,
} from "@inrupt/solid-client-authn-browser";

function auth() {
  return login({
    oidcIssuer: "https://broker.pod.inrupt.com",
    redirectUrl: window.location.href,
    clientName: 'My awesome application',
  });
}

async function handleRedirectAfterLogin() {
  await handleIncomingRedirect();
  const session = getDefaultSession();
  if (session.info.isLoggedIn) {
    // Add logic for logged in user
  }
}

export {
  auth,
  handleRedirectAfterLogin
};
```

The above code block shows how we could define a utility function that takes `login`, `handleIncomingRedirect` and `getDefaultSession` methods from `solid-client-authn-browser`.
It exposes two functions `auth` and `handleRedirectAfterLogin` which we can use in our application to initiate login and authenticate the user.

Moving forward from here the retrieved `session` object has information about the user, their `webID` and `podURL` which can be derived and used to manage data using `solid-client`.

The below `codeSandbox` will showcase how to use the **login** functionality from `solid-client-authn-browser` and attach it to a button in react.

Upon clicking the button the user is redirected to the **idp** provider to authorize access to their pod before being redirected back to the application where the active authenticated webId is displayed from the current session object.

<CodeSandbox src="https://codesandbox.io/embed/ecstatic-swartz-y5neq?fontsize=14&hidenavigation=1&theme=dark"/>

::: tip
If the login redirect fails to load in the embedded sandbox please try opening it in a separate window.
:::

Let's have a look at a complete [react](/guide/react-app-example.html) application