# Universal access API

Now that we finally have some data stored in our `Pod` all in a structured manner so we know how to access it, it is time to see how we can share some of our resources with other users based on their `webId`. In this section we will be looking at the `universal access API` which is part of `solid-client` and allows us to check who has access to a given resource as well and change it given that we have all the necessary rights.

In our case we are going to be looking at a resource that we just created and stored in our `Pod` so as I resource owner we should have all the required access rights already.

First let's see how we can access the `access` :eyes: object and it's methods.

```js
import { access } from "@inrupt/solid-client";
```

With this we have some useful `get` and `set` methods available for **public**, **agent** and **group** access. The public access ones will simply tell is the given resource can be publicly accessible as wel las give us as the resource owner the ability to set our resource to be publicly accessible.

## Retrieve Access Data for a Resource

To check public access for a resource we could so something like the following assuming that we are already authenticated:

```js
import { access } from "@inrupt/solid-client";

access.getPublicAccess(
  "https://example.com/resource",   // Resource
  { fetch: fetch }                  // fetch function from authenticated session
).then(access => {
  if (access === null) {
    console.log("Could not load access details for this Resource.");
  } else {
    console.log("Returned Access:: ", JSON.stringify(access));
    console.log("Everyone", (access.read ? 'CAN' : 'CANNOT'), "read the Resource.");
    console.log("Everyone", (access.append ? 'CAN' : 'CANNOT'), "add data to the Resource.");
    console.log("Everyone", (access.write ? 'CAN' : 'CANNOT'), "change data in the Resource.");
    console.log("Everyone", (access.controlRead ? 'CAN' : 'CANNOT'), "see access to the Resource.");
    console.log("Everyone", (access.controlWrite ? 'CAN' : 'CANNOT'), "change access to the Resource.");
  }
});
```

The returned access object will look like this:

```json
{
  read: <boolean>,
  append: <boolean>,
  write: <boolean>,
  controlRead: <boolean>,
  controlWrite: <boolean>
}
```

The approach is really similar when checking access for a specific `agent` or `group` (a group is a specific `webId` which represents a list of respective agent `webId` as a group), we just need to pass the agents `webId` as an additional parameter like the below:

```js
import { access } from "@inrupt/solid-client";

access.getAgentAccess(
  "https://example.com/resource",       // Resource
  "https://example.pod/profile#webId",  // Agent
  { fetch: fetch }                      // Fetch function from authenticated session
).then(access => {
  ...
}
```

Again building on the example from before let's use the `readDataset` method we had to retrieve a resource and simply check what access is set for it.

:::tip
For the following two examples you will need to have an additional `webId` for a user other than yourself **the resource owner**
:::

<CodeSandbox src="https://codesandbox.io/embed/solid-universal-access-api-get-8lcm5?fontsize=14&hidenavigation=1&theme=dark" />

## Changing Access Data for a Resource

And finally to change the access for a given resource we can again continue from the previous example and add another function which will pass along an extra access object and call the relevant method from the `access` API.

For this purpose you can provide an agent `webId` other then the one from the resource owner and check what access is set for it. As we have just uploaded this new resource all access should be set to false. Upon clicking `set agent read access` we simply update the read access for this resource and the given `webId` to be `true`. The change in the access info should be displayed on the page as well as trackable in the network tab.

<CodeSandbox src="https://codesandbox.io/embed/solid-universal-access-api-set-lq4z0?fontsize=14&hidenavigation=1&theme=dark" />