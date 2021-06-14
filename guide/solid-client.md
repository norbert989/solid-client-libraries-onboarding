# solid-client

::: tip
Before you read on, consider the prerequisites below:

1. Create your own [pod](https://solidproject.org/users/get-a-pod) in order to store/access data to and from it.

2. Create a new react app using `npx create-react-app your-awesome-app` so you can start using the libraries hands on based on some handy examples.
:::

The `solid-client` library is used to manage data in our pod. The first and easiest step to see this in action would be to access the public data about a users profile using their `webId`.

> For full details on the `solid-client` read the official inrupt docs on [solid-client](https://docs.inrupt.com/developer-tools/javascript/client-libraries/structured-data/)

In order to start reading data from a pod we will need to use a couple of methods from `solid-client`

<CodeSandbox src="https://codesandbox.io/embed/solid-client-public-data-j0w88?fontsize=14&hidenavigation=1&theme=dark" />

In the above example we are requesting some publicly accessible information about the user, so we don't need to be authenticated. We use the provided `webId` which in this case by default is the profile of the inrupt docs team found at `https://pod.inrupt.com/docsteam/profile/card#me`. If we removing the `#me` part we can fetch the public data that is stored at `https://pod.inrupt.com/docsteam/profile/card` **URI** representing the profile of that user.

In the example folder structure you can check the `utils/readProfile.js` function to see how we work out the two pieces of data using methods from `solid-client`. By observing the network tab you can also see this information being returned as an `rdf` resource.

If we want to start managing private data we need to be authenticated and pass along some options when fetching data. By combining the above two examples we can build a super simple application that will provide a way to authenticate using our `webId` and start reading and writing structured data in our `Pod`.

## Read/Write Structured Data

First let's have a look at the below example that allows us to log in and write a piece of data into a specified location in our `Pod`.
For this example we will be using `@inrupt/vocab-common-rdf` to easily define some information we want to store as well as retrieve.

> For full details on `vocab-common-rdf` read the official inrupt docs on [vocabulary](https://docs.inrupt.com/developer-tools/javascript/client-libraries/tutorial/use-vocabularies/)

As we will be reading and writing data in a structured matter you may also want to read full details on [structured data](https://docs.inrupt.com/developer-tools/javascript/client-libraries/structured-data/).

Let's say we want to write some data into our pods representing a `vCard` also known as a virtual contact card. We might want to store multiple of these cards at some point. In order to achieve that we will set up some structure. In our **vCard** we want to store some information like name, nickname, title and email.

A typical `vCard` would look like the below data

```json
  <v:VCard>
    <v:fn>Bruce Wayne</v:fn>
    <v:nickname>Batman</v:nickname>
    <v:title>Billionaire, philanthropist, industrialist and vigilante</v:title>
    <v:email>batman@cave.com</v:email>
  </v:VCard>
```

To start off we will expand on the previous `auth` example to obtain a few things.

## Retrieving the `webId` and `podRoot`

After login we want to retrieve the users `webId` and work out the base URI of their `Pod`

```js{9,23-26}
// login.js

import {
  login,
  handleIncomingRedirect,
  getDefaultSession,
} from "@inrupt/solid-client-authn-browser";

import { getSolidDataset, getThing, getUrl } from "@inrupt/solid-client";

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
    const webId = session.info.webId;
    const profileDoc = await getSolidDataset(session.info.webId);
    const profile = getThing(profileDoc, session.info.webId);
    const podRoot = getUrl(profile, "http://www.w3.org/ns/pim/space#storage");
  }
}

export {
  auth,
  handleRedirectAfterLogin
};
```
We have now expanded the redirect handler with some logic to extract the two values we will be using later on to manage data.

> The complete docs on the `solid-client` API are available at [solid-client API](https://docs.inrupt.com/developer-tools/api/javascript/solid-client/index.html).

## Creating a container in the `Pod`

Once we have the `webId` and `podRoot` the next step is to decide on a location where we aim to save our data. For this example since we want to save one or more `vCard` resources we will save them in our `podRoot` under a `vCards` container. So the location of our container will be a url formed of the `podRoot` followed by `vCards`. 

Let's see how we can create a container using the `solid-client` API

```js{5,14,34-42}
// login.js

import {
  login,
  fetch,
  handleIncomingRedirect,
  getDefaultSession,
} from "@inrupt/solid-client-authn-browser";

import {
  getSolidDataset,
  getThing,
  getUrl,
  createContainerAt
} from "@inrupt/solid-client";

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
    const webId = session.info.webId;
    const profileDoc = await getSolidDataset(session.info.webId);
    const profile = getThing(profileDoc, session.info.webId);
    const podRoot = getUrl(profile, "http://www.w3.org/ns/pim/space#storage");

    // Create a dataset Container
    try {
    const savedDataSetContainer = await createContainerAt(`${podRoot}vcards/`, {
      fetch: fetch,
    });

  } catch (error) {
    console.log(error);
  }
  }
}

export {
  auth,
  handleRedirectAfterLogin
};
```

Note that we are now passing in `fetch` as an option since we are no longer accessing public data. We should now have a container created where we can store our `vCard`.

## Creating a `vCard` and storing it in the `Pod`

Let's have a look at creating a `dataSet` first and then adding some data into it.

```js
import {
  createSolidDataset,
  saveSolidDatasetAt,
  createThing,
  setThing,
  addStringNoLocale,
} from "@inrupt/solid-client";

import { fetch } from "@inrupt/solid-client-authn-browser";

import { VCARD } from "@inrupt/vocab-common-rdf";

async function saveDataToPod(podRoot) {

  let dataSet = createSolidDataset();
  let vCard = createThing({ name: "vcard" });

  vCard = addStringNoLocale(vCard, VCARD.fn, "Bruce Wayne");
  vCard = addStringNoLocale(vCard, VCARD.nickname, "Batman");
  vCard = addStringNoLocale(vCard, VCARD.title, "Billionaire, philanthropist, industrialist and vigilante");
  vCard = addStringNoLocale(vCard, VCARD.email, "batman@cave.com");

  dataSet = setThing(dataSet, vCard);

  try {
    const savedDataSet = await saveSolidDatasetAt(`${podRoot}vcards/card_batman`, dataSet, {
      fetch: fetch,
    });

  } catch (error) {
    console.log(error);
  }
}

export default CreateDataset;

```

The above function will create a solid `dataSet` using the `podRoot` provided as a param. It will create a `Thing` representing our `vCard` and holding data in a form of strings which can all later be retrieved back using their respective predicates like `VCARD.fn`.

---
Below you will find a working example for writing structured data into your pod based on the above examples.

<CodeSandbox src="https://codesandbox.io/embed/solid-client-structured-data-judz1?fontsize=14&hidenavigation=1&theme=dark" />

## Retrieving a resource from the Pod

Now that we have some data in our `Pod` it's time to see how we can retrieve that data back. Since we stored our data in a structured manner it will be easy for us to read the specific resource we want. So let's use the resource uploaded from the previous example which is stored in our container under `card_batman`. All we want to do is retrieve it and list it's details on the page. In the previous example we also worked out the root of our `Pod` so building up from that example we can put together the `podRoot` with the container called `vcards` and the resource name which we build using the nickname. These three strings combined give us our resource `URI`.

To make this simple let's separate the logic for retrieving the `dataset` resource and the logic for getting the `thing` we want out of that retrieved resource.

```js
// Read dataset resource

import { getSolidDataset } from "@inrupt/solid-client";

import { fetch } from "@inrupt/solid-client-authn-browser";

async function ReadDataset(resourceUrl, setDataSet) {
  try {
    let savedDataset = await getSolidDataset(resourceUrl, {
      fetch: fetch
    });

    if (savedDataset) {
      /* Set retrieved dataSet to state */
      setDataSet(savedDataset);
    }
  } catch (error) {
    console.log(error.response);
    /* If fetching the resource fails with a 401,
      reload the page */
    if (error.response.status === 401) {
      window.location.reload();
    }
  }
}

export default ReadDataset;
```

the above :point_up: is a simple function that will accept a `resource url` and a function to set the retrieved resource into our application state for easier further management. while the below :point_down: is an example of how we can use this in an actual example to get a `thing` out of the resource. In our case the `thing` we want is a `vcard` holding the information we stored and display it on the page.

<CodeSandbox src="https://codesandbox.io/embed/solid-universal-access-api-get-5in5n?fontsize=14&hidenavigation=1&theme=dark" />