# DRAFT

This post will show you how to install and run a local chain using the Cosmos-sdk, and how to connect to it using the Keplr wallet. When it's all set up, you will have your own little test net, so you won't have to worry about finding a faucet to get gas to test your new app. All you need for this tutorial is a Command Line, an internet connection and a working Go installation on your machine.

## Setup dev environment

1. Clone the [cosmos-hub git repo](https://github.com/cosmos/cosmos-sdk) `git@github.com:cosmos/cosmos-sdk.git`. I also like to [checkout the latest release](https://github.com/cosmos/cosmos-sdk/releases)

```bash
git fetch
git checkout tags/v0.45.4
```

2. Make sure [Go is installed on your machine](https://go.dev/doc/install). I like to check with: `go version`. At the time of this writing, at least version `1.17` of golang is required.

3. Run `make build` in the comos-sdk repo. This will create the `build` folder in your `cosmos-sdk` directory.

If you've already run a local node using `./simd` see the [footnotes](#footnotes) for how to clear your environment to allow you to create another chain.

## Create your local chain

1. Navigate to `cosmos-sdk/build` directory
2. Initiate a new directory for your chain:
   `./simd init test-node --chain-id cheetah-chain`
   This command takes in a few values, of which only two are necessary for this demo:

- `test-node` in the above command is the moniker of the new directory you are working in, and doesn't matter since we won't be referencing it again.
- `cheetah-chain` in the above command is the name of the test chain we want to make. You can name this whatever you want (no whitespace allowed). Make a note of it, since we will use this value later in the demo.

3. Create a new key
   `./simd keys add test-validator --keyring-backend test`
   This creates a new key (`test-validator`, you can name this whatever you want, but make a note of it as we will use it later), and uses the `test` key backend which is appropriate for a local testnet. The `test` backend SHOULD NOT be used for a chain that holds any value.

4. Add genesis account
   `./simd add-genesis-account $(./simd keys show test-validator -a --keyring-backend test) 10000000000000000000000000stake`

The genesis account is the account that starts the local chain and in a normal scenario, you would have multiple of these, but we only need one for our testnet. The `10000000000000000000000000stake` value doesn't matter too much, but it needs to be pretty big. I recommend using the exact number value I did above. The bash helper command `$(./simd keys show test-validator -a --keyring-backend test)` simply fetches the address of the key we created in step 3.

5. Create the genesis transaction for your chain
   `./simd gentx test-validator 1000000000000000000stake --chain-id cheetah-chain --keyring-backend test`

This command creates an initial transaction (giving some `stake` to `test-validator` account we created earlier). Be sure to match both `--chain-id` and `--keyring-backend` options to the values you used earlier in this demo.

6. Collect the genesis transactions
   `./simd collect-gentxs`
   In a real-world scenario, there would be many genesis transactions we would need to collect to properly start our chain, but since this is a single node testnet, there is only one, and this step is really easy.

7. Start your test chain
   `./simd start`

If this command works, you should see a streaming log in your command line of chain events. To halt your local node (and halt your chain), use `ctrl-c`, but you should leave it running so we can connect to it in the next section.

So all together, the commands we ran are as follows:

```bash
./simd init test-node --chain-id cheetah-chain
./simd keys add test-validator --keyring-backend test
./simd add-genesis-account $(./simd keys show test-validator -a --keyring-backend test) 10000000000000000000000000stake
./simd gentx test-validator 1000000000000000000stake --chain-id cheetah-chain --keyring-backend test
./simd collect-gentxs
./simd start
```

## Enable HTTP API

By default, your local node will accept RPC and GRPC connections, which is great for talking to other nodes, but we will need to enable the REST API to connect with Keplr. To do this, you will need to modify the `~/.simapp/config/app.toml` file that was created when we initialized our new chain. In `~/.simapp/config/app.toml` you will see:

```toml
###############################################################################
###                           API Configuration                             ###
###############################################################################

[api]

# Enable defines if the API server should be enabled.
enable = true
```

Make sure `enable` is set to `true`, which means the API will automatically be started whenever you start your node. Restart your node and you should see the following in the logs:

```
INF starting API server... module=api-server
INF Starting RPC HTTP server on [::]:1317 module=api-server
```

## Add your test chain to Keplr

To actually use your running test chain in your app, you will need to add it to the Keplr wallet. Keplr doesn't expose a way to do this in the extension, but does expose a javascript method that makes it pretty straightforward — `window.keplr.experimentalSuggestChain()`. You can paste the following into a browser console, or use this app for a more approachable experience.

```javascript
try {
  if (!window.keplr) {
    alert("Please install keplr extension");
  } else {
    const res = await window.keplr.experimentalSuggestChain({
      chainId: "cheetah-chain",
      chainName: "cheetah test chain",
      rpc: "http://127.0.0.1:26657",
      rest: "http://127.0.0.1:1317",
      bip44: {
        coinType: 118,
      },
      bech32Config: {
        bech32PrefixAccAddr: "cosmos",
        bech32PrefixAccPub: "cosmos" + "pub",
        bech32PrefixValAddr: "cosmos" + "valoper",
        bech32PrefixValPub: "cosmos" + "valoperpub",
        bech32PrefixConsAddr: "cosmos" + "valcons",
        bech32PrefixConsPub: "cosmos" + "valconspub",
      },
      currencies: [
        {
          coinDenom: "stake",
          coinMinimalDenom: "stake",
          coinDecimals: 6,
        },
      ],
      feeCurrencies: [
        {
          coinDenom: "stake",
          coinMinimalDenom: "stake",
          coinDecimals: 6,
        },
      ],
      stakeCurrency: {
        coinDenom: "stake",
        coinMinimalDenom: "stake",
        coinDecimals: 6,
      },
      coinType: 118,
      gasPriceStep: {
        low: 1,
        average: 1,
        high: 1,
      },
    });

    console.log("Successfully added chain to Keplr");
  }
} catch (error) {
  console.log(error);
}
```

When your Keplr wallet has succesfully added the test chain, you will be able to see an address for the chain and receive tokens from other accounts on the chain (which at the moment will only be our `test-validator` account from above).

To send some tokens from the validator account to your new address use this command in your command line (the test net must be running in another window).

```bash
./simd tx bank send $(./simd keys show test-validator -a --keyring-backend test) [YOUR KEPLR ADDRESS HERE] 1000000000stake --chain-id cheetah-chain --keyring-backend test
```

Once that command goes through, open Keplr and make sure you see the tokens reflected in your wallet balance.

### Footnotes

If you've previously created a chain on your local machine, you may need to reset the database before you create a new one (THIS WILL REMOVE ANY CHAIN DATA FROM PREVIOUS INITIALIZATIONS):

1. In your home directory, remove the `.simapp` directory

2. In the `cosmos-sdk/build` directory, run:
   `./simd tendermint unsafe-reset-all`
