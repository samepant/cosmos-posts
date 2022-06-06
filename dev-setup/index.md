# DRAFT

This post will show you how to install and run a local chain using the Cosmos-sdk, and how to connect to it using the Keplr wallet. When it's all set up, you will have your own little test net, so you won't have to worry about finding a faucet to get gas to test your new app. All you need for this tutorial is a Command Line, an internet connection and a working Go install on your machine.

## Setup dev environment

1. Clone the cosmos-hub git repo, I like to [checkout the latest release](https://github.com/cosmos/cosmos-sdk/releases)
   `git checkout tags/v0.45.4`

2. Make sure [Go is installed on your machine](https://go.dev/doc/install). I like to check with: `go version`

3. Run `make build` in the comos-sdk repo. This will create the `build` folder in your `cosmos-sdk` directory.

If you've already run a local node using `./simd` see the [footnotes](#footnotes) for how to clear your environment to allow you to create another chain.

## Create your local chain

1. Navigate to `cosmos-sdk/build` directory
2. Initiate a new directory for your chain:
   `./simd init test-node --chain-id cheetah-chain`
   This command takes in a few values, of which only two are necessary for this demo:

- `test-node` in the above command is the moniker of the new directory you are working in, and doesn't matter since we won't be referencing it again.
- `cheetah-chain` in the above command is the name of the test chain we want to make. Make a note of it, since we will use this value later in the demo.

3. Create a new key
   `./simd keys add test-validator --keyring-backend test`
   This creates a new key (`test-validator`, you can name this whatever you want, but make a note of it as we will use it later), and uses the `test` key backend which is appropriate for a local testnet. The `test` backend SHOULD NOT be used for a chain that holds any value.

4. Add genesis account
   `./simd add-genesis-account $(./simd keys show test-validator -a --keyring-backend test) 10000000000000000000000000paw`

The genesis account is the account that starts the local chain and in a normal scenario, you would have multiple of these, but we only need one for our testnet. The `10000000000000000000000000paw` value is the amount of tokens our chain will start with, and doesn't matter too much, but it needs to be pretty big. I recommend using the exact number value you above, `paw` is the name of the chain's main token and can be whatever you want. The bash helper command `$(./simd keys show test-validator -a --keyring-backend test)` simply fetches the address of the key we created in step 3.

5. Create the genesis transaction for your chain
   `./simd gentx test-validator 1000000000000000000paw --chain-id cheetah-chain --keyring-backend test`

This command creates an initial transaction (giving some `paw` to `test-validator` account we created earlier). Be sure to match both `--chain-id` and `--keyring-backend` options to the values you used earlier in this demo.

6. Collect the genesis transactions
   `./simd collect-gentxs`
   In a real-world scenario, there would be many genesis transactions we would need to collect to properly start our chain, but since this is a single node testnet, there is only one, and this step is really easy.

7. Start your test chain 😮
   `./simd start`

If this command works, you should see a streaming log in your command line of chain events. To halt your local node (and halt your chain), use `ctrl-c`, but you should leave it running so we can connect to it in the next section.

```
./simd init sam-node --chain-id purp-chain
./simd keys add sam-validator --keyring-backend test
./simd add-genesis-account $(./simd keys show sam-validator -a --keyring-backend test) 10000000000000000000000000stake
./simd gentx sam-validator 1000000000000000000stake --chain-id purp-chain --keyring-backend test
./simd collect-gentxs
./simd start
```

## Add your test chain to Keplr

add your local chain to your keplr wallet :)

send your test wallet some tokens

```
./simd tx bank send $(./simd keys show sam-validator -a --keyring-backend test) cosmos1vqpjljwsynsn58dugz0w8ut7kun7t8ls2qkmsq 1000000000stake
```

Connect wallet to local chain

### Footnotes

If you've previously created a chain on your local machine, you may need to reset the database before you create a new one:

1. In your home directory, CAREFULLY run:
   `rm -rf ./.simapp`

2. In the `cosmos-sdk/build` directory, run:
   `./simd unsafe-reset-all`
