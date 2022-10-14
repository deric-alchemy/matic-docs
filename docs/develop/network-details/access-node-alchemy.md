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

This tutorial guides you through starting and running a full node using binaries. For the system requirements, see the [Minimum Technical Requirements](technical-requirements) guide.

:::note

Steps in this guide involve waiting for the Heimdall and Bor services to fully sync. This process takes several days to complete.

Alternatively, you can use a maintained snapshot, which will reduce the sync time to a few hours. For detailed instructions, see [<ins>Snapshot Instructions for Heimdall and Bor</ins>](https://forum.polygon.technology/t/snapshot-instructions-for-heimdall-and-bor/9233).

For snapshot download links, see the [<ins>Polygon Chains Snapshots</ins>](https://snapshots.matic.today/) page.

:::

## **Overview**

- Prepare the machine
- Install Heimdall and Bor binaries on the full node machine
- Set up Heimdall and Bor services on the full node machine
- Configure the full node machine
- Start the full node machine
- Check node health with the community

:::note

You have to follow the exact outlined sequence of actions, otherwise you will run into issues.

:::

### **Install `build-essential`**

This is **required** for your full node. In order to install, run the below command:

```bash
sudo apt-get update
sudo apt-get install build-essential
```

### **Install GO**

This is also **required** for running your full node. Installing **v1.18 or above** is recommended.

```bash
wget https://raw.githubusercontent.com/maticnetwork/node-ansible/master/go-install.sh
bash go-install.sh
sudo ln -nfs ~/.go/bin/go /usr/bin/go
```

## **Install Binaries**

Polygon node consists of 2 layers: Heimdall and Bor. Heimdall is a tendermint fork that monitors contracts in parallel with the Ethereum network. Bor is basically a Geth fork that generates blocks shuffled by Heimdall nodes.

Both binaries must be installed and run in the correct order to function properly.

### **Heimdall**

Install the latest version of Heimdall and related services. Make sure you checkout to the correct [release version](https://github.com/maticnetwork/heimdall/releases). Note that the latest version, [Heimdall v.0.2.11](https://github.com/maticnetwork/heimdall/releases/tag/v0.2.11), contains enhancements such as:
1. Restricting data size in state sync txs to:
    * **30Kb** when represented in **bytes**
    * **60Kb** when represented as **string**
2. Increasing the **delay time** between the contract events of different validators to ensure that the mempool doesn't get filled very quickly in case of a burst of events which can hamper the progress of the chain.

The following example shows how the data size is restricted:

```
Data - "abcd1234"
Length in string format - 8
Hex Byte representation - [171 205 18 52]
Length in byte format - 4
```

To install **Heimdall**, run the below commands:

```bash
cd ~/
git clone https://github.com/maticnetwork/heimdall
cd heimdall

# Checkout to a proper version, for example
git checkout v0.2.11
git checkout <TAG OR BRANCH>
make install
source ~/.profile
```

That will install the `heimdalld` and `heimdallcli` binaries. Verify the installation by checking the Heimdall version on your machine:

```bash
heimdalld version --long
```

### **Bor**

Install the latest version of Bor. Make sure you git checkout to the correct [released version](https://github.com/maticnetwork/bor/releases).

```bash
cd ~/
git clone https://github.com/maticnetwork/bor
cd bor

# Checkout to a proper version
# For e.g: git checkout 0.2.16
git checkout <TAG OR BRANCH>
make bor-all
sudo ln -nfs ~/bor/build/bin/bor /usr/bin/bor
sudo ln -nfs ~/bor/build/bin/bootnode /usr/bin/bootnode
```

That will install the `bor` and `bootnode` binaries. Verify the installation by checking the Bor version on your machine:

```bash
bor version
```

## **Configure Node Files**

### **Fetch launch repo**

```bash
cd ~/
git clone https://github.com/maticnetwork/launch
```

### **Configure launch directory**

To set up the network directory, the network name and type of node are required.

**Available networks**: `mainnet-v1` and `testnet-v4`

**Node type**: `sentry`

:::tip

For Mainnet and Testnet configuration, use appropriate `<network-name>`. Use `mainnet-v1` for Polygon mainnet and `testnet-v4` for Mumbai Testnet.
:::

```bash
cd ~/
mkdir -p node
cp -rf launch/<network-name>/sentry/<node-type>/* ~/node
```

### **Configure network directories**

**Heimdall data setup**

```bash
cd ~/node/heimdall
bash setup.sh
```

**Bor data setup**

```bash
cd ~/node/bor
bash setup.sh
```

## **Configure Service Files**

Download **service.sh** file using appropriate `<network-name>`. Use `mainnet-v1` for Polygon mainnet and `testnet-v4` for Mumbai Testnet.

```bash
cd ~/node
wget https://raw.githubusercontent.com/maticnetwork/launch/master/<network-name>/service.sh
```

Generate the **metadata** file:

```bash
sudo mkdir -p /etc/matic
sudo chmod -R 777 /etc/matic/
touch /etc/matic/metadata
```

Generate **.service** files and copy them into system directory:

```bash
cd ~/node
bash service.sh
sudo cp *.service /etc/systemd/system/
```


## **Setup Config Files**

- Log in to the remote machine / VM
- You will need to add a few details in the **config.toml** file. To open and edit the **config.toml** file, run the following command: `vi ~/.heimdalld/config/config.toml`.

    In the config file, you will have to change `Moniker` and add `seeds` information:

    ```bash
    moniker=<enter unique identifier>
    # For example, moniker=my-sentry-node

    # Mainnet:
    seeds="f4f605d60b8ffaaf15240564e58a81103510631c@159.203.9.164:26656,4fb1bc820088764a564d4f66bba1963d47d82329@44.232.55.71:26656"

    # Testnet:
    seeds="4cd60c1d76e44b05f7dfd8bab3f447b119e87042@54.147.31.250:26656,b18bbe1f3d8576f4b73d9b18976e71c65e839149@34.226.134.117:26656"
    ```

    - Change the value of **Pex** to `true`
    - Change the value of **Prometheus** to `true`
    - Set the `max_open_connections` value to `100`

  Make sure you **keep the proper formatting** when you make the above changes.

- Configure the following in `~/.heimdalld/config/heimdall-config.toml`:

    ```jsx
    eth_rpc_url=<insert Infura or any full node RPC URL to Goerli>
    ```

- Open the **start.sh** file for Bor using this command: `vi ~/node/bor/start.sh`. Add the following flags to start params:

  ```bash
  # Mainnet:
  --bootnodes "enode://0cb82b395094ee4a2915e9714894627de9ed8498fb881cec6db7c65e8b9a5bd7f2f25cc84e71e89d0947e51c76e85d0847de848c7782b13c0255247a6758178c@44.232.55.71:30303,enode://88116f4295f5a31538ae409e4d44ad40d22e44ee9342869e7d68bdec55b0f83c1530355ce8b41fbec0928a7d75a5745d528450d30aec92066ab6ba1ee351d710@159.203.9.164:30303"

  # Testnet:
  --bootnodes "enode://320553cda00dfc003f499a3ce9598029f364fbb3ed1222fdc20a94d97dcc4d8ba0cd0bfa996579dcc6d17a534741fb0a5da303a90579431259150de66b597251@54.147.31.250:30303"
  ```

- To enable **Archive** mode, you can add the following flags in the `start.sh` file:

  ```jsx
  --gcmode 'archive' \
  --ws --ws.port 8546 --ws.addr 0.0.0.0 --ws.origins '*' \
  ```

## **Start Services**

Run the full Heimdall node with these commands on your Sentry Node:

```bash
sudo service heimdalld start
sudo service heimdalld-rest-server start
```

Now, you need to make sure that **Heimdall is synced** completely, and then only start Bor. If you start Bor without Heimdall syncing completely, you will run into issues frequently.

**To check if Heimdall is synced**
  1. On the remote machine/VM, run `curl localhost:26657/status`
  2. In the output, `catching_up` value should be `false`

Once Heimdall is synced, run the below command:

```bash
sudo service bor start
```

## **Logs**

Logs can be managed by the `journalctl` linux tool. Here is a tutorial for advanced usage: [How To Use Journalctl to View and Manipulate Systemd Logs](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs).

**Check Heimdall node logs**

```bash
journalctl -u heimdalld.service -f
```

**Check Heimdall Rest-server logs**

```bash
journalctl -u heimdalld-rest-server.service -f
```

**Check Bor Rest-server logs**

```bash
journalctl -u bor.service -f
```

## **Ports and Firewall Setup**

Open ports 22, 26656 and 30303 to world (0.0.0.0/0) on sentry node firewall.

You can use VPN to restrict access for port 22 as per your requirement and security guidelines.

# Getting Started

:wave: *New to Alchemy? Get access to Alchemy for free* ***[here](https://alchemy.com/?r=e68b2f77-7fc7-4ef7-8e9c-cdfea869b9b5)***.

_Estimated time to complete this guide: < 10 minutes_

------------------

## :clipboard: Steps to get started with Alchemy

This guide assumes you already have an [Alchemy account](https://alchemy.com/?r=e68b2f77-7fc7-4ef7-8e9c-cdfea869b9b5) and access to our [Dashboard](https://dashboard.alchemyapi.io).

**1**. :key: [Create an Alchemy key](doc:alchemy-quickstart-guide#1key-create-an-alchemy-key)

**2**. ✍️ [Make a request](doc:alchemy-quickstart-guide#2-%EF%B8%8F-make-your-first-request)

**3**. 🤝 [Set up Alchemy as your client](doc:alchemy-quickstart-guide/#3-🤝-set-up-alchemy-as-your-client)

4\. :computer:[ Start building!](doc:alchemy-quickstart-guide#4-computer-start-building)

------------------

## 1.:key: Create an Alchemy Key

To use Alchemy's products, you need an API key to authenticate your requests.

You can [create API keys from the dashboard](http://dashboard.alchemyapi.io). Check out this video on how to create an app:


[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://www.youtube.com/watch?v=tfggWxfG9o0)


[block:embed]
{
  "html": "<iframe class=\"embedly-embed\" src=\"//cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fwww.youtube.com%2Fembed%2FtfggWxfG9o0%3Ffeature%3Doembed&display_name=YouTube&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DtfggWxfG9o0&image=https%3A%2F%2Fi.ytimg.com%2Fvi%2FtfggWxfG9o0%2Fhqdefault.jpg&key=f2aa6fc3595946d0afc3d76cbbd25dc3&type=text%2Fhtml&schema=youtube\" width=\"854\" height=\"480\" scrolling=\"no\" title=\"YouTube embed\" frameborder=\"0\" allow=\"autoplay; fullscreen\" allowfullscreen=\"true\"></iframe>",
  "url": "https://www.youtube.com/watch?v=tfggWxfG9o0",
  "title": "Create an Ethereum API Key with Alchemy",
  "favicon": "https://www.youtube.com/s/desktop/592786db/img/favicon.ico",
  "image": "https://i.ytimg.com/vi/tfggWxfG9o0/hqdefault.jpg"
}
[/block]


Or follow the written steps below:

First, navigate to the "create app" button in the "Apps" tab.


![img](https://files.readme.io/693457a-Getting_Started.png)


Fill in the details under "Create App" to get your new key. You can also see apps you previously made and those made by your team here. Pull existing keys by clicking on "View Key" for any app.

![img](https://files.readme.io/d6172a5-Create_App_Details.png)


You can also pull existing API keys by hovering over "Apps" and selecting one. You can "View Key" here, as well as "Edit App" to whitelist specific domains, see several developer tools, and view analytics.

![img](https://files.readme.io/f0dbb19-ezgif.com-gif-maker_1.gif)


-------------------

## 2. ✍️ Make Your First Request

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

## 3. 🤝 Set up Alchemy as your Client

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
  network: Network.ETH_MAINNET, // Replace with your network.
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


Woo! Congrats! You just wrote your first web3 script using Alchemy and sent your first request to your Alchemy API endpoint 🎉

The project associated with your API key should now look like this on the dashboard:

![img](https://files.readme.io/e3d2ffd-Alchemy_Tutorial_Result1.png)

![img](https://files.readme.io/bcfc9ff-Alchemy_Tutorial_Result2.png)


----------------

## 4. :computer: Start Building!

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
