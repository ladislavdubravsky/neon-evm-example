# neon-evm-example

Deploy Solidity smart contracts to Solana using [Neon EVM](https://neonevm.org/).

## Running local test setup

The following will run a local Solana cluster, deploy Neon EVM to it and run the offchain services which expose a standard Ethereum like interface to the user:

`docker compose -f docker-compose.yml pull`

`docker compose -f docker-compose.yml up`

To later clean up everything `docker compose -f docker-compose.yml down`.

We follow the [guide](https://neonevm.org/docs/operating/basic) but deviate from this point on to interact with the EVM proxy in an automated way using foundry scripts.

### Create an EVM account and fund it in NEON

Define an EVM keypair in `.env` following `.env.example`, then `source .env`.

Fund the test account:
```
curl --location --request POST 'http://localhost:3333/request_neon' \
--header 'Content-Type: application/json' \
--data-raw '{
"wallet": "'${ETH_ACCOUNT}'",
"amount": 1000
}'
```

Let's check the balance, at the same time verifying the Neon proxy can respond to Ethereum RPC formatted requests with correct responses:

`cast rpc --rpc-url http://localhost:9090/solana eth_getBalance $ETH_ACCOUNT latest`

If we want, we can use ETH style call to send funds from one funded Neon VM address to another one:

```
cast send --private-key $ETH_PRIVATE_KEY --rpc-url $NEON_ETH_RPC_URL 0x1804c8AB1F12E6bbf3894d4083f33e07309d1f38 --value 1wei --legacy
```

### Deploy the Counter.sol contract on Solana

```
cd contracts
forge build --evm-version cancun
forge script script/Counter.s.sol:CounterScript \
    --private-key $ETH_PRIVATE_KEY \
    --rpc-url $NEON_ETH_RPC_URL \
    --legacy \
    --skip-simulation \
    --evm-version cancun \
    --broadcast
```

(note the `--legacy` flag, otherwise forge will try and `Fail to get EIP-1559 fees`)

Forge may not recognize Neon included the transaction, so check it with:

`cast rpc --rpc-url $NEON_ETH_RPC_URL eth_getTransactionByHash <TXHASH>`

where TXHASH needs to be fetched from `./contracts/broadcast` folder.

### Interact with the contract

We'll do a basic sanity check - read, increase and read the counter value.

Get:

`cast call --rpc-url $NEON_ETH_RPC_URL 0xab8476cfa76302a646ae49e6ce84238030b7d9a2 "number()(uint256)"`
