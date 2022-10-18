---
id: access-node-alchemy
title: Access a Full Node with Alchemy
description: Make blockchain queries on Polygon using the Alchemy SDK.
keywords:
  - docs
  - matic
  - polygon
  - node
  - alchemy
  - alchemy sdk
  - query 
  - full node
image: https://matic.network/banners/matic-network-16x9.png
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import useBaseUrl from '@docusaurus/useBaseUrl';

# Getting Started

*New to Alchemy? Get access to Alchemy for free* ***[here](https://alchemy.com/?r=e68b2f77-7fc7-4ef7-8e9c-cdfea869b9b5)***.

_Estimated time to complete this guide: < 10 minutes_

------------------

## Steps to get started with Alchemy

This guide assumes you already have an [Alchemy account](https://alchemy.com/?r=e68b2f77-7fc7-4ef7-8e9c-cdfea869b9b5) and access to our [Dashboard](https://dashboard.alchemyapi.io).

**1**. [Create an Alchemy key](doc:alchemy-quickstart-guide#1key-create-an-alchemy-key)

**2**. [Make a request](doc:alchemy-quickstart-guide#2-%EF%B8%8F-make-your-first-request)

**3**. [Set up Alchemy as your client](doc:alchemy-quickstart-guide/#3-🤝-set-up-alchemy-as-your-client)

**4**. [Start building!](doc:alchemy-quickstart-guide#4-computer-start-building)

------------------

## 1.:key: Create an Alchemy Key

To use Alchemy's products, you need an API key to authenticate your requests.

You can [create API keys from the dashboard](http://dashboard.alchemyapi.io). Check out this video on how to create an app:


[![img](https://i.imgur.com/pXvdWLU.png)](https://www.youtube.com/watch?v=tfggWxfG9o0)

Or follow the written steps below:

First, navigate to the "create app" button in the "Apps" tab.


![img](https://files.readme.io/693457a-Getting_Started.png)


Fill in the details under "Create App" to get your new key. You can also see apps you previously made and those made by your team here. Pull existing keys by clicking on "View Key" for any app.

![img](https://files.readme.io/d6172a5-Create_App_Details.png)


You can also pull existing API keys by hovering over "Apps" and selecting one. You can "View Key" here, as well as "Edit App" to whitelist specific domains, see several developer tools, and view analytics.

![img](https://files.readme.io/f0dbb19-ezgif.com-gif-maker_1.gif)


-------------------

## 2. Make Your First Request

You can interact with Alchemy's Polygon infrastructure provider using JSON-RPC and your [command line](https://www.computerhope.com/jargon/c/commandi.htm).

For manual requests, we recommend interacting with the `JSON-RPC` via `POST` requests. Simply pass in the `Content-Type: application/json` header and your query as the `POST` body with the following fields:

* `jsonrpc`: The JSON-RPC version—currently, only `2.0` is supported.
* `method`: The ETH API method. [See API reference.](https://alchemyenterprisegroup.readme.io/reference/ethereum-api-quickstart)
* `params`: A list of parameters to pass to the method.
* `id`: The ID of your request. Will be returned by the response so you can keep track of which request a response belongs to.

Here is an example you can run from the Terminal/Windows/LINUX command line to retrieve the current gas price:

```console
curl https://eth-mainnet.alchemyapi.io/v2/demo \
-X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":73}'
```

:::note

Want to send requests to your own app instead of our public demo?

Replace https://eth-mainnet.alchemyapi.io/jsonrpc/demo with your own API key https://eth-mainnet.alchemyapi.io/v2/your-api-key

:::

Results:


```json
{ "id": 73,
  "jsonrpc": "2.0",
  "result": "0x09184e72a000" // 10000000000000 }
```

----------------

## 3. Set up Alchemy as your Client

Want to integrate Alchemy into your production app?

Find out how to set up or switch your current provider to Alchemy by using the Alchemy SDK, the easiest and most powerful way to access Alchemy's suite of Enhanced APIs and tools.

:::note

**If you have an existing client,** change your current node provider URL to an Alchemy URL with your API key: \"https://eth-mainnet.alchemyapi.io/v2/your-api-key\"\n\n**Note:** The scripts below need to be run in a **node context** or **saved in a file**, not run from the command line.

:::


### The Alchemy SDK

There are tons of Web3 libraries you can integrate with Alchemy. However, we recommend using the [Alchemy SDK](https://github.com/alchemyplatform/alchemy-sdk-js), a drop-in replacement for Ethers.js, built and configured to work seamlessly with Alchemy. This provides multiple advantages such as automatic retries and robust WebSocket support.

### 1. From your [command line](https://www.computerhope.com/jargon/c/commandi.htm), create a new project directory and install the Alchemy SDK. 

To install the Alchemy SDK, you want to create a project, and then navigate to your project directory to run the installation. Let's go ahead and do that! Once we're in our home directory, let's execute the following:

With Yarn:

```console
mkdir your-project-name
cd your-project-name
yarn init # (or yarn init --yes)
yarn add alchemy-sdk

```

With NPM:

```console
mkdir your-project-name
cd your-project-name
npm init   # (or npm init --yes)
npm install alchemy-sdk
```


### 2. Create a file named `index.js` and add the following contents:

:::note

You should ultimately replace `demo` with your Alchemy HTTP API key.

:::

```js
// Setup: npm install alchemy-sdk
const { Network, Alchemy } = require("alchemy-sdk");

// Optional Config object, but defaults to demo api-key and eth-mainnet.
const settings = {
  apiKey: "demo", // Replace with your Alchemy API Key.
  network: Network.MATIC_MAINNET, // Replace with your network.
};

const alchemy = new Alchemy(settings);

async function main() {
  const latestBlock = await alchemy.core.getBlockNumber();
  console.log("The latest block number is", latestBlock);
}

main();

```


Unfamiliar with the async stuff? Check out this [Medium post](https://betterprogramming.pub/understanding-async-await-in-javascript-1d81bb079b2c).

### 3. Run it using node

```console
node index.js
```

### 4. You should now see the latest <<glossary:block>> number output in your console!

```console
The latest block number is 11043912
```


Congratulations - you just wrote your first web3 script using Alchemy and sent your first request to your Alchemy API endpoint.

The project associated with your API key should now look like this on the dashboard:

![img](https://files.readme.io/e3d2ffd-Alchemy_Tutorial_Result1.png)

![img](https://files.readme.io/bcfc9ff-Alchemy_Tutorial_Result2.png)


----------------

## 4. Start Building

Don't know where to start? Check out these four tutorials to get more familiar with Alchemy and blockchain development:

1. [Examples of Common Queries Using the Alchemy SDK](ref:using-the-alchemy-sdk)
2. Write a [simple web3 script](doc:how-to-get-the-latest-block-on-ethereum) 
3. Learn [How to Send Transactions on Ethereum](doc:how-to-send-transactions-on-ethereum) 
4. Try deploying your first [Hello World Smart Contract](doc:hello-world-smart-contract) and get your hands dirty with some solidity programming!

Once you complete this tutorial, let us know how your experience was or if you have any feedback by tagging us on Twitter [@alchemyplatform](https://twitter.com/AlchemyPlatform)!

----------------
###Other Web3 Libraries

Check out the documentation for each library:

* [Web3.py](https://web3py.readthedocs.io/en/stable/)
* [Web3j](https://docs.web3j.io)
* [Ethers.js](https://docs.ethers.io/v5/)
* [Web3.js](https://web3js.readthedocs.io/en/v1.2.9/)

Using the below code snippets, you can install and use Alchemy as a provider via any of the following libraries!

```python
# Setup: pip install web3
from web3 import Web3
alchemy = Web3(Web3.HTTPProvider("https://eth-mainnet.alchemyapi.io/v2/your-api-key"));

```

```java
// Setup: curl -L get.web3j.io | sh
Web3j web3 = Web3j.build(new HttpService("https://eth-mainnet.alchemyapi.io/v2/your-api-key"));
```

```js
// Setup: npm install ethers
const ethers = require("ethers");
const url = "https://eth-mainnet.alchemyapi.io/v2/your-api-key";
const customHttpProvider = new ethers.providers.JsonRpcProvider(url);
```

```js
// Setup: npm install web3
const Web3 = require('web3');
const web3 = new Web3("https://eth-mainnet.alchemyapi.io/v2/your-api-key");
```
