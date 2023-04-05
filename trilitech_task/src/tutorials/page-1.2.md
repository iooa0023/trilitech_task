# Wallet and user’s tokens

### Setting up the wallet

The wallet is a key element of your dapp, without it, the users won’t be able to interact with the Tezos blockchain, which defeats your purpose. There are multiple considerations to take into account when you are setting up the wallet that we will explain below.

1. Make sure the `Wallet.svelte` file looks something like this:

```TS
<script lang="ts">
  ... your TypeScript code
</script>

<style lang="scss">
   ... your SASS code
</style>

... your HTML code
```

Import the following libraries and components inside the `<script>` tag:

``` TS
  import { onMount } from "svelte";
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import store, { type TezosAccountAddress } from "../store";
  import { rpcUrl, network } from "../config";
  import { shortenHash, fetchBalances } from "../utils";
```

Below this, also declare these variables:

```TS
 let connectedNetwork = "";
 let walletIcon = "";
 let walletName = "";
```
```admonish tip title="What are these?"
- `connectedNetwork`: Used to store the name or identifier of a network that a user is currently connected to.
- `walletIcon`: Used to store the file path or URL of an icon or logo that represents a user's wallet.
- `walletName`: Used to store the name or identifier of a wallet that a user is currently using.
```

Also don't forget to include the `onMount` function we created previous and the end of the `<script>`.

2. We want to isolate the wallet and its different interactions and values in the same component, called `Wallet.svelte` in our example. When using the Beacon SDK, it is crucial to keep a single instance of Beacon running in order to prevent bugs.

Add the following inside the `<script>` tag:

```TS 
onMount(async () => {
    const wallet = new BeaconWallet({
      name: "Tezos dev portal dapp tutorial",
      preferredNetwork: network
    });
    store.updateWallet(wallet);
    const activeAccount = await wallet.client.getActiveAccount();
    if (activeAccount) {
      const userAddress = (await wallet.getPKH()) as TezosAccountAddress;
      store.updateUserAddress(userAddress);
      $store.Tezos.setWalletProvider(wallet);
      await getWalletInfo(wallet);
      // fetches user's XTZ, tzBTC and SIRS balances
      const res = await fetchBalances($store.Tezos, userAddress);
      if (res) {
        store.updateUserBalance("XTZ", res.xtzBalance);
        store.updateUserBalance("tzBTC", res.tzbtcBalance);
        store.updateUserBalance("SIRS", res.sirsBalance);
      } else {
        store.updateUserBalance("XTZ", null);
        store.updateUserBalance("tzBTC", null);
        store.updateUserBalance("SIRS", null);
      }
    }
  });
```

We create the instance of the `BeaconWallet` by providing a name for the dapp (it can be whatever you want) that will be displayed in the wallet UI and the network you want to connect to (imported from the config file). The instance of the wallet is then saved in the store.

Now, you want to check if the user connected a wallet before. Beacon will keep track of live connections in the local storage, this is how your users can navigate to your dapp and have their wallet connected automagically!

The `BeaconWallet` instance provides a `client` property with different methods, the one you need here is `getActiveAccount()`, which will retrieve any live connection stored in the local storage. If there is a live connection, you can fetch the user's address and save it into the store, update the store with the user's address before setting up the wallet as the signer with `$store.Tezos.setWalletProvider(wallet)`, get the information you need about the wallet (mainly, the name of the wallet) with the getWalletInfo() function and then, fetch the balances for the address that is connected with the fetchBalances() function described earlier. 

Once the balances are fetched, they are saved into the store to be displayed in the interface.

```admonish
TezosAccountAddress is a custom type I like to use to validate Tezos addresses for implicit accounts: `type TezosAccountAddress = tz${"1" | "2" | "3"}${string}`, TypeScript will raise a warning if you try to use a string that doesn't match this pattern.
```

### Connecting the wallet

Taquito and Beacon working in unison makes it very easy to connect the wallet. 

1. By adding a few lines of code so it looks like this:

```TS
const connectWallet = async () => {
  if (!$store.wallet) {
    const wallet = new BeaconWallet({
      name: "Tezos dev portal dapp tutorial",
      preferredNetwork: network
    });
    store.updateWallet(wallet);
  }
  await $store.wallet.requestPermissions({
    network: { type: network, rpcUrl }
  });
  const userAddress = (await $store.wallet.getPKH()) as TezosAccountAddress;
  store.updateUserAddress(userAddress);
  $store.Tezos.setWalletProvider($store.wallet);
  // finds account info
  await getWalletInfo($store.wallet);
  // fetches user's XTZ, tzBTC and SIRS balances
  const res = await fetchBalances($store.Tezos, userAddress);
  if (res) {
    store.updateUserBalance("XTZ", res.xtzBalance);
    store.updateUserBalance("tzBTC", res.tzbtcBalance);
    store.updateUserBalance("SIRS", res.sirsBalance);
  } else {
    store.updateUserBalance("XTZ", null);
    store.updateUserBalance("tzBTC", null);
    store.updateUserBalance("SIRS", null);
  }
};
```

The connection will be handled in a specific function called connectWallet. If the store doesn't hold an instance of the `BeaconWallet` (if the dapp didn't detect any live connection on mount), you create that instance and save it in the store.

Next, you ask the user to select a wallet with the `requestPermissions()` method present on the instance of the BeaconWallet. The parameter is an object where you indicate the network you want to connect to as well as the URL of the Tezos RPC node you will interact with.

After the user selects a wallet to use with our dapp, you get their address with the `getPKH()` method on the `BeaconWallet` instance, you update the signer in the `TezosToolkit` instance by passing the wallet instance to `setWalletProvider()`, you get the information you need from the wallet and you fetch the user's balances.

```admonish success title=""
Now, the wallet is connected and the user is shown their different balances, as well as a connection status in the sidebar!

```

```admonish warning title="IMPORTANT"
No matter how you decide to design your dapp, it is crucial to maintain only one instance of the BeaconWallet, and it is strongly advised to do the same with the instance of the TezosToolkit. If you create multiple instances, it can lead to complications with the state of your application and cause issues with Taquito overall.
```

### Disconnecting the wallet

Disconnecting the wallet is as important as connecting it. A lot of users have multiple wallets and addresses within the same wallet that they want to use to interact with your dapp.

Therefore adding the following code inside the `<script>` tag after our `connectWallet` funcctionlity will ensure this making it easier:

```TS
const disconnectWallet = async () => {
  $store.wallet.client.clearActiveAccount();
  store.updateWallet(undefined);
  store.updateUserAddress(undefined);
  connectedNetwork = "";
  walletIcon = "";
};
```

```admonish tip title="What are these?"
There are different steps to disconnect the wallet and reset the state of the dapp:

- `$store.wallet.client.clearActiveAccount()`: Terminates the current connection to Beacon.
- `store.updateWallet(undefined)`: Removes the wallet from the state in order to trigger a reload of the interface.
- `store.updateUserAddress(undefined)`: Removes the current user's address from the state to update the UI.
- `connectedNetwork = ""; walletIcon = ""`: needed to reset the state of the dapp and present an interface where no wallet is connected.

The call to `clearActiveAccount()` on the wallet instance is the only thing that you will do in whatever dapp you are building, it will remove all the data in the local storage and when your user revisits your dapp, they won't be automatically connected with their wallet.
```

```admonish example title ="Design considerations"
- Avoid prompting users to connect their wallet immediately after the dapp loads. Provide information about your dapp and a clear, prominently displayed button for users to manually connect their wallet.

- Ensure the button to connect a wallet stands out in your interface and is easy to locate.

- Keep the button in a predictable position, such as the top-left or top-right of the UI, to avoid user confusion.

- Use clear and concise language for the button text, such as "Connect." Display the wallet status, network, and balance to keep users informed.

- Enable/disable interactions that depend on a connected wallet to prevent user confusion.
```

### Fetching user’s balances

Fetching and displaying user balances is crucial in a dapp, and should not be overlooked. To ensure balances are shown and updated correctly, create a function in the utils.ts file and import it where necessary. In order to fetch balances, we use Taquito for XTZ and the [TzKT API](https://api.tzkt.io/) for tzBTC and SIRS. TzKT is a useful tool for building more complex applications on Tezos.

1. Import the following to the top the `utils.ts`:

```TS
import BigNumber from "bignumber.js";
import type { TezosToolkit } from "@taquito/taquito";
import type { token } from "./types";
import type { TezosAccountAddress } from "./store";
import { tzbtcAddress, sirsAddress } from "./config";
```

2. Then add the function `fetchBalances`:

```TS
export const fetchBalances = async (
  Tezos: TezosToolkit,
  userAddress: TezosAccountAddress
): Promise<{
  xtzBalance: number;
  tzbtcBalance: number;
  sirsBalance: number;
} | null> => {
  try {
    // the code will be here
  } catch (error) {
    console.error(error);
    return null;
  }
}
```

The `fetchBalances` function will take 2 parameters: An instance of the `TezosToolkit` to fetch the user's XTZ balance and the user's address to retrieve the balances that match the address. It will return an object with 3 properties: `xtzBalance`, `tzbtcBalance`, and `sirsBalance` or `null` if any error occurs.

3. Inisde the `try` add the following:

```TS
const xtzBalance = await Tezos.tz.getBalance(userAddress);
if (!xtzBalance) throw "Unable to fetch XTZ balance";
```

The instance of the `TezosToolkit` includes a property called tz that allows different Tezos-specific actions, one of them is about fetching the balance of an account by its address through the `getBalance()` method that takes the address of the account as a parameter.

Next, you check for the existence of a balance and you reject the promise if it doesn’t exist. If it does, the balance will be available as a [BigNumber](https://mikemcl.github.io/bignumber.js/).

```admonish
Taquito returns numeric values from the blockchain as BigNumber, because some values could be very big numbers and JavaScript is notorious for being bad at handling large numbers
```

4. Once the XTZ balance has been fetched, we can continue and fetch the balances using the following:

```TS
import { tzbtcAddress, sirsAddress } from "./config";

// previous code for the function
const res = await fetch(
  `https://api.tzkt.io/v1/tokens/balances?account=${userAddress}&token.contract.in=${tzbtcAddress},${sirsAddress}`
);
if (res.status === 200) {
  const data = await res.json();
  if (Array.isArray(data) && data.length === 2) {
    const tzbtcBalance = +data[0].balance;
    const sirsBalance = +data[1].balance;
    if (!isNaN(tzbtcBalance) && !isNaN(sirsBalance)) {
      return {
        xtzBalance: xtzBalance.toNumber(),
        tzbtcBalance,
        sirsBalance
      };
    } else {
      return null;
    }
  }
} else {
  throw "Unable to fetch tzBTC and SIRS balances";
}
```
```admonish info
This [link](https://api.tzkt.io/#operation/Tokens_GetTokenBalances) to get more details about how to `fetch` token balances with the TzKT API. It’s a simple fetch with a URL that is built dynamically to include the user's address and the addresses of the contracts for tzBTC and SIRS.
```
When the promise resolves with a `200 OK` code, this means that the data has been received. You parse it into JSON with the `.json()` method on the response and we check that the data has the expected shape, i.e. an array with 2 elements in it.

The first element is the tzBTC balance and the second one is the SIRS balance. You store them in their own variables that you cast to numbers before verifying that they were cast properly with `isNaN`. If everything goes well, the 3 balances are returned and if anything goes wrong along the way, the function returns `null`.

5. After fetching the balances in any component of our application, this data is stored to update the state in the `wallet.svelte`:

```TS
const res = await fetchBalances($store.Tezos, userAddress);
if (res) {
  store.updateUserBalance("XTZ", res.xtzBalance);
  store.updateUserBalance("tzBTC", res.tzbtcBalance);
  store.updateUserBalance("SIRS", res.sirsBalance);
} else {
  store.updateUserBalance("XTZ", null);
  store.updateUserBalance("tzBTC", null);
  store.updateUserBalance("SIRS", null);
}
```
```admonish success title=""
Now you can fetch a user’s balances in XTZ, tzBTC, and SIRS!

```

[← Previous Page](/tutorials/page-1.1.md)

[Next Page →](/tutorials/page-1.3.md)