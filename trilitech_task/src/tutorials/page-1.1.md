# Setting up the dapp

As we are building a web app with the Svelte framework, the steps to set up the project will be very similar to the ones you would follow to set up any other web app.

In this tutorial, we will make a Svelte SPA, so we won’t need SvelteKit, which will also make our life easier.

1. The first thing to do is to install Svelte with TypeScript and Vite:

```
npm create vite@latest app -- --template svelte-ts
cd lb-dex
npm install
```

2. Next, we will install all the dependencies we need for the dapp:

```
npm install --save-dev sass
npm install @taquito/taquito @taquito/beacon-wallet
```

```admonish info
Sass is a development-only dependency, `@taquito/taquito` is the NPM package for the Taquito library and `@taquito/beacon-wallet` is the NPM package that contains Beacon with some little configuration to make it easier to plug into Taquito.
```

3. There are a couple of other libraries we need to install:

```
npm install --save-dev buffer events vite-compatible-readable-stream
```

These libraries are required to be able to run Beacon in a Svelte app. We will see down below how to use them.
Once everything has been installed, we have to set up the right configuration.

4. In your app folder, you will see the `vite.config.js` file, it's the file that contains the configuration that Vite needs to run and bundle your app. Make the following changes:

```JS
import { defineConfig, mergeConfig } from "vite";
import path from "path";
import { svelte } from "@sveltejs/vite-plugin-svelte";

export default ({ command }) => {
  const isBuild = command === "build";
  
  return defineConfig({
    plugins: [svelte()],
    define: {
      global: {}
    },
    build: {
      target: "esnext",
      commonjsOptions: {
        transformMixedEsModules: true
      }
    },
    server: {
      port: 4000
    },
    resolve: {
      alias: {
        "@airgap/beacon-sdk": path.resolve(
          path.resolve(),
          `./node_modules/@airgap/beacon-sdk/dist/${
            isBuild ? "esm" : "cjs"
          }/index.js`
        ),
        // polyfills
        "readable-stream": "vite-compatible-readable-stream",
        stream: "vite-compatible-readable-stream"
      }
    }
  });
};
```

Here are a few changes we made to the template configuration given by Vite:

- We set `global` to `{}` and we will later provide the global object in our HTML file.
- We provide a path to the Beacon SDK.
- We provide polyfills for `readable-stream` and `stream`.

5. Once these changes have been done, there is a last step to finish setting up the project: we have to update the HTML file where the JavaScript code will be injected.

Inside the `index.html` file, you should have the following:


```HTML
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script>
      const global = globalThis;
    </script>
    <script type="module">
      import { Buffer } from "buffer";
      window.Buffer = Buffer;
    </script>
    <title>Liquidity Baking DEX</title>
  </head>
  <body>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

- In the first script tag, we set the global variable to globalThis. 
- In the second script tag with a module type, we import Buffer from the buffer library and add it to the window global object.

```admonish
This configuration is required to run the Beacon SDK with a Vite app. Taquito works completely out of the box and doesn’t require any settings.
```

6. Once we updated the configuration in the `vite.config.js` file and in the `index.html` file, our project is successfully set up! 

Run the following in your terminal at the root of the project to check that everything works properly:

```
npm run dev
```

```admonish success title="Expected Output"
  VITE v4.2.1  ready in 680 ms

  ➜  Local:   http://localhost:4000/

  ➜  Network: use --host to expose

  ➜  press h to show help

```
Congratulations! The dapp should be running on http://localhost:4000. You can also see the activity log on the application in your terminal.

### File Structure

The entrypoint of every Svelte app is a file called `App.svelte`, this is where you will import all your components to be bundled together into your final app. If you haven't done so, create the folders and files n eccessary for this tutorial so the file structure of our project looks like this:

```
├── src/
│   ├── assets/
│   │   └── svelte.png
│   ├── lib/
│   │   ├── AddLiquidityView.svelte
│   │   ├── Interface.svelte
│   │   ├── RemoveLiquidity.svelte
│   │   ├── Sidebar.svelte
│   │   ├── SirsStats.svelte
│   │   ├── SwapView.svelte
│   │   ├── Toast.svelte
│   │   ├── UserInput.svelte
│   │   ├── UserStats.svelte
│   │   └── Wallet.svelte
│   ├── styles/
│   │   ├── index.scss
│   │   └── settings.scss
│   ├── App.svelte
│   ├── config.ts
│   ├── lbUtils.ts
│   ├── main.ts
│   ├── store.ts
│   ├── types.ts
│   └── utils.ts
├── index.html
├── svelte.config.js
├── tsconfig.json
└── vite.config.js
```

```admonish tip title="What are these?"

- __assets__: contains the favicon (here, this is the default Svelte favicon, but you can choose another one)
- __lib__: contains the different components that will make up our interface, here is what each does:
	- `SwapView.svelte`: the interface to swap XTZ and tzBTC tokens
	- `AddLiquidityView.svelte`: the interface to add liquidity to the LB DEX
	- `RemoveLiquidity.svelte`: the interface to remove liquidity from the LB DEX
	- `Interface.svelte`: the higher-order component to hold the different views to interact with the LB DEX
	- `Sidebar.svelte`: the component to navigate between the different interfaces and to connect or disconnect the wallet
	- `SirsStats.svelte`: the component to display the amount of XTZ, tzBTC, and SIRS present in the contract
	- `Toast.svelte`: a simple component to display the progression of the transactions and other messages when interacting with the contract
	- `UserInput.svelte`: a utility component to make it easier to interact and control input fields
	- `UserStats.svelte`: the component to display the user's balance in XTZ, tzBTC, and SIRS
	- `Wallet.svelte`: the component to manage wallet interactions
- __styles__: contains the SASS files to style different elements of our interface
- __App.svelte__: the entrypoint of the application
- __config.ts__: different immutable values needed for the application and saved in a separate file for convenience
- __lbUtils.ts__: different methods to calculate values needed to interact with the Liquidity Baking contract
- __main.ts__: this is where the JavaScript for the app is bundled before being injected into the HTML file
- __store.ts__: a file with a [Svelte store](https://svelte.dev/tutorial/writable-stores) to handle the dapp state
- __types.ts__: custom TypeScript types
- __utils.ts__: different utility methods
```
1. The first thing to do is to import our styles into the main.ts file:

```TS
import App from './App.svelte'
import "./styles/index.scss";

const app = new App({
 target: document.body
});
export default app;
```

```admonish
We recommend targetting the `body` tag to inject the HTML produced by JavaScript instead of a `div` inside the `body`.
```

2. Check the `App.svelte` file looks something like this:

```TS
<script lang="ts">
  ... your TypeScript code
</script>

<style lang="scss">
   ... your SASS code
</style>

... your HTML code
```

Svelte components are fully contained, which means that the style that you apply inside a component doesn’t leak into the other components of your app. The style that we want to share among different components will be written in the `index.scss` file.

There is a `script` tag with a lang attribute set to `ts` for TypeScript, a style tag with a `lang` attribute set to `scss` for SASS and the rest of the code in the file will be interpreted as HTML.

### Configuring the dapp

1. Let’s set up things differently in our `App.svelte` file.

The HTML part is just going to put all the higher-order components together. Replace the HTML with the following:

```HTML
<main>
  <Toast />
  {#if $store.Tezos && $store.dexInfo}
    <Sidebar />
    <Interface />
  {:else}
    <div>Loading</div>
  {/if}
</main>
```

The interface will change after different elements are available to the dapp, mostly, the data about the liquidity pools from the liquidity baking contract.

2. Replace the SASS part of the file with the following:

```CSS
@import "./styles/settings.scss";

main {
 display: grid;
 grid-template-columns: 250px 1fr;
 gap: $padding;
 padding: $padding;
 height: calc(100% - (#{$padding} * 2));
}
@media screen and (max-height: 700px) {
 main {
   padding: 0px;
   height: 100%;
 }
}
```

3. For TypeScript part import the libraries and components we need inside the `<script>` tag:

```TS
import { onMount } from "svelte";
import { TezosToolkit } from "@taquito/taquito";
import store from "./store";
import { rpcUrl, dexAddress } from "./config";
import Sidebar from "./lib/Sidebar.svelte";
import Interface from "./lib/Interface.svelte";
import Toast from "./lib/Toast.svelte";
import type { Storage } from "./types";
import { fetchExchangeRates } from "./utils";
```
```admonish tip title="What are these?"
- `onMount` is a method exported by Svelte that will run some code when the component mounts (more on that below)
- `TezosToolkit` is the class that gives you access to all the features of Taquito
- `store` is a Svelte feature to manage the state of the dapp
- From the `config.ts` file, we import `rpcUrl` (the URL of the Tezos RPC node) and `dexAddress`, the address of the Liquidity Baking contract
- `Storage` is a custom type that represents the signature type of the LB DEX storage
- `fetchExchangeRates` is a function to fetch the exchange rates of XTZ and tzBTC (more on that below)
```

4. Additionally include the `onMount` to set up the state of the dapp also inside the `<script>` tag after the imprted librarires and components:

```TS
onMount(async () => {
  const Tezos = new TezosToolkit(rpcUrl);
  store.updateTezos(Tezos);
  const contract = await Tezos.wallet.at(dexAddress);
  const storage: Storage | undefined = await contract.storage();

  if (storage) {
    store.updateDexInfo({ ...storage });
  }
  // fetches XTZ and tzBTC prices
  const res = await fetchExchangeRates();
  if (res) {
    store.updateExchangeRates([
      { token: "XTZ", exchangeRate: res.xtzPrice },
      { token: "tzBTC", exchangeRate: res.tzbtcPrice }
    ]);
  } else {
    store.updateExchangeRates([
      { token: "XTZ", exchangeRate: null },
      { token: "tzBTC", exchangeRate: null }
    ]);
  }
});
```

The first thing to do is to create an instance of the `TezosToolkit` by passing the URL of the RPC node we want to interact with. In general, you want to have a single instance of the `TezosToolkit` in order to keep the same configuration across all your app components, this is why we save it in the store with the `updateTezos` method.

After that, we want to fetch the storage of the LB DEX to get the amounts of XTZ, tzBTC, and SIRS in the contract. We create a `ContractAbstraction`, an instance provided by Taquito with different properties and methods that are useful to work with Tezos smart contracts. From the `ContractAbstraction`, we can call the `storage` method that returns a JavaScript object that represents the storage of the given contract. We then pass the storage to the `updateDexInfo` method present on the `store` to update this data and display them to the user.

5. To finish this part of the tutorial, we need to fetch the exchange rates for XTZ and tzBTC to make the conversions required by this kind of app. 

Inside the `utils.ts` file, paste the folllowing function:

``` TS
export const fetchExchangeRates = async (): Promise<{
  tzbtcPrice: number;
  xtzPrice: number;
} | null> => {
  const query = `
      query {
        overview { xtzUsdQuote },
        token(id: "KT1PWx2mnDueood7fEmfbBDKx1D9BAnnXitn") { price }
      }
    `;
  const res = await fetch(`https://analytics-api.quipuswap.com/graphql`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      query
    })
  });
  if (res.status === 200) {
    const resData = await res.json();
    let xtzPrice = resData?.data?.overview?.xtzUsdQuote;
    let tzbtcPrice = resData?.data?.token?.price;
    // validates the 2 values
    if (xtzPrice && tzbtcPrice) {
      xtzPrice = +xtzPrice;
      tzbtcPrice = +tzbtcPrice;
      if (!isNaN(xtzPrice) && !isNaN(tzbtcPrice)) {
        // tzBTC price is given in XTZ by the API
        tzbtcPrice = tzbtcPrice * xtzPrice;
        return { tzbtcPrice, xtzPrice };
      }
    } else {
      return null;
    }
  } else {
    return null;
  }
};
```

The exchange rates required for the DEX are obtained by utilizing the [QuipuSwap GraphQL API](https://analytics-api.quipuswap.com/graphql). Upon receiving the exchange rates, we parse the response to validate the prices offered for XTZ and tzBTC. Following successful validation, the function returns the confirmed prices which can then be stored. The exchange rates are instrumental in calculating the total value locked in the contract, denominated in USD or other fiat currencies.

[← Previous Page](/tutorials/page-1.0.md)

[Next Page →](/tutorials/page-1.2.md)